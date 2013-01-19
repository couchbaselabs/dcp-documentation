# Unified Protocol for Replication (UPR)

* status: PROPOSAL / DRAFT
* latest: https://github.com/couchbaselabs/cbupr/blob/master/spec.md

## Table of contents

* Summary
* Motivation
* Previous writeups / proposals / related links
* Terminology
** Partition == VBucket
** Logical objects / properties
* Basic scenarios
** Connection by a replica
** Reconnection by a replica
** Master server restarts
** Master server restarts with slow persistence
** Failover
** Rebalance
* Additional concepts
* Interesting scenarios
* Conclusion & next steps
* TODO / followup / feedback
* Document version history

## Summary

Couchbase Unified Protocol for Replication (UPR or “upper”) is the
proposed replication networking protocol for Couchbase “2.next”.  It
addresses replication stream restartability.

## Motivation

Many upcoming features will require restartable, incremental data
replication.  For example...

* 1. Indexing, especially distributed indexing.
* 2. Incremental backup.
* 3. Intra-cluster replication (e.g., fewer backfills).
* 4. Inter-cluster replication (e.g., higher performance XDCR)
* 5. Integration with outside datastores and indexes (OLAP, Hadoop, ElasticSearch, ETL software, etc).
* 6. Persistence can also be logically “modeled” using the concepts of replication.

UPR addresses these multiple replication needs with a single, unified,
synchronization model and networking protocol.

The existing TAP checkpoint protocol (Couchbase 1.8.x and 2.0.0) is
insufficient because checkpoints are ephemeral (does not handle
clients that have been disconnected a long time),

The existing XDCR protocol (Couchbase 2.0.0, HTTP/REST-based) is also
insufficient for performance reasons.

This document will eventually become a detailed network protocol
specification.  This document and the steps of reviewing and approving
it are meant to help the team think through difficult cases before we
make code changes.  With a well-specified network protocol, separate
teams should be able to concurrently implement clients, servers,
orchestrators, tools, 3rd party integrations, test suites, etc.
Protocol changes will also be via “change controlled” updates to this
spec so that teams can work without ambiguity.

## Previous writeups / proposals / related links

* 1. http://hub.internal.couchbase.com/confluence/display/PM/3.0+-+Indexing+and+Querying+links
* 2. http://hub.internal.couchbase.com/confluence/display/cbeng/Checkpointing+and+Incremental+Replication+Through+TAP
* 3. [Insert pointers to Dustin’s writeups here.]
* 4. [Insert pointers to Aaron’s writeups here.]

## Terminology

### Partition == VBucket

In this document, “partition” and “vbucket” are synonyms, where we
favor the word “partition” for less confusion to newcomers.  So, a
bucket has 1024 partitions.

### Logical objects / properties

The following are important logical “objects” or concepts in the
protocol.  Real client or server implementations might not use these
names or have a 1-for-1 physical representation of these concepts.

#### Bucket

A Bucket has properties of:

* Bucket Name (like “default” or “product-catalog”).
* Bucket Version (new for UPR).
* A Bucket also has 1024 Partitions.

#### Partition

A Partition has properties of:

* Partition Id (0 to 1023).
* Partition State (active, replica, pending, or dead).
* Partition Takeover History (new for UPR).
** This is a sequence of Partition Takeover Records.
** A Partition Takeover Record == active Node Id + Sequence Number.
* A Partition also has a logical Base Data Set.
* A Partition also has a logical Changes Stream.

#### Node Id

A Node Id is a “Host:Port”.

#### Sequence Number

A Sequence Number is a 64-bit unsigned number starting from 0 and is
used to identify Mutations, Safe Messages, Partition Takeover Records,
etc.

Unlike in Couchbase 2.0.0 where Sequence Numbers are updated at
persistence flushing time by the Couchstore library, in UPR the
Sequence Numbers should be incremented or assigned immediately at
mutation request processing time (e.g., incremented in-memory).

[???? Is a Sequence Number == CAS number?  Some say for XDCR they
might be !=, but would be nice to unify them.  If the CAS value is not
necessarily the same across clusters, then we can possibly unify
Sequence Number and CAS]

#### Base Data Set

A Base Data Set is a logical, starting set of distinct keys and their
associated values.  For example, after you fully, deeply compact a
Partition (which includes purging deletion tombstones) and are left
with a set of remaining, live key-values (no more deletion and
mutation history), you have a Base Data Set.

#### Changes Stream

A Changes Stream is a logical sequence of Mutations and Partition
Takeover Records, ordered by their Sequence Number.  A Base Data Set
plus Changes Stream should fully, but not necessarily compactly,
describe the latest key-value data of a Partition. A Changes Stream
might also have ephemeral (not necessarily repeatable) Safe Messages
interspersed in it.

#### Mutation

A Mutation is a key-value create/update/delete.

