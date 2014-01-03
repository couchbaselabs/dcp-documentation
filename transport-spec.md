
#UPR Transport Specification

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


###Open Connection (opcode 0x50)

Sent by en external entity to a producer and a consumer to create a
logical channel.

The request:
* Must have extras
* Must have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| sequence number                                               |
       +---------------+---------------+---------------+---------------+
      4| flags                                                         |
       +---------------+---------------+---------------+---------------+

Flags is specified as a bitmask in network byte order with the
following bits defined:

     1 - Type: Producer (bit set) / Consumer (bit cleared).

The following example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x50          | 0x00          | 0x18          |
        +---------------+---------------+---------------+---------------+
       4| 0x08          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x20          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x62 ('b')    | 0x75 ('u')    | 0x63 ('c')    | 0x6b ('k')    |
        +---------------+---------------+---------------+---------------+
      36| 0x65 ('e')    | 0x74 ('t')    | 0x73 ('s')    | 0x74 ('t')    |
        +---------------+---------------+---------------+---------------+
      40| 0x72 ('r')    | 0x65 ('e')    | 0x61 ('a')    | 0x6d ('m')    |
        +---------------+---------------+---------------+---------------+
      44| 0x20 (' ')    | 0x76 ('v')    | 0x62 ('b')    | 0x5b ('[')    |
        +---------------+---------------+---------------+---------------+
      48| 0x31 ('1')    | 0x30 ('0')    | 0x30 ('0')    | 0x2d ('-')    |
        +---------------+---------------+---------------+---------------+
      52| 0x31 ('1')    | 0x30 ('0')    | 0x35 ('5')    | 0x5d (']')    |
        +---------------+---------------+---------------+---------------+
    UPR_OPEN command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x0018
    Extra length (4)    : 0x08
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000020
    Opaque       (12-15): 0x00000001
    CAS          (16-23): 0x0000000000000000
      seqno      (24-27): 0x00000000
      flags      (28-31): 0x00000000 (consumer)
    Key          (32-55): bucketstream vb[100-105]

Upon success, the following message is returned.

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x50          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_OPEN response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0x00000001
    CAS          (16-23): 0x0000000000000000

###Add Stream (opcode 0x51)

Sent by ebucketmigrator to the consumer to tell the consumer to
initiate a stream request

The request:
* Must have extras
* Must not have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| flags                                                         |
       +---------------+---------------+---------------+---------------+

Flags is specified as a bitmask in network byte order with the
following bits defined:

     1 - Takeover

The following example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x51          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x04          | 0x00          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
    UPR_ADD_STREAM command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x51
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x04
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0005
    Total body   (8-11) : 0x00000004
    Opaque       (12-15): 0x00000001
    CAS          (16-23): 0x0000000000000000
      flags      (24-27): 0x00000001 (takeover)

The UPR consumer will now try to set up an UPR stream between the
producer and the consumer, and once it is established it will respond
with the following message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x51          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x04          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x10          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_ADD_STREAM response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x51
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x04
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000004
    Opaque       (12-15): 0x00000001
    CAS          (16-23): 0x0000000000000000
      opaque     (24,27): 0x00001000

The opaque field in the extra field of the response contains the
opaque value used by messages passing for that vbuckt. The vbucket
identifier in the extra field is the vbucket identifier this response
belongs to.

###Close Stream (opcode 0x52)

Sent by ebucketmigrator to parties in an UPR stream to close a stream for
a named vbucket as soon as possible

The request:
* Must not extras
* Must not have key
* Must not have value

The layout of a message looks like:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x52          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_CLOSE_STREAM command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x52
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0005
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

The UPR consumer will close the UPR stream for the specified vbucket
imediately and let the UPR producer know as soon as it receives a
message bound for that vbucket.

The UPR producer will stop sending messages for this vbucket. It may
still receive response messages for this stream.

###Failover Log Request (opcode 0x54)

The Failover log request is used by the consumer to request all known
failover ids a client may use to continue from. A failover id consists
of the vbucket UUID and a sequence number. If a client can't find a
known failover id, it should select the vbucket with the highest
sequence number since that is the stream with the shortest path
to completion.

