# Unified Protocol for Replication (UPR)

 * status: PROPOSAL / DRAFT
 * latest: <https://github.com/couchbaselabs/cbupr/blob/master/spec.md>

## Summary

Couchbase Unified Protocol for Replication (UPR, or "upper") is the
proposed replication networking protocol for Couchbase "2.next" It
addresses replication stream restartability.

UPR is an application of the existing TAP protocol, adding new message
types and parameters, plus some additional required behavior by clients
and servers.

## Motivation

UPR will provide restartable, incremental data replication, facilitating
things like:

 * Indexing, especially distributed indexing.
 * Incremental backup.
 * Intra-cluster replication (w/ fewer backfills)
 * Inter-cluster replication (XDCR)
 * Integration with other data systems (Hadoop, ElasticSearch, other
   OLAP and ETL software.)
 * Persistence (even though persistence may not be an UPR consumer, it
   will be helpful to think of it as an UPR conforming replica, as it
   has to play by pretty much all of the same rules)

UPR addresses these multiple replication needs with a single unified,
synchronization model and networking protocol.

The TAP checkpoint protocol used in Couchbase 1.8.x and 2.0.0 is
insufficient because checkpoints are ephemeral. That is, it does not
handle clients that have been disconnected a long time.

The existing XDCR protocol (Couchbase 2.0.0, HTTP/REST-based) is also
insufficient for performance reasons.

This document will eventually become a detailed protocol
specification.  This document and the steps of reviewing and approving
it are meant to help the team think through difficult cases before we
make code changes.  With a well-specified protocol, separate
teams should be able to concurrently implement clients, servers,
orchestrators, tools, 3rd party integrations, test suites, etc.

Protocol changes will happen "change controlled" updates to this
spec so that teams can work without ambiguity.

## Previous writeups, proposals, and other related links

 * [Wiki: Indexing and Querying links][iql]
 * [Wiki: Checkpointing and Incremental Replication Through
    TAP][tapcheck]
 * [Indexing Performance (Dustin)][indexingperf]
 * [TAP2013 (Aaron)][tap2013aaron]

[iql]: http://hub.internal.couchbase.com/confluence/display/PM/3.0+-+Indexing+and+Querying+links
[tapcheck]: http://hub.internal.couchbase.com/confluence/display/cbeng/Checkpointing+and+Incremental+Replication+Through+TAP
[indexingperf]: http://cbugg.hq.couchbase.com/api/bug/bug-128/attachments/att-bug-128-FljPEvAB/indexing.pdf
[tap2013aaron]: http://drop.crate.im/tap2013.html

## Terminology

### Partition is the new VBucket

In this document, "partition" is used in place of "vBucket". We favor
the word "partition" in order to be less confusing for newcomers.

### Logical objects / properties

The following are important logical "objects" or concepts in the
protocol. Real client or server implementations might not use these
names or have a 1:1 physical representation of these concepts.

#### Bucket

A bucket has:

* Bucket name (like "default" or "product-catalog").
* Bucket version (new for UPR).
* A bucket has 1024 partitions.

#### Partition

A partition has:

* Partition ID (0 to 1023).
* Partition state (active, replica, pending, or dead).
* Partition Failover History (new for UPR).
    * This is a sequence of Partition Failover Records.

#### Sequence Number

A sequence number is a 48-bit unsigned number starting from 0 and is
used to identify and order mutations within a partition.

*All* messages will be marked by a sequence number, however messages that
are not mutations will not have *their own* sequence numbers, but the
sequence number of the most recent mutation at the time of the event.

Unlike in Couchbase 2.0.0 where sequence numbers are updated at
persistence flushing time by the Couchstore library, in UPR the
sequence numbers must be assigned in memory, as mutations come in.

#### Changes Stream

A changes stream is a the sequence of messages sent by an UPR server to
an UPR client for a partition. It is mostly key-value item mutations,
but will also contain other message types.

#### Mutation

A Mutation is a key-value create/update/delete.

#### Snapshot Markers and Deduplication

A snapshot marker is a message in a changes stream which conveys a
guarantee that that client has been sent the latest version of all keys
that have existed at that sequence or before. The client effectively has
seen a snapshot of the data in the partition at that sequence.

Without snapshot markers, this is *not* guaranteed because an UPR server
*may* perform **deduplication** in the changes stream. That is, if a
mutation arrives that obsoletes another one (it is an update or delete
of an existing key), the obsoleted mutation *may* be omitted from
outgoing changes streams.

Replicas *must* remember the last snapshot marker they have seen.

When reconnecting after a failover clients *must* roll back onto a
snapshot marker that it received before the sequence the new master took
over at and therefore need to keep some history of received snapshot
markers.

#### Partition Failover Record

A Partition Failover Record is a record of a server becoming the
master for a partition in a situation where it cannot guarantee it
was entirely caught up with the previous master.

If a node is restarted but does not shut down cleanly, it must create a
Partition Failover Record. This can be understood as if the server's
disk storage were a replica that was not caught up when it was failed
over to.

