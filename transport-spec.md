
#UPR Transport Specification

##Terminology

* **Consumer** - The endpoint in a connection that is responsible for requesting different kinds of data. The consumer is responsible for doing some sort of processing with the data sent from the consumer.
* **Producer** - The endpoint in a connection that is responsible for producing data for given requests.
* **Stream** - A stream is a series of messages sent over a given period of time that are generate from a particular request message.
* **Snapshot** - A unique sequence of keys that is sent by a stream.

##Messages

###UPR Header

	UPR Header

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       01      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       0A      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header
    Field          (offset)  (value)
    Magic          (0)     : 0x80 (Request)
	Opcode         (1)     : 0x01 (UPR Request)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x00
    VBucket        (6-7)   : 0x0000
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000000A
    Cas            (16-23) : 0x0000000000000000

#####Magic

Used to specify that the packet being recieved is a valid memcached binary protocol packet. Below are the currently available magic values.

* **Request** (0x80) - Specifies that the packet is a memcached request packet.
* **Response** (0x81) - Specifies that the packet is a memcached response packet.

#####Opcode

The opcode field is used in order to specify the what type of message the client is receiving. Below are the currently available opcodes.

* **Request** (1) - Sent by the consumer side to the producer specifying that the consumer want some piece of data (Ex. XDCR).
* **Response** (2) - A producer message that is sent to the consumer for a certain requeste piece of data (Ex. XDCR).
* **Stream Begin** (3) - Means that you will recieve multiple responses for a request.
* **Stream Message** (4) - A single response that is contained in a stream
* **Stream Close** (5) - Says that the stream has ended.

#####Key Length

Specifies the length of the key field. This field should be set to 0 if there is no key in the data section of the packet.

#####Extra Length

Specifies the length of the extra data that is part of this header. Extra data is considered anything after the header excluding the key and value.

#####Request Type

A field for specifying the subtype for the opcode.

#####VBucket

The Vbucket ID for that this message is working on. Set to 0 if this command is not specific to a particular VBucket.

#####Total Body

The length of the data that follows this header message.

#####Request ID

The request ID is a 32-bit number that is used to identify the request that caused the producer to send a given piece of data.

#####CAS

The check and set value associated with a given key. If this message is not specific to a particular key then this field is set to 0.

###Request and Response Messages

UPR request messages are used by the consumer in order to specify a piece of data or a stream of data that the consumer wishes to obtain from the producer. Each request message will be answered with a response message. This message will contain a succes code or an error code indicating that the data requested by the consumer cannot be obtained at that time.

#####Request Types

* **Stream Request** (1) - Requests that data be streamed from a given VBucket.
* **Failover Log Request** (2) - Requests that failover log information be provided by the producer.

#####Response Types

* **OK** (1) - Specifies that the request was successful.
* **Not My VBucket** (2) - Specifies the the request was for a non-existent VBucket.
* **Rollback** (3) - Specifies that the consumer needs to rollback data before it can receive the data requested.

####Starting a Stream

In order to initial a stream from a vbucket the consumer must send the following command below. In order to initiate multiple stream the consumer needs to send multiple commands.

	UPR Stream Request Command

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       01      |       00      |       0E      |
      +---------------+---------------+---------------+---------------+
     4|       28      |       01      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       36      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+    
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    32|       FF      |       FF      |       FF      |       FF      |
      +---------------+---------------+---------------+---------------+
    36|       FF      |       FF      |       FF      |       FF      |
      +---------------+---------------+---------------+---------------+
    40|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    44|       00      |       03      |       C5      |       8A      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       00      |       00      |       0A      |       78      |
      +---------------+---------------+---------------+---------------+
    56|       72      |       65      |       70      |       6C      |
      +---------------+---------------+---------------+---------------+
    60|       69      |       63      |       61      |       74      |
      +---------------+---------------+---------------+---------------+
    64|       69      |       6F      |       6E      |       5F      |
      +---------------+---------------+---------------+---------------+
    68|       31      |       32      |
      +---------------+---------------+

    UPR Stream Request
    Field          (offset)  (value)
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x01                 (UPR Request)
    Key Length     (2-3)   : 0x000E               (14)
    Extra Length   (4)     : 0x28                 (40)
    Request Type   (5)     : 0x01                 (Stream Request)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000036           (56)
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
	Start By Seqno (24-31) : 0x0000000000000000   (0)
    End By Seqno   (32-39) : 0xFFFFFFFFFFFFFFFF   (2^64)
    VBucket UUID   (40-47) : 0x000000000003C58A   (247178)
    High Seqno     (48-55) : 0x0000000000000A78   (2680)
    Group ID       (56-69) : "replication_12"

#####Fields

* **VBucket** - Specifies the vbucket that data should be streamed from.
* **Start By Seqno** -  Specified the last by sequence number that has been seen by the consumer.
* **End By Seqno** - Specifies that the stream should be closed when the sequence number with this ID has been sent.
* **VBucket UUID** - A unique identifier that is generated that is assigned to each VBucket. This number is generated on an unclean shutdown or when a Vbucket becomes active.
* **High Seqno** - The high sequence number at the time that the VBucket UUID was generated.
* **Group ID** - A name that can be given to a stream. The length of this field should be specified in the key length section.


After a consumer sends a request to the server for a stream the server can response in one of the following ways.

    UPR Stream OK Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       02      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       01      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x02                 (UPR Response)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x01                 (OK)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Not My VBucket Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       02      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       02      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x02                 (UPR Response)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x02                 (Not My VBucket)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Rollback Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       02      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       03      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       45      |       A8      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x02                 (UPR Response)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x03                 (Rollback)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
    Rollback Seqno (14-31) : 0x00000000000045A8   (17832)

