
#UPR Overview (3.x)

###Motivation

The current intra-cluster replication protocol for Couchbase was designed for the needs of a clustered key-value store. As the system is designed for high throughput and low latency, it presents special challenges for reliability and consistency in the face of network and server failures, as well as slow components and occasionally connected processes. As we've added functionality like indexing and cross datacenter replication, as well as integration with other systems, like Hadoop and ElasticSearch, incremental backup, etc, these challenges become harder and more relevant and we need a well defined protocol that addresses the needs elasticity/rebalance, replicating changes and state to local cluster members and indexers and inter-cluster systems, such that the system continues to be highly available and consistent regardless of minor failures, and the ability to recover or failover to a different datacenter in cases of major failures.

Couchbase is designed to be a strongly-consistent system, such that updates, edits and deletes to documents are immediately and predictably visible to all clients of the system. Yet for performance and scalability reasons, the system uses asynchronous replication to prevent blocking and prevent slow or disconnected components from impacting the performance of the system as a whole. This means the system components must be able to reconnect to the old or a new master after either the failure of the component, or the master, and quickly determine the differences in state, resolve those differences, and resume normal operation, such that consistency and availability are achieved with little impact on performance.

###Terminology

**Application Client** - This is a normal client that performance reads, writes, updates, deletions and queries to the server cluster, usually for an interactive web application.

**UPR Client** - This is a special client that streams data from one or more Couchbase server nodes, for purposes of intra-cluster replication (to be a backup in case the master server fails), indexing (to answer queries in aggregate about the data in the whole cluster), XDCR (replicate data from one cluster to another cluster, usually located in a separate datacenter), incremental backup, and any 3rd party component that wants to index, monitor, or analyze Couchbase data in near real time, or in batch mode on a schedule.

**Server** - This is a master or replica node that serves as the network storage component of a cluster. For a given partition, only one node can be master in the cluster. If that node fails or becomes unresponsive, the cluster will select a replica node to become the new master.

**Partition (sometimes called vBucket)** - Couchbase splits the key space into a fixed amount of partitions, usually 1024. That is, keys are deterministically assigned to a partition, and partitions are assigned to nodes to balance load across the cluster.
Trond comments: renaming the field from the well established name will most likely make it harder to find information why it was created. Please see Dustin's blog post describing the vbuckets.

**Sequence Number** - Each mutation (a key takes on a value or is deleted), that occurs on a partition is assigned a number, which should be strictly increasing as events are assigned numbers (there is no harm in skipping numbers, but they must increase), that can be used to order that event against other mutations within the same partition.

This does not give a cluster-wide ordering of events, but it does enable processes watching events on a partition to resume where they left off after a disconnect.

**Partition Version** - A UUID, Sequence Number pair associated with a partition. A new version is assigned to a partition by its new master node any time there may have been a history branch. The UUID is a randomly generated number, and the Sequence Number is the Sequence that that partition last processed at the time the Version was created.

**History Branch** - Whenever a node becomes the master node for a partition in the event of a failover or uncontrolled shutdown and restart, if it was not the farthest ahead of all processes watching events on that partition and starts taking mutations, it is possible it will reuse sequence numbers that other processes have already seen on this partition. This may be a History Branch, and the new master MUST assign the partition a new Partition Version, so that UPR clients in the distributed system can recognize when they may have been ahead of the new master, and rollback changes at the point this happened in the stream.
If there is a controlled handover from an old master to a new master, then the sequence history cannot have branches, and there is no need to assign the partition being handed off a new version. This happens in the case of a rebalance for elasticity (add or remove a node) or a swap rebalance in the case of a upgrade (new version of server added to cluster, old version removed).

