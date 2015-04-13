#3.0.2 VM Based Testing

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
	| 11,000 |   181/1628   |  224/1316   |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 |  2096/62764  |  1855/40807 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 |  14170/85159 | 14957/88260 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 26816/100796 | 26566/97725 |             |                    |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.7/18.1/5 |  13.7/15/4  |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 14.8/37.7/4 |  14.9/33/3  |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 13.6/32.7/0 | 14.4/53.9/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 13.7/51.3/0 | 13.7/36.8/0 |             |                    |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|-------------|--------------------|
	|Ops/Sec | Replication    | Replication  | Replication | Replication, Views |
	|        |                | and Views    | and XDCR    | and XDCR           |
	|--------|----------------|--------------|-------------|--------------------|
	| 11,000 |  148/835/1031  | 150/834/1014 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 12,000 |  156/937/5753  | 231/940/5801 |             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 13,000 |  623/5378/7148 | 611/5688/7718|             |                    |
	|--------|----------------|--------------|-------------|--------------------|
	| 14,000 |  1975/6503/8794|2297/6628/8039|             |                    |
	|--------|----------------|--------------|-------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Views and XDCR](rep-views-xdcr.md)

### Tests with external clients

Tests with external clients were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly in the presence of different number of external clients.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|---------------|
	|Ops/Sec | 1 Client     | 5 Clients   | 10 Clients  | 25 Clients    |
	|--------|--------------|-------------|-------------|---------------|
	| 6,000  |      NA      |     NA      | 3007/78316  |1568199/3830359|
	|--------|--------------|-------------|-------------|---------------|
	| 11,000 |   444/28929  |  953/23822  | 11673/286232|     NA        |
	|--------|--------------|-------------|-------------|---------------|
	| 12,000 |  1990/46684  | 3637/95067  | 26651/469551|     NA        |
	|--------|--------------|-------------|-------------|---------------|
	| 13,000 |  5043/47999  | 6871/107210 |    NA       |     NA        |
	|--------|--------------|-------------|-------------|---------------|
	| 14,000 | 20961/83365  | 14668/152165|    NA       |     NA        |
	|--------|--------------|-------------|-------------|---------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|-------------|
	|Ops/Sec | 1 Client    | 5 Clients   | 10 Clients  | 25 Clients  |
	|--------|-------------|-------------|-------------|-------------|
	| 6,000  |     NA      |     NA      | 7.1/12.9/0  | 1.9/18.8/0  |
	|--------|-------------|-------------|-------------|-------------|
	| 11,000 | 13.6/15/6   | 13.5/24.9/2 | 11.9/21/0   |     NA      |
	|--------|-------------|-------------|-------------|-------------|
	| 12,000 | 13.7/29/0   | 14.2/26/2.5 | 12/26/5     |     NA      |
	|--------|-------------|-------------|-------------|-------------|
	| 13,000 | 15.9/34.4/1 | 14.9/29/0   |    NA       |     NA      |
	|--------|-------------|-------------|-------------|-------------|
	| 14,000 | 14.4/32.3/0 | 13.8/32/0   |    NA       |     NA      |
	|--------|-------------|-------------|-------------|-------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|--------------|-----------------|
    |Ops/Sec | 1 Client       | 5 Clients    | 10 Clients   | 25 Clients      |
    |--------|----------------|--------------|--------------|-----------------|
	| 6,000  |       NA       |      NA      | 242/861/1399 | 316/1563/66352  |
	|--------|----------------|--------------|--------------|-----------------|
	| 11,000 |  150/850/1068  | 160/780/1091 | 163/843/1503 |       NA        |
	|--------|----------------|--------------|--------------|-----------------|
	| 12,000 |  86/751/1119   | 167/853/1230 | 318/1055/4165|       NA        |
	|--------|----------------|--------------|--------------|-----------------|
	| 13,000 | 160/1145/6436  | 317/1172/6473|     NA       |       NA        |
	|--------|----------------|--------------|--------------|-----------------|
	| 14,000 | 855/5994/7723  | 569/4082/6875|     NA       |       NA        |
	|--------|----------------|--------------|--------------|-----------------|

* [Replication with 1 client](rep-1_client.md)
* [Replication with 5 clients](rep-5_clients.md)
* [Replication with 10 clients](rep-10_clients.md)
* [Replication with 25 clients](rep-25_clients.md)
