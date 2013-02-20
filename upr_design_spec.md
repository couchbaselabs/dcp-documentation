Unified Protocol for Replication Design Specification
=================================================

# INTRODUCTION

## Purpose of this Document

This document describes the software design and implementation to support the Unified Protocol for Replication (UPR) inside Couchbase Server as both a client and server, and a generic client implementation.

## Motivation

UPR is a mechanism for efficiently streaming mutations from a Master partition server to replicas/slaves, indexers (Incremental Map/Reduce, GeoSpatial, Elastic Search), XDCR, incremental backup, and third party systems.

It's intention is to allow consistency of data and work efficiently and reliably with the Couchbase networked distributed system and all problems associated, including network slowness and disconnects, slow servers, slow clients, server rebalance/partition relocation, server failover, server crashes and data loss, client crashes and data loss.

## Scope of this Document

This document will describe the high level algorithms, data structures, and components necessary to implement UPR. It will not specify byte level protocol implementation details, or required changes in existing code necessary to support the implementation.

# SYSTEM ARCHITECTURAL DESIGN

## Overview of Modules and Components

### Couchbase Server

This is a single node of a Couchbase installation. It contains an implementation of all components.

### memcached/ep-engine

This is the networked data serving component of Couchbase Server.

### Couchstore

This is the storage engine component of Couchbase Server. It arranges a record of each document by update sequence ID, a  monotonically increasing value, in a btree. Each document mutation is a assigned a new sequence number, for existing documents in the store, the old sequence value is deleted. This allows servers to quickly scan recent documents and deletions persisted since a particular sequence number, or from zero if wants all changes.

### Write Queue/Immutable Tree

These are per-partition, in-memory immutable tree ordered by update seq that allows for snapshotting unpersisted items by multiple concurrent consumers (clients and flusher), allowing each an independent snaphot while also allowing concurrent updates. The immutable btree will allow for relatively low memory consumption, fast reads and writes, and automatic freeing of unused tree nodes and values.

The tree nodes link to immutable data and are ref counted, with each node pointed to by one or more parent snapshots, and the root ref count is incremented by each partition write queue manager. 

Creating a snapshot is simply incrementing the root once and handing it off to the snapshot user. When all users of that root/snapshot are done, including the write queue manager, the ref count of the root falls to zero and the node is freed, and any child nodes/values it points have their ref count decremented, and if they fall to zero, are also freed. This happens recursively to immediately free any unreferenced subtrees and values.

Possible library we can use, from the LLVM project:

<http://llvm.org/docs/ProgrammersManual.html#llvm-adt-immutableset-h>

<http://llvm.org/docs/doxygen/html/ImmutableSet_8h.html>

### Immutable Values

Values that are assigned to the bucket/partition hashtable are immutable and ref counted. The hash table record itself is not immutable and can be assigned a new immutable values.

When the value is retrieved from the hashtable, the hash table record is locked, and the value the ref count is incremented, and then the record is unlocked. When the value is no longer referenced by the hash table, write queue or any other process/thread in the server, it is automatically freed as it's ref count falls to zero.

### Failover Log

The failover log tracks a history of failover events so that UPR clients can find a safe sequence number to stream a new partition snapshot with a guarantee of not losing any changes. A failover event, and new failover ID, is when a master is started and it's unknown if the master was completely in sync with all mutations.

If the master starts up but doesn't know if a clean shutdown occured, a new failover log entry is generated, even if there there is no failover to a new machine.

The failover log is a list of UUIDs/sequence pairs and each is marker of high sequence number of last complete snapshot the new master persisted from the old master. If a new log entry has a sequence number lower than existing entries, those exsting entries are removed from the log.

Clients will persist the failover log whenever it completely persists a snapshot, as well as the high sequence it most recently persisted.

When a client connects to a server and before it stream a new partition snapshot, it will compare it's failover log with the server to find highest safe sequence number it can stream. It will compare it's log entries and find the most common entry. It will then use the smaller of the sequence number between it's copy of the failover entry and the server's.

### Client and Ringbuffer

The client is a process that can talk to a server and streaming changes from it. It will have a ring buffer of the latest key mutations to provide a way to rollback changes if it is ahead the of a server after a failover or crash. The ring buffer size should be configurable. If the client needs to use a sequence that's lower than the latest sequence it's seen, it will use the ring buffer to undo any changes that might be out of sync with the master before streaming from the common sequence number.

### Barrier Cookies

Barrier cookies are a way a for a indexer to ensure consistency storage quickly and at multiple points in time, so that UPR indexer clients can service multiple concurrent query clients demanding consistent results. UPR clients generate unique IDs and send them to the server, and the server sends them back once all mutations since that point in time have been streamed to the clients.

# NETWORK INTERFACE DESIGN AND FLOW

## How KV's mutations flow into ep-engine and write queue
1. A mutation (insert,update,delete) request from an memcached client occurs.
2. The hash table entry where the item will reside is locked. This might be granualur to the entry itself, or something higher like the whole hashtable.
3. If a CAS operation, the mutation is either accepted or rejected.
4. If the mutation is accepted, the partition seq is incremented and assigned to the mutation.
5. The mutation is placed into the locked hash table entry with a new seq item, and then locks the write queue.
6. If an existing item, the old seq from the hashtable entry is used to delete the previous by seq entry from the write queue.
7. If a deleted item, the item in the hash table is kept around, and will be until the deletion is persisted.
8. The hash table partition is unlocked.
9. The client gets a response that the item was accepted.

## How items in the partition write queue flow to couchstore