**Snapshot** - In order to send a client a consistent picture of the data it has, the server will take a snapshot of the state of its disk write queue or the state of its storage, depending on where it needs to read from to satisfy the client’s current requests. This snapshot should represent the exact state of the mutations it contains at the time it was taken. Using this snapshot, the server should be able to send the items that existed at the point in time the snapshot was taken, and *only* those items, in the state they were in when the snapshot was taken.
Snapshots do not imply that everything is locked or copied into a new structure. In the current Couchbase storage subsystem, snapshots are essentially “free”, the only cost is when a file is copy compacted to remove garbage and wasted space, the old file cannot be freed until all snapshot holders have released the old file. It’s also possible to “kick” a snapshot holder if the system determines the holder of the snapshot is taking too long. UPR clients that are kicked can reconnect and a new snapshot will be obtained, allowing it to restart from where it left off.

**Rollback Point** - The server will use the Failover Log to find the first possible History Branch between the last time a client was receiving mutations for a partition and now. The sequence number of that History Branch is the Rollback Point that is sent to the client.

**Partition Stream** - A grouping of messages related to receiving mutations for a specific partition, this includes Mutation/Deletion/Expiration messages and Snapshot Marker messages. The transport layer is able to provide a way to separate and multiplex multiple streams of information for different partitions.
All messages between Snapshot Markers messages are considered to be one snapshot. A snapshot will only contain the recent update for any given key within the snapshot window. It may require several complete snapshots to get the current version of the document.

**Failover Log** - A (possibly capped) list of previous known Partition Versions for a partition. If a client connects to a server and was previously connected to a different version of a partition than that server is currently working with, this list is used to find a Rollback Point.

**Mutation** - The value a key points to has changed (create, update, delete, expire)

###Scope

The new intra-cluster replication protocol should provide an ordering of document state in a partition, even across cluster topology changes. This allows a client of the protocol to correctly resume from where it left off, avoiding unnecessary network load or index computation.
The protocol should also specify how to handle cases where components in the cluster disagree on the history of mutations in a partition.

###Protocol

####Message Types

#####Open Connection

TODO

#####Add Stream

TODO

#####Close Stream

TODO

#####Flush

TODO

#####Set vBucket State

TODO

#####Stream Request
A client is requesting the event stream for a partition since some Sequence Number. The client sends its known Partition Versions, with the most recent Partition Version it was connected to first.

If the client's most recently connected-to Partition Version is not the current one on the server, the server will determine how far the client must roll back to sync up with the now current version of this partition.
Clients that try to stay connected as often as possible likely only need to keep the most recent Partition Version for the partitions they are watching, whereas clients that connect rarely (for example, a backup process), may want to keep several Partition Versions, which allows for finding a better rollback point than zero when multiple history branches have occurred since the last time the client was connected.
It is undefined behaviour to request more than one stream concurrently for a partition. Clients that need to do this should open another connection.

**Fields**

* Partition to stream from
* Sequence number to stream since.
* A sequence number to end on or after (the stream will end after sending the snapshot that contains this sequence. It will *not* send a partial snapshot however, so the stream *may* contain items after this sequence)
* A Partition Version (pair of UUID and the Sequence at which that version was created) from the client's known partition history

#####Stream Request Response (ROLLBACK)
There has been a history branch since the client was last connected, so the server responds with the sequence number to roll back at least to.

The client must roll back data such that its current state is as of the end of a snapshot at a sequence less than or equal to this sequence. Once client has rolled back and repaired its state, it will reissue the Stream Request message with its newly repaired state reflected in the request.

**Fields**

* Sequence number to roll back (at least) to.

#####Stream Request Response (OK)

There was no history branch since the client last connected, or the client is requesting the entire history (change since sequence zero). The server will now start streaming the requested items.

**Fields**

*None*

#####Stream Request Response (NOT_MY_PARTITION)

This server cannot serve items for this partition

#####Stream End

When the full snapshot that contains the sequence number to end at was sent, a Stream End message will be sent. It contains a flag to indicate whether the end is due to an error or not.

**Fields**

* Flag

#####Mutation

A Mutation always has a key, and the value the key now points to.

**Fields**