The request:
* Must not have extras
* Must not have key
* Must not have value

The response:
* Must not have extras
* Must not have key
* Must have value on Success

The following example requests the failover log for vbucket 0:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x54          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_GET_FAILOVER_LOG command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x54
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

If the command executes successful (see the status field), the
following packet is returned from a server which have 4 different
failover ids available:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x54          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x40          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x54          | 0x32          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      44| 0x00          | 0xde          | 0xca          | 0xfe          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0x01          | 0x34          | 0x32          | 0x14          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0xfe          | 0xed          | 0xfa          | 0xce          |
        +---------------+---------------+---------------+---------------+
      64| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      68| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      72| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      76| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      80| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      84| 0x00          | 0x00          | 0x65          | 0x24          |
        +---------------+---------------+---------------+---------------+
    UPR_GET_FAILOVER_LOG response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x54
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000040
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      vb UUID    (24-31): 0x00000000feeddeca
      vb seqno   (32-39): 0x0000000000005432
      vb UUID    (40-47): 0x0000000000decafe
      vb seqno   (48-55): 0x0000000001343214
      vb UUID    (56-63): 0x00000000feedface
      vb seqno   (64-71): 0x0000000000000004
      vb UUID    (72-79): 0x00000000deadbeef
      vb seqno   (80-87): 0x0000000000006524

There are multiple reason's why the request may fail (see the status
field), but the one that's most likely to expect is "not my vbucket"
if the requested vbucket isn't located on the server.

###Stream Request (opcode 0x53)

Sent by the consumer side to the producer specifying that the consumer
want some piece of data (Ex. XDCR). In order to initial a stream from
a vbucket the consumer must send the following command below. In order
to initiate multiple stream the consumer needs to send multiple
commands. The value specified in opaque in the stream request packet
will be used as opaque field in all commands sent for the stream.

The request:
* Must have extras
* Must not have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| Flags                                                         |
       +---------------+---------------+---------------+---------------+
      4| RESERVED                                                      |
       +---------------+---------------+---------------+---------------+
      8| Start sequence number                                         |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| End sequence number                                           |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     24| VBucket UUID                                                  |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     32| High sequence number                                          |
       |                                                               |
       +---------------+---------------+---------------+---------------+
       Total 40 bytes

* **Flags** - Used to specify extra information added in the extra
    section for modifying what the stream send.
* **Start By Seqno** - Specified the last by sequence number that has
    been seen by the consumer.
* **End By Seqno** - Specifies that the stream should be closed when
    the sequence number with this ID has been sent.
* **VBucket UUID** - A unique identifier that is generated that is
    assigned to each VBucket. This number is generated on an unclean
    shutdown or when a Vbucket becomes active.
* **High Seqno** - The high sequence number at the time that the
    VBucket UUID was generated.

Set VBucket UUID and start sequence number to 0 to perform a full
backfill from the "current" vbucket UUID. This may be thought of
as a special case where the consumer don't have any data at all (to
eliminate the need of requesting the failover log first).

The response:
* Must not have extras
* Must not have key
* May have value

On an "OK" response the failover log is included. The "rollback"
response contains the sequence number to roll back to.

