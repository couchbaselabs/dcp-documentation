#Sherlock VM Based Testing

##Hardware Setup

**Server Specs:**

* Centos 6 VM
* 8 Core ()
* 8GB RAM
* 2 Servers
* DGM scenario

**Client Specs:**

* Centos 6 VM
* 4 Core ()
* 4GB RAM
* 1 Server

##Testing

###Baseline Tests

Baseline tests were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly. We consider the expected amount of items to be remaining for replication at any given time to be 15% of the incoming sets/sec to be an expected scenario. Each test uses value sizes of 1KB. Note that stats are collected across all nodes. For example this means and 1000 items remaining on average means means 500 items reminaing on each node given a two node cluster.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication  | Replication | Replication | Replication, Views |
	|        |              | and Views   | and XDCR    | and XDCR           |
	|--------|--------------|-------------|-------------|--------------------|
	| 11,000 | 736/30941    |  921/33853  |     NT      |       NT           |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 | 1910/37624   |  2342/46154 |     NT      |       NT           |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 | 5284/71019   |  8913/99744 |     NT      |       NT           |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 14628/113448 |      NT     |     NT      |       NT           |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.7/43.1/2 |13.5/18.8/4.7|     NT      |       NT           |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 |14.5/28.3/2.3| 14.4/21.2/3 |     NT      |       NT           |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 14.8/30/0   | 14.6/33.5/0 |     NT      |       NT           |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 14.8/33.8/0 |     NT      |     NT      |       NT           |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|----------------|--------------------|
	|Ops/Sec | Replication    | Replication  | Replication    | Replication, Views |
	|        |                | and Views    | and XDCR       | and XDCR           |
	|--------|----------------|--------------|----------------|--------------------|
	| 11,000 | 43/693/1020    | 39/648/1015  |      NT        |       NT           |
	|--------|----------------|--------------|----------------|--------------------|
	| 12,000 | 55/791/3080    | 34/751/2950  |      NT        |       NT           |
	|--------|----------------|--------------|----------------|--------------------|
	| 13,000 | 99/945/5705    | 105/1067/6418|      NT        |       NT           |
	|--------|----------------|--------------|----------------|--------------------|
	| 14,000 | 369/5185/7066  |     NT       |      NT        |       NT           |
	|--------|----------------|--------------|----------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)

### Tests with external clients

Tests with external clients were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly in the presence of different number of external clients.

	Replication Items Remaining (Avg/Max)
    (NUM) indicates steady state avg

	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	|Ops/Sec | 1 Client     | 5 Clients   | 10 Clients  | 25 Clients   | 50 Clients   | 100 Clients  |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 11,000 |  539/28360   | 740/29650   | 1320/46611  | 1638/51146   | 42465/427730 | 35500/272604 |
	|        |              |             |             |              | (1225)       |  (1182)      |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 12,000 |  1109/28191  | 1856/38207  | 1707/39140  | 8341/161846  |     NT       |     NT       |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 13,000 | 5181/63455   | 5177/46633  | 2505/34838  |     NT       |     NT       |     NT       |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 14,000 | 8889/124293  |     NT      |     NT      |     NT       |     NT       |     NT       |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	|Ops/Sec | 1 Client    | 5 Clients   | 10 Clients  | 25 Clients  | 50 Clients  | 100 Clients |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 11,000 | 13.5/19/4.6 | 13.2/16.8/3 | 13.2/20.6/2 | 13.3/20/2   |  10.5/18/0  | 11.3/22/0   |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 12,000 | 14/20.4/1   | 14.5/29/2   | 14.6/32/6   | 13/19.6/0   |     NT      |     NT      |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 13,000 | 15.2/39.3/0 | 15.2/32/0   | 14.5/23/2   |     NT      |     NT      |     NT      |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 14,000 | 14.8/31/0   |     NT      |      NT     |     NT      |     NT      |     NT      |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
    |Ops/Sec | 1 Client       | 5 Clients    | 10 Clients   | 25 Clients      |  50 Clients     |  100 Clients    |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 11,000 | 63/626/997     | 71/606/1020  | 70/632/1030  | 106/630/1034    |  446/5112/33488 |  456/4706/33580 |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 12,000 | 138/686/1027   | 44/732/3677  | 52/659/1367  | 168/903/4976    |       NT        |       NT        |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 13,000 | 73/900/5693    | 138/983/6066 | 202/830/2324 |       NT        |       NT        |       NT        |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 14,000 | 119/1083/6684  |      NT      |     NT       |       NT        |       NT        |       NT        |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|

* [Replication with 1 client](rep-1_client.md)
* [Replication with 5 clients](rep-5_clients.md)
* [Replication with 10 clients](rep-10_clients.md)
* [Replication with 25 clients](rep-25_clients.md)
* [Replication with 50 clients](rep-50_clients.md)
* [Replication with 100 clients](rep-100_clients.md)