#### Safe Message

A Safe Message includes a Sequence Number.

A Safe Message, when sent from a master to a client, says "I, the
master, can guarantee to you, the client, that before Sequence Number
X there are no `holes' in the data you've seen”.  That is, the client
has been sent at least one version of all keys that have existed at X
or before.  The client can use the Safe Message’s sequence number X to
establish a rollback point in whatever manner it wants.  Without a Safe
Message it would not be possible for clients to determine a safe
message to roll back to, because it could never be sure it didn't have
holes in its data.  Related to Safe Message promises, the master may
no longer de-duplicate mutations older than the Safe Message’s
sequence number since de-duplication would introduce holes.

A Safe Message, when sent from a client to a master, says “I, the
client, am okay with you throwing away data (e.g., purging deletion
tombstones) before this sequence number Y, because I have either seen
them or am confident I don't need to see them later.”

A master does not necessarily need to persist the Safe Messages that
it sends to clients, so a “replay” of the Changes Stream might have
different Safe Messages.

A client also does not necessarily need to persist any or all of the
Safe Messages that it receives or to prepare its own rollback’able
points in its persistent storage, at the risk of having its
reconnected streams restarting from zero.

#### Partition Takeover Record

A Partition Takeover Record is a record of a server becoming the
master for a partition.  A restarted master is also recorded with its
own Partition Takeover Record, as that's treated as an edge case of a
server (re-)becoming the master.

The properties of a Partition Takeover Record include...

* A Sequence Number.
* The active Node Id that took over the Partition.

#### Partition Takeover History

A Partition Takeover History is a sequence of Partition Takeover
Records.  For example...

    A:14, B:24, C:201

That would read as node A was the master/active server for the
partition at sequence number 14.  Then node B took over as
active/master at sequence number 24, and node C took over as
active/master at sequence number 201.

#### Client Registry

A set of identifiers of clients (each client provides its own unique
identifier) tracked by the master, and replicated to interested
replicas (usually, the replicas that can takeover for a master).  It
is used to help the master manage deletion tombstone purging.

[???? This just popped up 2013/01/18 - do we actually need this
concept / optimization?]

### Example conventions

Examples in this document follow these conventions:

* numbers are Sequence Numbers.
* “A”, “B”, “C” are Node Id’s.
* “X”, “Y”, “Z” are Node Id’s of machines from a different, remote “east coast” cluster.
* “R” prefix before a number means a rollback’able point (e.g. R91)
* “m” prefix before a number means a Mutation (e.g., m321).
* “n” prefix before a number means a Mutation from the east coast cluster (e.g., n123).
* “s” prefix before a number means a Safe Message (e.g., s17).
* “t” prefix before a number means a Safe Message from the east coast cluster (e.g., t31).
* CapitalLetter:number means a Partition Takeover Record (e.g, “A:16).

For example, here’s the changes stream of a partition...

    A:14, m15, m16, m17, m18, s18, m19, m20

That would read in English as node A took over active/master ownership
of the partition at sequence number 14.  Then, there were some
mutations: 15, 16, 17, 18.  Then, there was a safe message at sequence
number 18.  Then, more mutations: 19, 20.

## Basic scenarios

Readers aware of DVCS systems like git may find basic scenario
handling to be familiar.

### Connection by a replica

When a replica makes a connection (“⇐”) to a master server in order to
replicate a partition (whether the first-time or not), the replica
provides its partition takeover history and rollback'able points as
part of its UPR/TAP-Connect request.  For the first time, those
parameters are empty...

    ⇐ Partition Takeover History: (none) and Rollback’able To: (none)

The master will determine that there are no conflicts and no replica
rewinding in this easy case.  So, the master next streams its
partition takeover history, its base data set and then its changes
stream to the replica (“⇒”).  For example...

    ⇒ A:14, m15, m16, m17, m18, s18, m19, m20, m21, s21, m22, m23

The replica persists what it receives, and perhaps handles the Safe
Message points of s18 and s21 by snapshot’ing or recording rollback
information into its persistence storage at those times (e.g., R18 and
R21), by replica-specific means.

### Reconnection by a replica

If the replica exits, restarts and reconnects, it might not have
persisted everything that the master had sent (e.g., replica process
crashed).  For example, the restarted replica might be missing
mutation m23...

    replica state: A:14, m15, m16, m17, m18, R18, m19, m20, m21, R21, m22

Then the replica would reconnect using these parameters, which mean “I
think A was the master at sequence number 14, and I can rollback to
either sequence numbers 18 or 21”...

    ⇐ Partition Takeover History: A:14 and Rollback’able To: 18, 21

Or, the replica might be missing even more records (missing safe
message / rollback’able point at 21)...

    replica state: A:14, m15, m16, m17, m18, R18, m19

...and the replica would instead reconnect with...

    ⇐ Partition Takeover History: A:14 and Rollback’able To: 18

An even slower replica might only have...

    replica state: A:14, m15, m16

...and might reconnect with...

    ⇐ Partition Takeover History: A:14 and Rollback’able To: (none)

In these cases, the master can determine what mutations the replica is
missing and resend those missing entries.  If the master no longer has
relevant entries (perhaps it did a deep compaction while the replica
was disconnected for a very long time), then the replica will have to
rewind or reset back to zero and the master will have to resend
everything (the master’s base data set and full changes stream).

### Master server restarts

If the master server restarts, it must generate and persist a new
partition takeover record in its partition takeover history before it
services requests, as though it were taking over from itself...

    master state: A:14, m15, m16, m17, m19, m20, m22, m23, A:24

Note: the master did not persist any Safe Message records that it had
sent out earlier since Safe Messages are ephemeral.

Note: If a master server restarts in quick succession, we might end up
with multiple partition takeover records, like A:24, A:25, A:26, A:27.
Some servers may choose to optimize, realizing there are no
intermediate mutations, and keep just the single, last A:24 record.

### Master server restarts after slow persistence

In the case when persistence on the master was slow and the master
process crashes, the system must determine that reconnecting replicas
may need to rewind or rollback some of their mutations.  For example,
if the master restarts, assigns itself a new partition takeover record
of A:20’, and also services some new mutations...

    restarted master state: A:14, m15, m16, m17, m19, A:20’, m21’, m22’, m23’

A replica that later reconnects might have a state of “being ahead” of
the restarted master...

    replica state: A:14, m15, m16, m17, m18, R18, m19, m20, m21, R21, m22

Above, the replica’s m20, m21, m22 records have diverged from the
restarted master.  The replica reconnects to the restarted master
with...

    ⇐ Partition Takeover History: A:14 and Rollback’able To: 18, 21

The system can determine that the replica has diverged and that the
replica would need to rewind or rollback to m18 (their shared history
point) and have the master replay from m19 onwards (thereby sending
m19, A:20’, m21’, m22’, m23’ and onwards to the replica).

Note in this example that m19 is resent to the replica, which is safe,
but is needed to meet the replica’s claimed rollback’able points.

[???? Who determines divergence?  The master or the replica?  If it’s
the replica, then the replica would have more freedom to choose what
to do but there’s perhaps more chance for bugs on repeated divergence
implementation logic.]

### Failover

If the master server dies and a different server B (a different
replica) becomes the new master, server B might also have been behind
compared to other replicas.  As soon as server B becomes the
active/master for the partition, following the same protocol rules, it
must generate and persist a new partition takeover record in its
partition takeover history before it services requests...

    master B state: A:14, m15, m16, m17, m18, s18, m19, B:20

As it services requests, master B will get new entries in its changes
stream...

    master B state: A:14, m15, m16, m17, m18, s18, m19, B:20, m21’’, m22’’, m23’’, s23’’, m24’’

When a replica that has this state...

    replica state: A:14, m15, m16, m17, m18, R18, m19, m20, m21, R21, m22

...connects to the new master B...

    ⇐ Partition Takeover History: A:14 and Rollback’able To: 18, 21

...the system can determine that the replica has diverged from the new
master B, and the replica has to rollback to the shared point of m18
and have the new entries streamed (m19, B:20, m21’’, m22’’, m23’’,
s23’’, m24’’ and onwards) to the rolled-back replica.

As a very rough analogy to DVCS (e.g., git), a partition takeover
record is kind of like the start of a branch in the revision history.

### Rebalance

A rebalance, which is a controlled partition takeover, would also have
the new master immediately assign a new partition takeover record,
thereby following the scenario similar to failover.  In general,
whenever a partition is switched to active state, it should be
assigned a new partition takeover record (with its own assigned
sequence number).

## Additional concepts

### Bucket versions

Every bucket will have a version. These bucket version identifiers
must be unique, especially as buckets are created and (especially)
re-created.

For example, after taking an incremental backup, you might delete and
recreate the “default” bucket.  The next incremental backup process
should be able to detect that bucket “default” (0) is not the same as
bucket “default” (1), via their different bucket versions.

## UPR is based on TAP protocol, not REST/HTTP

UPR is an application of the TAP protocol, introducing new
sub-commands and parameters into TAP’s existing protocol framing.  TAP
protocol framing itself is not expected to change.

### Partition takeover history is replicated

As a partition is reassigned to a new master, it gathers more and more
partition version history.  For example, partition 23 might have this
takeover history: [A:14, B:21, C:71, A:117].  This partition takeover
history information is replicated to replicas.

Partition takeover history may eventually be pruned, and the pruned
takeover history must be replicated to interested replicas.

### De-duplication occurs for the latest, mutations

An important feature of Couchbase is to provide de-duplication of
recent mutations of the same hot keys.  In UPR, this can only happen
for mutations that are newer than the latest Safe Message that a
master has sent.

### Deep compaction

Eventually, there will be a long sequence of mutations in a Changes
Stream.  To reclaim space, a deep compaction (compaction and deletion
tombstone purging) can be performed on the oldest section of the
Changes Stream.  After such a “deep compaction”, you’re left with
just the set of active keys (and their associated values).  That is,
you’re left with a new Base Data Set.

### Clients send Safe Messages back to master

To help track which part of a Changes Stream can be safely “deeply
compacted”, clients should occasionally send Safe Messages back to the
master.

[???? Need more info here.]

### Replicas should support rollbacks for efficiency

If a replica (like a Hadoop integration) disconnects, a lot can happen
while it’s disconnected.  If the replica cannot support rewinding /
rollback, or has been disconnected so long that the master server has
compacted/purged older mutation history, then a full dump (starting
from zero) is needed.

Replicas that support MVCC and rollback to any past point in time
would have the easiest implementation of this.

Incremental backup might give the option to users to just keep
blithely rolling forward with “just give me your best effort
incremental delta, even if there’s a gap or inconsistency”, because
the user would rather backup at least some inconsistent recent data
rather than nothing.

## Interesting scenarios

Interesting cases to consider are listed next.

### T1 - Deduplication.

### T2 - Failover.

### T3 - Failback.

### T4 - Master server restarts.

### T5 - Replica server restarts.

### T6 - Replica reconnects after a long absence.

### T7 - Master has slow persistence and dies.

### T8 - Replica has slow persistence and dies.

### T9 - Replica does not know how to rewind.

Especially, consider XDCR.

### T10 - Rebalance (smooth takeover).

### T11 - Multiple failovers and failbacks while replica was disconnected.

### T12 - Preserving causality (“happens before”).

### T13 - Pulling plugs.

This was called chaos monkey on the whiteboard.

### T14 - Total datacenter reboot.

### T15 - Total datacenter reload from backups.

### T15.1 - Partial restore from backups.

### T16 - Deletions.

### T17 - Expirations.

### T18 - Compaction.

### T19 - Deletion tombstone purging.

### T20 - Multi-master XDCR.

### T21 - Heterogeneous / mixed versions during online upgrade of a cluster (also with XDCR).

### T22 - Clocks are skewed.

### T23 - Offline upgrade of a cluster (also with XDCR).

### T24 - Cluster and/or server recovery by copying files to the right places.

At the node level versus full cluster level copying.

### T25 - IP address changes.

### T26 - Works with development environments (cluster_run).

### T27 - Bucket deletions & re-creations while replica was disconnected.

### T28 - RAM consumption when client is slow or disconnected.

### T29 - OBSERVE.

### T30 - Immediately consistent indexes.

### T31 - Chain and star replication topologies and switching between them.

The sequence numbers can be used to incremental move data from the
most up-to-date replica to the replica chose to be the new master?

### T32 - Disk corruption.

## Conclusion & next steps

Next steps are to get this reviewed, feedback’ed, improved, approved and built.

## TODO / followup / feedback

From alk... In addition to that, urp source can _always_ drop current
safe message/checkpoint and start sending mutations towards more fresh
one. Where I understand shapshot/checkpoint as "there cannot be any
'unclosed' holes".  Unless there are deletion purges in between.  I
can formally prove that.  And in fact, combined to idea of associating
_existing_ tap checkpoints with couch seq numbers we can _fix existing
tap_ to be resumable from any point in time, provided deletion purging
is handled.

From aaron... Last Safe Seq doesn't replace snapshotting. Last Safe
Seq is part of a proposed way of dealing with deletion purging, it's
an UPR Client -> Provider message that means "I am okay with you
throwing away data (deletes) before this sequence, because I have
either seen them or am confident I don't need to". It is not tied to
snapshots in any way really, just sequence numbers.

My conception of (non-persistent) snapshots is:

A Snapshot, to a client, is an UPR Provider -> Client message that
says "I can guarantee you that before sequence X there are no `holes'
in the data you've seen." (That is, the client has seen at least one
version of all keys that have existed at X or before) which the client
can use to establish a rollback point in whatever manner it
wants. Without snapshots it would not be possible for clients to
establish a safe message to roll back to, because it could never be
sure it didn't have holes in its data.

## Document version history

* 0.0 - shared with Damien for review.
* 0.1 - shared with Damien for review.
* 0.2 - shared with Damien, Alk, Chiyoung, Aaron for review.
* 0.3 - incorporated Aaron feedback about safe points or safe
  messages, replacing the older “snapshot” concept.  Shared with
  Dustin, Marty, Filipe, Damien, Alk, Chiyoung, Aaron, Yaseen for
  review.
