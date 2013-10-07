
#Upgrade 2.x to 3.x

Couchbase 3.0 will contain both the TAP and UPR protocols. As a result we can choose to use either of the protocols for replication. In order to make things as simple as possible when upgrading we will only use the tap protocol when we are in a mixed cluster scenario. Once all nodes are running Couchbase 3.x NS_Server will start closing tap streams and replacing those closed streams with UPR streams. NS_Server will continue to do this until the entire cluster is running the UPR protocol.

Switching from a TAP stream to an UPR stream will cause a replica VBucket to need to be rematerialized because the UPR protocol requires that each VBucket, no matter what its state is, must have the exact same mutation history. As a result NS_Server will need to slowly switch TAP streams to UPR streams in order to avoid too many replicas being completely erased and to limit the backfills that take place. As a result NS_Server should limit stream switches on individual nodes to one incoming stream and one outgoing stream.

* Note: NS_Server team will figure out how to resolve partial transitions which can occur if the rebalance is stopped.