The following example tries to initiate a stream for vbucket 0 that
continues from a given point in time, but the server can't continue
from that point and tells the client to roll back to a different
sequence. The client retries with the information that the server
replied with, and the stream is established successfully.

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x53          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x28          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x28          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x10          | 0x00          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0xff          | 0xee          | 0xdd          |
        +---------------+---------------+---------------+---------------+
      40| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      44| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x53
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x28
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000028
    Opaque       (12-15): 0x00001000
    CAS          (16-23): 0x0000000000000000
      flags      (24-27): 0x00000000
      reserved   (28-31): 0x00000000
      start seqno(32-39): 0x0000000000ffeedd
      end seqno  (40-47): 0xffffffffffffffff
      vb UUID    (48-55): 0x00000000feeddeca
      high seqno (56-63): 0x0000000000000000


      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x53          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x08          | 0x00          | 0x00          | 0x23          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x08          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x10          | 0x00          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x53
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x08
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0023 (Rollback)
    Total body   (8-11) : 0x00000008
    Opaque       (12-15): 0x00001000
    CAS          (16-23): 0x0000000000000000
      rollback # (24-31): 0x0000000000000000

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x53          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x28          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x28          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x10          | 0x00          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      40| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      44| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x53
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x28
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000028
    Opaque       (12-15): 0x00001000
    CAS          (16-23): 0x0000000000000000
      flags      (24-27): 0x00000000
      reserved   (28-31): 0x00000000
      start seqno(32-39): 0x0000000000000000
      end seqno  (40-47): 0xffffffffffffffff
      vb UUID    (48-55): 0x00000000feeddeca
      high seqno (56-63): 0x0000000000000000

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x53          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x10          | 0x00          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x54          | 0x32          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      44| 0x00          | 0xde          | 0xca          | 0xfe          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0x01          | 0x34          | 0x32          | 0x14          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0xfe          | 0xed          | 0xfa          | 0xce          |
        +---------------+---------------+---------------+---------------+
      64| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      68| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      72| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      76| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      80| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      84| 0x00          | 0x00          | 0x65          | 0x24          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x53
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000040
    Opaque       (12-15): 0x00001000
    CAS          (16-23): 0x0000000000000000
      vb UUID    (24-31): 0x00000000feeddeca
      vb seqno   (32-39): 0x0000000000005432
      vb UUID    (40-47): 0x0000000000decafe
      vb seqno   (48-55): 0x0000000001343214
      vb UUID    (56-63): 0x00000000feedface
      vb seqno   (64-71): 0x0000000000000004
      vb UUID    (72-79): 0x00000000deadbeef
      vb seqno   (80-87): 0x0000000000006524

#####Error codes

* **PROTOCOL_BINARY_RESPONSE_KEY_ENOENT (0x01)** - If the producer does not know about the vbucket uuid specified. This menas the consumer should roll all the way back to 0.
* **PROTOCOL_BINARY_RESPONSE_KEY_EEXISTS (0x02)** - If a stream for this VBucket already exists on the same connection.
* **PROTOCOL_BINARY_RESPONSE_NOT_MY_VBUCKET (0x07)** - If the VBucket the stream is requested for does not exist.
* **PROTOCOL_BINARY_RESPONSE_ERANGE (0x22)** - If the start and end sequence numbers are specified incorrectly. For example the start sequence number cannot be bigger than the current high sequence number in the VBucket. The start sequence number must also be less than the end sequence number.
* **PROTOCOL_BINARY_RESPONSE_ROLLBACK (0x23)** - If the consumer needs to rollback its data before reconnecting.
* **(Disconnect)** - If the connection is invalid or if the connection is for a consumer.

###Stream End (opcode 0x55)

Sent to tell the consumer that the producer will has no more messages to stream.

The request:
* Must have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x55          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x04          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_END command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x55
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x04
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000004
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      flag       (24-27): 0x00000000 (OK)

The flag may have the following values:

* OK (0x00) - The stream has finished without error.
* State Changed (0x01) - The state of the VBucket that is being streamed has changed to state that the consumer does not want to receive.


###Snapshot Marker (opcode 0x56)

Sent by the producer to tell the consumer that a new snapshot is being
sent. A snaphot is simply a series of commands that is guarenteed to
contain a unique set of keys.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x56          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_SNAPSHOT_MARKER command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x56
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000


###Mutation (0x57)
Tells the consumer that the message contains a key mutation.