#####Fields

* **Rollback Seqno** - The rollback sequence number tells the consumer that it has newer mutations than the producer does. As a result the consumer needs to remove those new mutations. The sequence number can be used by the consumer to figure our which mutations to remove.

####Requesting A Failover Log

In order to request failover log information the following packet should be sent to the producer.

    UPR Failover Log Request

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       01      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       02      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Request Failover Log Command
    Field        (offset) (value)
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x01                 (UPR Request)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x02                 (Failover Log Request)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

The consumer should expect to receive a response from the producer that contains a list of zero or more Vbucket UUID and High Seqno pairs.

    UPR Stream Rollback Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       02      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       02      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       0D      |       F2      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       F2      |       94      |
      +---------------+---------------+---------------+---------------+
    40|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    44|       00      |       00      |       7B      |       43      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       00      |       00      |       93      |       21      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x02                 (UPR Response)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x02                 (Failover Log Response)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
    VBucket UUID   (24-31) : 0x0000000000000DF2   (3570)
    High Seqno     (32-39) : 0x000000000000F294   (62100)
    VBucket UUID   (40-47) : 0x0000000000007B43   (31555)
    High Seqno     (48-55) : 0x0000000000009321   (37665)

Note that in the message above we can have anywhere from 1-1024 VBucket UUID/High Seqno pairs. The number of pairs in the message can be calculated by dividing the body length by 16 (sizeof(VBucket UUID) + sizeof(HighSeqno)).

###Streams

Stream can be used in order to request a sequence of data from a given VBucket. A stream is created by sending the stream request message that has been defined above. Below we define the packets that can may be received by the consumer after a stream has been created successfully.

#####Stream Message Types

* Snapshot Start (1)
* Snapshot End (2)
* Mutation (3)
* Deletion (4)
* Expiration (5)
* Flush (6)

After a stream has been successfully created the first packet that is seen by the client will be a stream start message. This packet structure is defined below.

    UPR Stream Start Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       03      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x03                 (UPR Stream Start)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

An open stream will send packets in a series of snapshots. A snaphot is simply a series of packets that is guarenteed to contain a unique set of keys. Snapshots a signified by snapshot start and end messages. These messages are defined below.

    UPR Stream Snapshot Start Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       01      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x01                 (Snapshot Start)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Snapshot End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       02      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
	Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x02                 (Snapshot End)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

After receiving an UPR stream start message the consumer will receive a series of UPR stream messsage that will specify mutations, deletes, and expirations.

    UPR Stream Mutation Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       03      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       30      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       A5      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       08      |       CB      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       00      |       03      |
      +---------------+---------------+---------------+---------------+
    40|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    44|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       6D      |       79      |       6B      |       65      |
      +---------------+---------------+---------------+---------------+
    56|       79      |       6D      |       79      |       76      |
      +---------------+---------------+---------------+---------------+
    60|       61      |       6C      |       75      |       65      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Request Type   (5)     : 0x03                 (UPR Mutation)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000030           (48)
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x00000000000000A5   (165)
    By Seqno       (24-31) : 0x00000000000008CB   (2251)
    Rev Seqno      (32-39) : 0x0000000000000003   (3)
	Item Flags     (40-43) : 0x00000000           (0)
    Item Exp       (44-47) : 0x00000000           (0)
    Lock time      (48-51) : 0x00000000           (0)
    Key            (52-56) : "mykey"
	Value          (57-63) : "myvalue"

    UPR Stream Delete Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       04      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       29      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       A5      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       08      |       CB      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       00      |       03      |
      +---------------+---------------+---------------+---------------+
    40|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    44|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       6D      |       79      |       6B      |       65      |
      +---------------+---------------+---------------+---------------+
    56|       79      |
      +---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Request Type   (5)     : 0x04                 (UPR Deletion)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000029           (41)
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x00000000000000A5   (165)
    By Seqno       (24-31) : 0x00000000000008CB   (2251)
    Rev Seqno      (32-39) : 0x0000000000000003   (3)
	Item Flags     (40-43) : 0x00000000           (0)
    Item Exp       (44-47) : 0x00000000           (0)
    Lock time      (48-51) : 0x00000000           (0)
    Key            (52-56) : "mykey"

    UPR Stream Expiration Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       05      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       29      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       A5      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       00      |       08      |       CB      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       00      |       03      |
      +---------------+---------------+---------------+---------------+
    40|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    44|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       6D      |       79      |       6B      |       65      |
      +---------------+---------------+---------------+---------------+
    56|       79      |
      +---------------+

    Header breakdown
    UPR Expiration Command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Request Type   (5)     : 0x05                 (UPR Expiration)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000029           (41)
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x00000000000000A5   (165)
    By Seqno       (24-31) : 0x00000000000008CB   (2251)
    Rev Seqno      (32-39) : 0x0000000000000003   (3)
	Item Flags     (40-43) : 0x00000000           (0)
    Item Exp       (44-47) : 0x00000000           (0)
    Lock time      (48-51) : 0x00000000           (0)
    Key            (52-56) : "mykey"

    UPR Stream Flush Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       04      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       06      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Expiration Command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x04                 (UPR Stream Message)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x06                 (UPR Flush)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

Once a stream has finished streaming data or an error has occurred on that stream the consumer will be be notified with a stream end message.

#####Stream End Error Codes

* OK (1) - The tap stream has finished without error.
* State Changed (2) - The state of the VBucket that is being streamed has changed to state that the consumer does not want to receive.

Examples of stream end packets are shown below.

    UPR Stream End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       05      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       01      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x05                 (UPR Stream End)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x01                 (OK)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       05      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       02      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       2D      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x05                 (UPR Stream End)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x02                 (State Changed)
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Request ID     (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000


