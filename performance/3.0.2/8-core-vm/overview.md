#3.0.2 VM Based Testing

###Hardware Setup

**Server Specs:**

* Centos 6 VM
* 8 Core ()
* 4GB RAM
* 2 Servers

**Client Specs:**

* Centos 6 VM
* 4 Core ()
* 4GB RAM
* 1 Server

###Testing

Baseline tests were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly. We consider the expected amount of items to be remaining for replication at any given time to be 15% of the incoming sets/sec to be an expected scenario. Each test uses value sizes of 1KB. Note that stats are collected across all nodes. For example this means and 1000 items remaining on average means means 500 items reminaing on each node given a two node cluster.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication  | Replication | Replication | Replication, Views |
	|        |              | and Views   | and XDCR    | and XDCR           |
	|--------|--------------|-------------|-------------|--------------------|
	| 10,000 |   411/3930   |  2218/31222 |  2534/32355 |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 11,000 |   923/24920  | 11991/61259 | 15568/60161 |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 |  1380/28284  | Not Tested  | Not Tested  |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 |  6785/42771  | Not Tested  | Not Tested  |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 25802/260762 | Not Tested  | Not Tested  |                    |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|------------ |-------------|--------------------|
	| 10,000 | 12.3/16.8/3 | 11.4/17.1/4 | 12.0/17.0/4 |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.6/18.4/5 | 11.9/30.6/0 | 11.8/21.0/0 |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 14.6/30.2/0 | Not Tested  | Not Tested  |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 15.3/28.5/0 | Not Tested  | Not Tested  |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 14.0/30.5/0 | Not Tested  | Not Tested  |                    |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|----------------|----------------|--------------------|
	|Ops/Sec |  Replication   |  Replication   | Replication    | Replication, Views |
	|        |                |  and Views     | and XDCR       | and XDCR           |
	|--------|----------------|----------------|----------------|--------------------|
	| 10,000 |  82/529/9922   | 306/1046/3566  | 387/960/2516   |                    |
	|--------|----------------|----------------|----------------|--------------------|
	| 11,000 | 169/636/1070   | 1542/4075/8575 | 1645/3433/6972 |                    |
	|--------|----------------|----------------|----------------|--------------------|
	| 12,000 |  84/601/1155   |   Not Tested   |   Not Tested   |                    |
	|--------|----------------|----------------|----------------|--------------------|
	| 13,000 | 307/1689/6158  |   Not Tested   |   Not Tested   |                    |
	|--------|----------------|----------------|----------------|--------------------|
	| 14,000 | 2463/6272/9619 |   Not Tested   |   Not Tested   |                    |
	|--------|----------------|----------------|----------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Viwes and XDCR](rep-views-xdcr.md)