The request:
* Must have extras
* Must have key
* May have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| flags                                                         |
       +---------------+---------------+---------------+---------------+
     20| expiration                                                    |
       +---------------+---------------+---------------+---------------+
     24| lock_time                                                     |
       +---------------+---------------+---------------+---------------+
     28| Metadata Size                 |
       +---------------+---------------*
       Total 30 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x57          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x1e          | 0x00          | 0x02          | 0x10          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x28          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x12          | 0x10          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x64          | 0xa5          |
        +---------------+---------------+---------------+---------------+
      20| 0xac          | 0xec          | 0x8a          | 0x56          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      44| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0x00          | 0x00          | 0x68 ('h')    | 0x65 ('e')    |
        +---------------+---------------+---------------+---------------+
      56| 0x6c ('l')    | 0x6c ('l')    | 0x6f ('o')    | 0x77 ('w')    |
        +---------------+---------------+---------------+---------------+
      60| 0x6f ('o')    | 0x72 ('r')    | 0x6c ('l')    | 0x64 ('d')    |
        +---------------+---------------+---------------+---------------+
    UPR_MUTATION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x57
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x1e
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0210
    Total body   (8-11) : 0x00000028
    Opaque       (12-15): 0x00001210
    CAS          (16-23): 0x000064a5acec8a56
      by seqno   (24-31): 0x0000000000000004
      rev seqno  (32-39): 0x0000000000000001
      flags      (40-43): 0x00000000
      expiration (44-47): 0x00000000
      lock time  (48-51): 0x00000000
      nmeta      (52-53): 0x0000
    Key          (54-58): hello
    Value        (59-63): world


###Deletion (0x58)
Tells the consumer that the message contains a key deletion.

The request:
* Must have extras
* Must have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| Metadata Size                 |
       +---------------+---------------*
       Total 18 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x58          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x12          | 0x00          | 0x02          | 0x10          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x17          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x12          | 0x10          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x68 ('h')    | 0x65 ('e')    |
        +---------------+---------------+---------------+---------------+
      44| 0x6c ('l')    | 0x6c ('l')    | 0x6f ('o')    |
        +---------------+---------------+---------------+
    UPR_DELETION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x58
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x12
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0210
    Total body   (8-11) : 0x00000017
    Opaque       (12-15): 0x00001210
    CAS          (16-23): 0x0000000000000000
      by seqno   (24-31): 0x0000000000000005
      rev seqno  (32-39): 0x0000000000000001
      nmeta      (40-41): 0x0000
    Key          (42-46): hello

###Expiration (0x59)
Tells the consumer that the message contains a key expiration.

The request:
* Must have extras
* Must have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| Metadata Size                 |
       +---------------+---------------*
       Total 18 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x59          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x12          | 0x00          | 0x02          | 0x10          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x17          |
        +---------------+---------------+---------------+---------------+
      12| 0x00          | 0x00          | 0x12          | 0x10          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x68 ('h')    | 0x65 ('e')    |
        +---------------+---------------+---------------+---------------+
      44| 0x6c ('l')    | 0x6c ('l')    | 0x6f ('o')    |
        +---------------+---------------+---------------+
    UPR_EXPIRATION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x59
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x12
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0210
    Total body   (8-11) : 0x00000017
    Opaque       (12-15): 0x00001210
    CAS          (16-23): 0x0000000000000000
      by seqno   (24-31): 0x0000000000000005
      rev seqno  (32-39): 0x0000000000000001
      nmeta      (40-41): 0x0000
    Key          (42-46): hello

###Flush (0x5a)
Tells the consumer to delete all of its data for a given vbucket.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x5a          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_FLUSH command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x5a
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

###Set VBucket State (0x5b)

The Set VBucket message is used during the VBucket takeover process to
hand off ownership of a VBucket between two nodes. The message format
as well as the state values for this operation is below.

The request:
* Must have extras
* Must not have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| state         |               |               |               |
       +---------------+---------------+---------------+---------------+
       Total 1 byte

State may have the following values:

* Active (0x01) - Changes the VBucket on the consumer side to active state.
* Pending (0x02) - Changes the VBucket on the consumer side to pending state.
* Replica (0x03) - Changes the VBucket on the consumer side to replica state.
* Dead (0x04) - Changes the VBucket on the consumer side to dead state.

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x5b          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x01          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x04          |
        +---------------+
    UPR_SET_VBUCKET_STATE command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x5b
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x01
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000001
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      state      (24)   : 0x4 (dead)
