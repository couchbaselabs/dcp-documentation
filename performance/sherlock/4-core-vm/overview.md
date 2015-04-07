#Sherlock VM Based Testing

##Hardware Setup

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

##Testing

###Baseline Tests

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
	|--------|----------------|--------------|-------------|--------------------|
	| 14,000 | 3389/6770/9387 |4021/6946/8536|             |                    |
	|--------|----------------|--------------|-------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Views and XDCR](rep-views-xdcr.md)

### Tests with external clients

Tests with external clients were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly in the presence of different number of external clients.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|--------------------|
	|Ops/Sec | 1 Client     | 5 Clients   | 10 Clients  | 25 Clients         |
	|--------|--------------|-------------|-------------|--------------------|
	| 11,000 |   236/1550   |  778/3942   |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 |  1286/36850  |  2140/36343 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 | 30897/120404 | 11428/61643 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 22075/98116  | 26990/161580|             |                    |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | 1 Client    | 5 Clients   | 10 Clients  | 25 Clients         |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.7/16/6.4 | 13.4/15.3/5 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 14.9/28/5.3 |  14.1/18/4  |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 12.5/29/0   | 13.6/24.4/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 13.5/34.9/0 | 13.3/43.1/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|-------------|--------------------|
    |Ops/Sec | 1 Client       | 5 Clients    | 10 Clients  | 25 Clients         |
	|--------|----------------|--------------|-------------|--------------------|
	| 11,000 |  84/830/1006   | 234/714/1061 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 12,000 |  90/800/1084   | 273/916/1739 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 13,000 | 557/5246/7544  |916/3445/7249 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 14,000 | 763/5604/7673  |1613/6454/7901|             |                    |
	|--------|----------------|--------------|-------------|--------------------|

* [Replication with 1 client](rep-1_client)
* [Replication with 5 clients](rep-5_clients)
* [Replication with 10 clients](rep-10_clients)
* [Replication with 25 clients](rep-25_clients)