#DCP Documentation

The Database Change Protocol (DCP) a protocol used by Couchbase for moving large amounts of data. The protocol intended for use in replication, indexing, and third-patry integrations where moving a lot of data quickly and efficiently is neccessary.

**Note:** This page is currently being worked on and links will be added once documentation is deemed to be official. Please refer to the [old contents page](deprecated/README.md) in the meantime.

* Overview
* Concepts
* Terminology
* DCP Architecture
	* [Protocol](documentation/protocol.md)
	* Protocol Flow
	* Failure Scenarios
	* Flow Control
	* Dead Connection Detection
	* [Notifier Connections](documentation/notifier-connection.md)
* Developing Clients
	* Building a simple client
	* Handling rollbacks
	* Handling topology changes
	* Flow control best practices
	* Detecting dead connections
	* Setting connection priority
* Core Server Architecture
	* [Rebalance](documentation/rebalance.md)
	* XDCR Integration
	* View Engine Integration
	* Backup Tool
	* [Upgrade (2.x to 3.x)](documentation/upgrade.md)
* [Future Work](documentation/future-work.md)