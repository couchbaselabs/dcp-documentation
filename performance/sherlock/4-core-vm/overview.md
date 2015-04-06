#Sherlock VM Based Testing

###Hardware Setup

**Server Specs:**

* Centos 6 VM
* 4 Core ()
* 8GB RAM
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
	| 11,000 |   151/1161   |  154/1059   |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 |  3134/74143  |  780/32325  |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 |  16207/92715 | 22705/98116 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 31311/116697 | 31592/114393|             |                    |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.7/17.1/6 |  13.8/18/4  |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 14.4/34/5   |  14.8/16/6  |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 14.1/49/0   | 13.5/34.9/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 13.4/41.8/0 | 12.5/44.4/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|-------------|--------------------|
	|Ops/Sec | Replication    | Replication  | Replication | Replication, Views |
	|        |                | and Views    | and XDCR    | and XDCR           |
	|--------|----------------|--------------|-------------|--------------------|
	| 11,000 |  133/835/1051  | 116/823/1009 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 12,000 |  235/933/5104  | 224/864/1066 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 13,000 | 676/5835/7955  |762/5604/7673 |             |                    |
	|--------|----------------|------------- |-------------|--------------------|
	| 14,000 | 3389/6770/9387 |4021/6946/8536|             |                    |
	|--------|----------------|--------------|-------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Viwes and XDCR](rep-views-xdcr.md)