If it can be determined that a takeover happened cleanly (the new master
was caught up with the old), the new master *should not* create a
Partition Failover Record.

A failover record consists of the sequence number of *the last snapshot
marker* the node taking over had seen for that partition, and a UUID to
identify the record.


#### Partition Failover Log

A Partition Failover Log is a sequence of Partition Failover
Records. When a replica connects to an upstream UPR server, it will
request that server's whole failover log, rather than only the ID of the
final entry. It is likely that this log will be limited to some `N` most
recent entries. If so, as long as fewer than `N` entries are entered
into the failover log between a client being disconnected and coming
back online, it will be able to find a point to roll back to, otherwise
it will need to start from 0 to ensure consistency.

Note that this does not propagate as cluster wide configuration data,
but propagates down the replication path. This means different paths
through the replication topology will have their own failover histories.

## Handshakes

When a client connects and wants to start receiving changes for a
partition, the steps are:

 1. The client sends the partition it wants changes for, and if it has
    been connected before, the failover ID that was current when it
    connected last.

 2. If the failover ID the client sent is the most recent, or it did not
    send one because it was new, the server sends an "OK, go ahead"
    message.

    Otherwise, the server will look at the first entry in the failover
    log after the one the client sent, and send that sequence to the client.

    The client must find a snapshot marker it received *before* that
    sequence. It can then rollback to a point at or before the sequence
    at which it received that marker. If no such marker exists, the
    client must restart from scratch.

    If the failover ID the client sent is not present, it sends that
    message with sequence 0, the client has to start from scratch.

    Both the "OK" and "Sorry, please roll back" message are sent with the
    most recent failover ID, so that the client can remember it, or if
    the client is a replica, the complete failover history.

 3. When the client is ready to start processing changes, the client
    tells the server the sequence number to start sending changes from. 
    This message should also be sent with the failover ID received in
    `2` to prevent races where rebalances happen during the handshake.

#### Replicas

We also need to add a command that replicas can use to retrieve failover
history when they connect.

When connecting, replicas should download failover history *before*
doing step 3 to start receiving data.

### Example Handshakes

#### Notation

 * `<-` signals client to server messages
 * `->` signals server to client messages
 * License-plate like sequences `'DRF394'` indicate failover IDs

#### Connection handshake by a brand new client

**Handshake**:

    <- Hello! I'm on partition 5, I'm a brand new client
    -> OK, go ahead. The last failover on partition 5 was 'DRF394'
    <- Got 'DRF394', and I'm ready to start receiving changes on partition 5
       from sequence 0
    # ... change stream follows ...

#### Connection handshake by a resuming client, no failovers

 * The client had previously seen up to sequence 434

**Handshake**:

    <- Hello! I'm on partition 5, last failover I saw was 'DRF394'
    -> OK, go ahead. The last failover on partition 5 was 'DRF394'
    <- Got 'DRF394', and I'm ready to start receiving changes on partition 5
       from sequence 434
    # ... change stream follows ...

#### Connection handshake by a resuming client, across failovers

 * The client had previously seen up to sequence 434
 * The first failover following `'DRF394'` was `'QER053'` at sequence 426
 * The client's latest received snapshot marker that is before sequence
   426 is at 418

**Handshake**:

    <- Hello! I'm on partition 5, last failover I saw was 'DRF394'
    -> Sorry, the last failover on partition 5 was 'QER053'!
       You need to roll back to sequence 426 or before.
    # The client rolls back to sequence 418.
    <- Got 'QER053', and I'm ready to start receiving changes on partition 5
       from sequence 418
    # ... change stream follows ...

### Additional notes on handshakes

 * Deletion purging will require us to keep track of clients and
   sequences they have safely seen (are okay with deleting). As part of
   this we *could* keep track of their last seen failover ID/sequence to
   allow a simplified handshake.

 * "Hello" and "OK/Sorry" messages are *stateless*. That is, a client
   should be able to send another "Hello" message and get back another
   "OK/Sorry" response, to enable clients to try different failover IDs.

 * The handshake described here is all for a single partition. It is
   probably a good idea to extend these messages to work over multiple
   partitions at once to reduce chattiness.

## Document version history

* 0.0 - shared with Damien for review.
* 0.1 - shared with Damien for review.
* 0.2 - shared with Damien, Alk, Chiyoung, Aaron for review.
* 0.3 - incorporated Aaron feedback about safe points or safe
  messages, replacing the older “snapshot” concept.  Shared with
  Dustin, Marty, Filipe, Damien, Alk, Chiyoung, Aaron, Yaseen for
  review.
* 0.4 - Aaron prep doc for discussion. Removed deletion purging related
  material as it is a separate, albeit related, animal. "Safe Message
  (Server to Client)" became "Snapshot Marker." "Takover History/Log"
  became "Failover History/Log."
* 0.4.1 - Aaron forgot important dedup/snapshot problems. Back in now.
