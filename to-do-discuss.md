#TODO/To Discuss

 * T1 - Deduplication.

 * T2 - Failover.

 * T3 - Failback.

 * T4 - Master server restarts.

 * T5 - Replica server restarts.

 * T6 - Replica reconnects after a long absence.

 * T7 - Master has slow persistence and dies.

 * T8 - Replica has slow persistence and dies.

 * T9 - Replica does not know how to rewind.
   Especially, consider XDCR.

 * T10 - Rebalance (smooth takeover).

 * T11 - Multiple failovers and failbacks while replica was disconnected.

 * T12 - Preserving causality (“happens before”).

 * T13 - Pulling plugs.
   This was called chaos monkey on the whiteboard.

 * T14 - Total datacenter reboot.

 * T15 - Total datacenter reload from backups.

 * T15.1 - Partial restore from backups.

 * T16 - Deletions.

 * T17 - Expirations.

 * T18 - Compaction.

 * T19 - Deletion tombstone purging.

 * T20 - Multi-master XDCR.

 * T21 - Heterogeneous / mixed versions during online upgrade of a cluster (also with XDCR).

 * T22 - Clocks are skewed.

 * T23 - Offline upgrade of a cluster (also with XDCR).

 * T24 - Cluster and/or server recovery by copying files to the right places.

   At the node level versus full cluster level copying.

 * T25 - IP address changes.

 * T26 - Works with development environments (`cluster_run`).

 * T27 - Bucket deletions & re-creations while replica was disconnected.

 * T28 - RAM consumption when client is slow or disconnected.

 * T29 - OBSERVE.

 * T30 - Immediately consistent indexes.

 * T31 - Chain and star replication topologies and switching between them.

### Notes

What are these T-Numbers?