* Partition/vBucket Id
* Sequence number
* Revision Sequence (“XDCR sequence #”)
* Key
* Value
* CAS
* Expiration
* Flags
* Lock time

#####Deletion

A Deletion always has a key, but no value.

**Fields**

* Partition/vBucket Id
* Sequence number
* Revision Sequence (“XDCR sequence #”)
* Key

#####Expiration

A Expiration has the same fields as a Deletion.

**Fields**

* Partition/vBucket Id
* Sequence number
* Revision Sequence (“XDCR sequence #”)
* Key

#####Snapshot Marker

This message will be sent after all items in a snapshot have been sent, to let the client know the snapshot is complete.

#####Failover Log Request

The client is asking the server for its Failover Log for a partition. When a client connects to the server the first time it needs to know about the Partition Version. It will make a Failover Log Request in order to get the latest one.

**Fields**

* Partition/vBucket Id

#####Failover Log Response

**Fields**

* List of Partition Versions in the server’s Failover Log

###Behaviors

#####The Failover Log

When a replica requests partition streams it should ask the master for a list of all Partition Versions that the upstream node knows about (for example, in chained replication, a replica would be asking for the list from the replica its pulling from).

The replica will keep that list in case it becomes master and needs to send the Failover Log to other replicas in the future.

When a node takes over as master for a partition, if it creates a new version for that partition (because it was not able to take over cleanly), it adds an entry to the Partition Version list that it maintains. This is also called the Failover Log, since it will have an entry for each failover, or unclean takeover, event.

#####Finding a rollback point

When a client connects and has not been most recently connected to the current version of the partition it is requesting, the server should search for the provided version in its Failover Log. If it finds an entry for that version, it should send a message to the client requesting it roll back at least to the sequence that begins the version after this version in the Failover Log.

If none of the clients known versions can be found in the server's Failover Log, it must request that the client roll back to zero, that is, discard all its data for this partition and start from scratch.

![Figure 1](images/overview_1.png)

###Client Flow

1. To begin receiving items (mutations/deletions/expirations) for a partition, a client must send a Stream Request message, indicating the partition it wants to receive items for, the last sequence it received the last time it was connected (or 0 otherwise), and the identifier for the last version of the partition it received items from.

2. The client waits for a response.
  * The client may receive an OK response, notifying it that a stream has been started on which it will receive item and snapshot messages.
  * Or, the client may receive a ROLLBACK response, notifying it that there has been a history branch on this partition since it was last connected, and a sequence that must be rolled back at least to. The client may roll back to a sequence that is earlier than that sequence. To ensure consistency and nothing gets missed, the client must rollback to snapshot boundary, the first before or at the rollback point. `The server will not send changes after a ROLLBACK response, and the client should send another Stream Request after it has completed its rollback operation if it still wants items for this partition. If the client is keeping a Failover Log (list of versions its been connected to in the past, usually because client is a replica), it MUST discard any entries with a sequence number higher than the sequence it rolled back to.

3. The client got an OK response, and begins listening for messages on the stream identified by the OK response message.

4. The client receives items, indicating that those items, until a Snapshot Marker message is received, will contain at most one version of any particular key, which will be the latest version of that key as of the sequence number included in the Snapshot Begin message.

5. The client receives a Snapshot Marker message, indicating that it has now been sent the latest updates for all keys that were updated between the sequence it requested in its Stream Request and the highest sequence number in this snapshot. The client should now wait for more items or a Stream End message (#4)

###Server Flow

1. A client asks for items for a partition, and includes the version(s) of the partition that it has been recently connected to.
  * If the client's most recently connected version is not the version the server is serving, look up the versions the client provided in the server's copy of the Failover Log. Send a ROLLBACK message with the sequence of the entry after latest entry found. Once the client has rolled back it should send a new request, because the server cannot know how far the client will roll back. The client MUST roll back at least to the sequence sent in the rollback message, but MAY roll back farther.

2. Otherwise, snapshot items around the client's current sequence, and send the items.
  * If the client is behind (that is the sequence it's currently reading from is already persisted to disk), this can be a snapshot only of storage. Once the client gets ahead of storage this will likely only need to be a snapshot of unpersisted items.

3. Send a Snapshot Marker message, and acquire a new snapshot, above the last key sent (Back to #2), or send a Stream End message.
