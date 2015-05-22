#DCP Performance Analysis

##Test Scenarios

* All tests were run on Centos 6 VMs
* Builds used: 3.0.2-1644 and 4.0.0-1655 (sherlock)
* Nodes: 2; Replica count: 1
* Tests were run for couchbase server on **4 Core** and **8 core** machines with **4GB** and **8GB** RAM
* Tests were run with 100% resident ratio and with DGM (about 50% resident ratio)
* Load: Total incoming mutations on all nodes
* Items and size: 1,000,000 items in non DGM case and 2,500,000 in DGM case. Item size: 1KB.
* Replication, replication with views, replication with xdcr, replication with xdcr and views and with multiple number of external clients tested with front end load.
* External clients were python clients that essentially kept consuming data without any inherent delays.

*Note: XDCR tests were not run on sherlock build*

##Performance Indicators
### Replication Latency
Replication latency is measured as the time taken for an item update to appear at the replica node. This is sampled across a subset of items that are updated by the front end load. 80th, 95th, 99th percentiles of the latencies are captured. A latency within **0.5s for 95th percentile** and **1s for 99th percentile** is considered **good enough**.

### Replication Items Queue Size
Number of items in the replication queue. This is got by polling EP Engine stats which indicates the items that are yet to be replicated at that instance. The average size of the queue and the max observed queue sizes are captured. Spikes in the queue size indicated that replication is suffering (since our test case only has a steady load with no rebalance or failover). 

### Rate of Replication Bytes Sent
Rate at which replication bytes are sent over the network is captured and is measured as bytes per second. This is got by polling EP Engine stats. Average, maximum and minimum of these sampled rates were captured. Fall in the replication rates, with instances of 0 bytes/sec is not considered good for replication.

## Flow Control Buffer

After a certain front end load, we see spikes in replication queue sizes and the replication latencies even though the CPU usage is not close to 100%. We suspect the reason for this is due to the flow control buffer becoming full. The flow control buffer is currently a fixed value irrespective of the bucket size and streaming of items is suspended until the buffer is drained.

                                    Replication only 11k ops/sec (acceptable latency)
![Good replication latency](3.0.2/8-core-4gb-ram-vm/images/replication_only/11k_latency_raw.png)

                                    Replication only 13k ops/sec (high latency)
![High replication latency](3.0.2/8-core-4gb-ram-vm/images/replication_only/13k_latency_raw.png)
                                            
## Backfill Consuming Large Memory
In 3.0.2 backfilling is done directly onto memory. In every backfill, we read all the items, in that particular sequence range, from the disk onto the memory. For every DCP connection we do a bunch of backfills in the beginning. With the increase in the number of external clients (connections) **beyond 5**, the memory used by backfills increase unboundedly. Hence the replication suffers initially until we reach a steady state. The time taken to reach the steady state increases with the increase in the number of connections. This problem is **not seen in sherlock** because we have a backfill buffer per DCP connection and the memory usage by the backfills is bound by this.

                                      Replication with 10 clients 6k ops/sec
![Initial Replication Queue Size build-up](3.0.2/8-core-8gb-ram-vm-DGM/images/replication_10_clients/6k_items.png)

On 3.0.2 sometimes we observed that with more external clients the front end load does not build up. Again we attribute this to the unbounded memory usage of backfills.

## Views and XDCR
**View** is equivalent to an additional external connection. But, there will be continuous stream open and stream close operations. That is because a view will repeatedly open a stream, read latest items and then close the stream. This will cause continuous backfilling.

                                      Replication with XDCR 11k ops/sec
![Replication with XDCR](3.0.2/8-core-4gb-ram-vm/images/xdcr/11k_latency_raw.png)

**XDCR** is equivalent to 16 external connections in 3.0.2. Unlike replication or external clients, DCP streams open and close continuously in XDCR contributing to continuous backfilling requests.

Both Views and XDCR contribute to increased memory usage and also contend with the replication thread for the CPU, thereby increasing the replication latencies.

##4 core vs 8 core
* Results with 8 cores is better but similar behaviour characteristics
* In general, if we solve the problem of memory used up by backfills in 3.0.2 and set appropriate size to backfill buffers in sherlock, with more cores we can support more load with good enough replication latencies.

##4GB RAM vs 8GB RAM
* In 3.0.2, with 8GB RAM we can have relatively more external clients, **about 10**, before we see backfills using up a lot of memory and thereby throttling the replication.

##DGM vs Non DGM
* Similar results with baseline tests
* DGM in 3.0.2 we can have relatively less external clients, **about 5**, before we see backfills using up a lot of memory and thereby throttling the replication..

##3.0.2 vs Sherlock
* In sherlock backfills do not use up a lot of memory intially as we have a backfill buffer per connection and the memory usage by the backfills is bound by this. Thereby we can have more external connections without seeing drop in replication.


## Conclusions

###Baseline Tests
* **Replication only:** Replication latencies good until 12k load
* **Replication with views:** Replication latencies good until 11k load
* **Replication with xdcr:** Replication latencies good until 8k load on 8core 8GB VM
* **Replication with xdcr and views:** Replication latencies good until 6k load

###External clients
We can characterize the behavior with external clients by considering it in 2 different phases, i) **Intial state:** start of all DCP connections when we do a backfill for all the connections  and ii) **Steady state:** after completion of the backfill.

* **Initial state:** In 3.0.2, replication suffers due to excessive memory usage by backfills. This problem is NOT seen in sherlock.
* **Steady state:** When compared to (the steady state of) baseline replication only case at the same front end load, increase is seen in the replication latencies and replication queue sizes. This is due to increased contention for CPU by the replication thread and the threads that serve external clients.


##Further Improvements
### Variable flow control buffer size ******
Set flow control buffer size as a percentage of the bucket memory size. Flow control is used to avoid a slow DCP consumer being overrun by a fast DCP producer. DCP producer stops sending the data when the flow control buffer is full on the consumer side. Currently the flow control buffer is set to a constant value of 10MB irrespective of the bucket size. In future, we plan to make it a percentage of bucket memory.

### Connection Prioritization
We need a way to specify that certain DCP connections should be visited and receive a higher priority than other connections. This will allow us to make sure replication is always getting the most attention, followed by xdcr and indexing, followed by backup/restore and other integration plugins.

### Slow Connection Detection (Cursor dropping)
We need to be able to deal with connections that do not read data quickly. Since we don't allow items in an in-memory checkpoint to be purged from memory if a cursor is registered a slow connection can cause high memory overhead in the server. One proposal for this is to drop the cursor from memory and close the stream forcing the client to reconnect the stream once it is ready to receive more items. 

##All test results
* [3.0.2 results](3.0.2/overview.md)
* [4.0.0 results](4.0.0/overview.md)