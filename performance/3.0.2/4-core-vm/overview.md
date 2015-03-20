#3.0.2 VM Based Testing

###Hardware Setup

**Server Specs:**

* Centos 6 VM
* 4 Core ()
* 4GB RAM
* 2 Servers

**Client Specs:**

* Centos 6 VM
* 4 Core ()
* 4GB RAM
* 1 Server

###Testing

Baseline tests were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly. We consider the expected amount of items to be remaining for replication at any given time to be 15% of the incoming sets/sec to be an expected scenario. Each test uses value sizes of 1KB.

	Replication Items Remaining (Avg/Max)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|------------ |-------------|--------------------|
	| 10,000 | 43/344      |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 |             |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 105/958     |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 3091/53790  |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 |             |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 15,000 | 21387/87032 |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
	|--------|-------------|------------ |-------------|--------------------|
	| 10,000 | 6.3/8/2.7   |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 |             |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 7.5/8.4/2   |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 7.3/25/0    |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 |             |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|
	| 15,000 | 6.6/29/0    |             |             |                    |
	|--------|-------------|-------------|-------------|--------------------|


Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Viwes and XDCR](rep-views-xdcr.md)