1. The flusher thread locks the write queue and increments the counter on the root node, giving it a snapshot of the partition, then unlocks the queue.
2. The flusher then walks the tree and copies the metadata and data into parallel arrays and hands them to Couchstore.
3. Once the all the items are successfully persisted, the queue is then locked and all items persisted are deleted from the tree.
4. Any deletions persisted are then removed from the hashtable, if the items in the hashtable still have the same sequence number and haven't been mutated again.


## How UPR clients and ep-engine handshake

1. An UPR client makes a connection.
2. The UPR client sends the partition numbers for the partitions it wants to stream.
3. ep-engine then sends the failover log for each partition it owns
4. For each partition it wishes to stream, the client sends to the server partition number. expected state of active or replica, and latest failover ID and start seq number.
5. For any partition the server doesn't own, is in the wrong state, or has too high a seq # or the wrong current failover ID, it sends a error to the client. The client is expected to disconnect, reload the client map and start back on step 1.

## How ep-engine sends snapshots to UPR clients

6. The server will send a complete snapshot of each partition, one at a time. It will send the snapshot of each partition with mutations in partition ID order.
6. For each partition with mutations. The server will first send a partition start message, which includes the partition ID it will send, then stream all changes, in sequence order with the sequence number. The server may send duplicates for  items, but it will never omit an item.
7. If a partition has no mutations, the server will send no partition start message, it will skip it.
7. When it's checked or sent all partitions, the server will send a "completed all partitions message".
7. The server then loops around and streams any new mutations for the registered partitions. If there are no mutations, the server will pause until there are mutations, or until it gets barrier cookie from the client.

## Barrier cookies, how clients request faster/multiple complete snapshots

8. If the client wishes to get a faster consistent view at a point in time, or multiple consistent snapshots for it's own clients, it sends to the server a "barrier cookie", a short text string that is unique per UPR connection.
9. The server then notes the new barrier cookie and keeps any old barrier cookie(s) around that is hasn't yet sent.
9. When the barrier cookie order is received and all partitions have been streamed after the barrier cookie is added, it sends back the barrier cooke so the client knows it's seen a consistent snapshot for all partitions.
10. The server forgets a barrier after sending it back.
12. When the client no longer wishes to stream changes, it breaks the connection.
13. If a partition the client is streaming becomes inactive or replica, the server returns a not "not my partition error" and the client reloads the partition map and re-handshakes.

## How ep-engine streams a consistent snapshot for a partition

1. Each partition has it's own in-memory write queue, which is an immutable tree ordered by sequence ID.
2. For each UPR connection, ep-engine maintains a list of high seq numbers for each partition.
2. When sending a stream for a partition, it grabs the root of the tree and increments the ref counter for the root. This prevents the root for being freed if it's concurrently updated.
2. It notes the high seq number for that snapshot.
3. If the starting seq number is on disk, not memory, it walks the Couchstore update sequence btree.
4. For any non-deleted item with an in-memory update seq that's lower than high seq number, it grabs it out of memory if resident, otherwise it loads it from disk.
5. It sends each item's metadata and body to the client.
6. Once it's finished walking the disk btree, it then walks the in-memory by seq tree, sending each items metadata and body to the client.

## How ep-engine tracks barrier cookies

1. For each UPR connection, ep-engine maintains a fifo queue of barrier markers and number of times it's cycled through all the partition.
2. When ep-engine gets a barrier cookie, it records the number of cycles it's completed and the partition it's currently streaming. This find the point after where it should send back the barrier cookie to the client. It places this into the barrier queue. 
3. Every time ep-engine completes a cycle for all partitions, it increments the cycle counter.
4. After each partition snapshot is completed or skipped it checks the barrier queue to see if it's streamed every partition since the barrier was added. It does this by checking if the number of cycles and the next partition are greater than the barrier marker. If so, it sends the barrier cookie back to the client and pops it out of the queue.
5. If there are not more mutations to stream, it sends back all barrier cookies.

## How clients roll back changes when seeing a new failover ID

1. Each client maintains a ring buffer of recent mutations, and the last complete snapshot seq number it processed for each partition as well as the last failover ID from that snapshot. The ring buffer should be persisted before mutations are applied to persistence, unless the client also has the ability to natively rollback changes. 
2. A client requests list of failover logs from the server for each partition.
3. It determine the starting sequence by scanning the for the most recent common failover ID in it's stored failover log, and it takes the smaller of it's own failover entry sequence or server's.
4. It rolls back it's own changes to the starting sequence number, by requesting the latest document for each higher seq in its ring buffer, and processing it. If the document doesn't exist, it deletes the document from it's state.
5. If needs to rollback to a sequence that's lower than it's ring buffer contents, it sets the starting sequence number to 0.
5. It removes the higher seq item IDs from the ring buffer.
6. It asks the server for a snapshot of changes using the start sequence number, and applies them to the ring buffer and it's state.
7. When done with the snapshot, it records the failover ID in it's state and the snapshot high seq.

## How servers and replicas record the failover log and add new entries

1. A replica requests the failover log from the server. It rolls back any changes like a normal client if it gets ahead of the master.
2. It gets a full snapshot like a normal client.
3. For each snapshot it receives and persists, it persists the high snapshot seq and the failover log.
2. If there is a smooth handover, it becomes the new master, and it preserves the failover log as is.
1. If there is a failover to the replica, it generates a new failover ID and pairs it with the last snapshot seq, and adds it to the front of the failover log.
2. If the new failover sequence is lower than any entries in the failover log, those entries are removed from the log.
1. It is now ready to serve as master, other replicas can stream changes from it.
1. As it persist new mutations, it updates the last failover ID with the high seq.


## How to maintain backwards compatibility with existing TAP replicas/masters.

TODO: it's possible we can add a client adaptor that converts UPR protocol into 2.0 TAP protocol?

ADDITIONAL MATERIAL
