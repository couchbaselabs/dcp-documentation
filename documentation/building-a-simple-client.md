# Building a Simple Client

In order to begin using DCP we first need a client capable of understanding the DCP protocol so that we can being streaming information. This page describes how to create a simple DCP client that will be able to connection to a single node Couchbase cluster and stream data. We will assume that the our single node cluster never crashes and that the client has no need for enabling any special mechanisms such as [flow control](flow-control.md) or [dead connection detection](dead-connections.md).

### DCP Is A Full Duplex Protocol

Responses for certain client requests can be received out of order and clients are required to inspect the opaque field of a DCP response in order to match it to a request. As a result client applications need to be able to read and write to a socket at the same time.

In order to support a full duplex connection clients can for example be implemented in one of the following ways:

1. By creating a reader thread responsible for reading from your socket and also creating a writer thread that will write to the socket.
2. By using epoll()
3. By using kqueue()
4. By using select()

Note that there are other ways to implement a full duplex connection and the list above is meant to be examples for how to accomplish this. Also keep in mind that clients should not be creating a lot of connections. As a result developers should not worry about choosing mechanisms known to scale the number of connections your client can handle.

### Creating a connection

Once your client networking code is in place the next step is to verify that your client can create and close a DCP connection. Creating a connection simply involves sending an [Open Connection](commands/open-connection.md) message which simply names the connection. After your client sends the [Open Connection](open-connection.md) message the client should check to make sure the connection was successfully created on the server. If it was you should be able to request dcp stats from the server and see your connection stats listed in the stats output. This connection will remain open until your client closes its socket. If the connection closes unexpectedly then this means something unexpected happened and the logs should be inspected since an error is always logged if a connection is closed unexpectedly.

### Creating a stream

Once you have a connection established with the server then the next thing to do is to open a stream to the server to stream out data for a specific VBucket. For a basic client the simplest thing to do is to always stream data starting with the first mutation that was received in the VBucket. To do this the Consumer should send [Stream Request](commands/stream-request.md) messages for each VBucket that it wants to recieve data for.

* VBucket - Set this to the VBucket ID that you want your client to receive data for. This number should always be between 0 and 1023 inclusive.
* Flags - The flags field is used to define specialized behavior for a stream. Since we don't need any specialized behavior we set the flags field to 0.
* Start By Seqno - Should be set to 0 since sequence numbers are assigned starting at sequence number 1. The Start sequence number is the last sequence number that the Consumer received and since for our basic streaming case we want to always start from the beginning we send 0 in this field.
* End By Seqno - For our basic client we want to recieve a continuous stream and receive all data as in enters Couchbase Server. To do this the highest sequence number possible should be specified. In this case the end sequence number should be 2^64-1.
* VBucket UUID - Since we that are starting to recieve data for the first time 0 should be specified.
* Snapshot Start Seqno - Since we that are starting to recieve data for the first time 0 should be specified.
* Snapshot End Seqno - Since we that are starting to recieve data for the first time 0 should be specified.


### Client Side State

DCP achieves restartability of DCP streams through the use of a failover log and sequence number.

* Failover Log
* Last Recieved Seqno
* Last Snapshot Start Seqno
* Last Snapshot End Seqno

### Restarting from where you left off


### Handling a rollback

