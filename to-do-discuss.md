#TODO/To Discuss

 * Deduplication.
 * Failover.
 * Failback.
 * Master server restarts.
 * Replica server restarts.
 * Replica reconnects after a long absence.
 * Master has slow persistence and dies.
 * Replica has slow persistence and dies.
 * Replica does not know how to rewind.
   Especially, consider XDCR.
 * Rebalance (smooth takeover).
 * Multiple failovers and failbacks while replica was disconnected.
 * Pulling plugs.
   This was called chaos monkey on the whiteboard.
 * Total datacenter reboot.
 * Total datacenter reload from backups.
   * Partial restore from backups.
 * Deletions.
 * Expirations.
 * Compaction.
 * Deletion tombstone purging.
 * Multi-master XDCR.
 * Heterogeneous / mixed versions during online upgrade of a cluster (also with XDCR).
 * Clocks are skewed.
 * Offline upgrade of a cluster (also with XDCR).
 * Cluster and/or server recovery by copying files to the right places.
   At the node level versus full cluster level copying.
 * IP address changes.
 * Works with development environments (`cluster_run`).
 * Bucket deletions & re-creations while replica was disconnected.
 * RAM consumption when client is slow or disconnected.
 * OBSERVE.
 * Immediately consistent indexes.
 * Chain and star replication topologies and switching between them.
 * Replicas may not easily be able to roll back, can we work around
   this?

### Notes

What are these T-Numbers?
