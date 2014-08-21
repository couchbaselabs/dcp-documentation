
#UPR Protocol Specification

##Terminology

* **Consumer** - The endpoint in a connection that is responsible for requesting different kinds of data. The consumer is responsible for doing some sort of processing with the data sent from the producer.
* **Producer** - The endpoint in a connection that is responsible for producing data for given requests.
* **Stream** - A stream is a series of messages sent over a given period of time that are generate from a particular request message.
* **Snapshot** - A unique sequence of keys that is sent by a stream.

##Messages

UPR utilize the Memcached binary protocol as the transport protocol
(see https://code.google.com/p/memcached/wiki/BinaryProtocolRevamped
for more information about the protocol layout), and defines a set
opcodes with new commands.

It does differ from the standard Memcached connections in the way that
a UPR connection is full duplex while the normal connections is
simplex (the client send a command, the server respond etc).

The typical scenario is that the client start requesting a stream and
upon success the server will start sending *command* messages back to
the client for mutations/deletions/expirations etc. The client may at
any time send additional commands to the server to start additional
UPR streams etc.

###Protocol Definitions

* [**Open Connection**](commands/open-connection.md)
* [**Add Stream**](commands/add-stream.md)
* [**Close Stream**](commands/close-stream.md)
* [**Stream Request**](commands/stream-request.md)
* [**Get Failover Log**](commands/failover-log.md)
* [**Stream End**](commands/stream-end.md)
* [**Snapshot Marker**](commands/snapshot-marker.md)
* [**Mutation**](commands/mutation.md)
* [**Deletion**](commands/deletion.md)
* [**Expiration**](commands/expiration.md)
* [**Flush**](commands/flush.md)
* [**Set VBucket State**](commands/set-vbucket-state.md)

