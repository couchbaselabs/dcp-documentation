
#UPR Project

UPR stands for "Universal Protocol for Replication" and the purpose of this project is to revamp the replication so that it can be used by all modules in the Couchbase ecosystem as well as third party applications.

###UPR High-level Details

* [UPR Overview](overview.md)
* [Transport Protocol Specification](transport-spec.md)

###Use Cases

#####VBucket Move

A cluster rebalance is made up of multiple VBucket moves. Below are links to the current (2.x) and future (3.x) procedures.

* [VBucket Move (2.x)](https://github.com/couchbaselabs/ep-engine-designs/blob/master/architecture/vbucket-move.md)
* [Vbucket Move (3.x)](vbucket-move.md)

#####Indexing

In future versions of Couchbase indexing will no longer read from disk and instead get there data from an UPR replication stream. Below are links to the current (2.x) and future (3.x) strategy.

* [Indexing (2.x)](https://github.com/couchbaselabs/ep-engine-designs/blob/master/architecture/indexing.md)
* [Indexing (3.x)](indexing.md)

#####XDCR

XDCR currently reads items from disk in order to replicate them accross wide area networks. Future versions will stream data directly to the XDCR replicators via an UPR stream. Below are links to the current (2.x) and future (3.x) strategy.

* [XDCR (3.x)](xdcr.md)

#####Consistent Views

A major feature that has been asked for by customers is the ability to support consistent views. This means that when a user does a "set" command they can immediately query the view and expect to see the data that was set in the view. Below are links to the current (2.x) and future (3.x) plan.

* [Consistent Views (3.x)](ryow.md)

#####Third-Party

The current tap implementation is too difficult to use and laks the features needed to build third-party applications. Below are links to use cases for how UPR can be used to succesfully build new applications.

* [Creating an UPR Session](upr-session.md)
* Streaming all data from a cluster
* [Using filters on a tap stream](https://docs.google.com/document/d/1K6RGIxVMygQNUwu3fSn3HiSTSHL_iISxTN6fiZYVn5U)

#####Backwards Compatibility

we need to support upgrades from 2.x to 3.x. The link below describes how the upgrade process will work.

* [Upgrade (2.x) to (3.x)](upgrade.md)


###Deprecated Documents

* [Mutation Queues](deprecated/mutation_queues.md)