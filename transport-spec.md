#UPR Transport Specification

##Terminology

* **Consumer** - The endpoint in a connection that is responsible for requesting different kinds of data. The consumer is responsible for doing some sort of processing with the data sent from the consumer.
* **Producer** - The endpoint in a connection that is responsible for producing data for given requests.
* **Stream** - A stream is a series of messages sent over a given period of time that are generate from a particular request message.
* **Snapshot** - A unique sequence of keys that is sent by a stream.

##Messages

###UPR Header

#####Request Header

	UPR Header

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       50      |       00      |       00      |
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
	Opcode         (1)     : 0x50 (UPR Request)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x0000
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000000A
    Cas            (16-23) : 0x0000000000000000

#####Response Header

	UPR Header

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       50      |       00      |       00      |
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
    Magic          (0)     : 0x81 (Response)
	Opcode         (1)     : 0x50 (UPR Stream)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x0000
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000000A
    Cas            (16-23) : 0x0000000000000000

#####Magic

Used to specify that the packet being recieved is a valid memcached binary protocol packet. Below are the currently available magic values.

* **Request** (0x80) - Specifies that the packet is a memcached request packet.
* **Response** (0x81) - Specifies that the packet is a memcached response packet.

#####Opcode

The opcode field is used in order to specify the what type of message the client is receiving. Below are the currently available opcodes.

* **Stream Request** (0x50) - Sent by the consumer side to the producer specifying that the consumer want some piece of data (Ex. XDCR).
* **Failover Log Request** (0x51) - Sent by the consumer in order to get the producers failover log.
* **Stream Start** (0x52) - Sent to tell the consumer that the producer will start streaming messages.
* **Stream End** (0x53) - Sent to tell the consumer that the producer will has no more messages to stream.
* **Snapshot Start** (0x54) - Sent by the producer to tell the consumer that a new snapshot is being sent.
* **Snapshot End** (0x55) - Sent by the producer to tell the consumer that the current snapshot is finshed.
* **Mutation** (0x56) - Tells the consumer that the message contains a key mutation.
* **Deletion** (0x57) - Tells the consumer that the message contains a key deletion.
* **Expiration** (0x58) - Tells the consumer that the message contains a key expiration.
* **Flush** (0x59) - Tells the consumer to delete all of its data for a given vbucket.
* **Set VBucket State** (0x5A) - Tells the consumer to change the VBuckets state.

#####Key Length

Specifies the length of the key field. This field should be set to 0 if there is no key in the data section of the packet.

#####Extra Length

Specifies the length of the extra data that is part of this header. Extra data is considered anything after the header excluding the key and value.

#####VBucket

The Vbucket ID for that this message is working on. Set to 0 if this command is not specific to a particular VBucket. (Only in the request header)

#####Status

The status of this operation. The status is 0 for success or non-zero to indicate and error. (Only in the response header)

#####Total Body

The length of the data that follows this header message.

#####Opaque

The opaque field contains the Request ID which is a 32-bit number that is used to identify the request that caused the producer to send a given piece of data.

#####CAS

The check and set value associated with a given key. If this message is not specific to a particular key then this field is set to 0.

###Request and Response Messages

UPR request messages are used by the consumer in order to specify a piece of data or a stream of data that the consumer wishes to obtain from the producer. Each request message will be answered with a response message. This message will contain a succes code or an error code indicating that the data requested by the consumer cannot be obtained at that time.

#####Request Types

* **Stream Request** (0x01) - Requests that data be streamed from a given VBucket.
* **Failover Log Request** (0x02) - Requests that failover log information be provided by the producer.

#####Error Codes

* **OK** (0x00) - Specifies that the request was successful.
* **Not My VBucket** (0x07) - Specifies the the request was for a non-existent VBucket.
* **Rollback** (0x23) - Specifies that the consumer needs to rollback data before it can receive the data requested.

####Starting a Stream

In order to initial a stream from a vbucket the consumer must send the following command below. In order to initiate multiple stream the consumer needs to send multiple commands.

	UPR Stream Request Command

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       50      |       00      |       0E      |
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
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    40|       FF      |       FF      |       FF      |       FF      |
      +---------------+---------------+---------------+---------------+
    44|       FF      |       FF      |       FF      |       FF      |
      +---------------+---------------+---------------+---------------+
    48|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    52|       00      |       03      |       C5      |       8A      |
      +---------------+---------------+---------------+---------------+
    56|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    60|       00      |       00      |       0A      |       78      |
      +---------------+---------------+---------------+---------------+
    64|       72      |       65      |       70      |       6C      |
      +---------------+---------------+---------------+---------------+
    68|       69      |       63      |       61      |       74      |
      +---------------+---------------+---------------+---------------+
    72|       69      |       6F      |       6E      |       5F      |
      +---------------+---------------+---------------+---------------+
    76|       31      |       32      |
      +---------------+---------------+

    UPR Stream Request
    Field          (offset)  (value)
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x50                 (UPR Stream)
    Key Length     (2-3)   : 0x000E               (14)
    Extra Length   (4)     : 0x28                 (40)
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000036           (56)
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
	Flags          (24-27) : 0x00000000
	Reserved       (28-31) : 0x00000000
	Start By Seqno (32-39) : 0x0000000000000000   (0)
    End By Seqno   (40-47) : 0xFFFFFFFFFFFFFFFF   (2^64)
    VBucket UUID   (48-55) : 0x000000000003C58A   (247178)
    High Seqno     (56-63) : 0x0000000000000A78   (2680)
    Group ID       (64-77) : "replication_12"

#####Fields

* **VBucket** - Specifies the vbucket that data should be streamed from.
* **Flags** - Used to specify extra information added in the extra section for modifying what the stream send.
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
     0|       81      |       50      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       00      |
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
	Opcode         (1)     : 0x50                 (UPR Stream)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x0000               (OK)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Not My VBucket Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       50      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       01      |
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
	Opcode         (1)     : 0x51                 (UPR Stream)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x0001               (Not My VBucket)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Rollback Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       50      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       23      |
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
	Opcode         (1)     : 0x50                 (UPR Stream)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x0023               (Rollback)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
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
     0|       80      |       51      |       00      |       00      |
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
	Opcode         (1)     : 0x51                 (UPR Failover Log)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

The consumer should expect to receive a response from the producer that contains a list of zero or more Vbucket UUID and High Seqno pairs.

    UPR Stream Rollback Response Message

	Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       51      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       00      |       00      |       00      |       00      |
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
	Opcode         (1)     : 0x51                 (UPR Failover Log)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x0000               (OK)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
    VBucket UUID   (24-31) : 0x0000000000000DF2   (3570)
    High Seqno     (32-39) : 0x000000000000F294   (62100)
    VBucket UUID   (40-47) : 0x0000000000007B43   (31555)
    High Seqno     (48-55) : 0x0000000000009321   (37665)

Note that in the message above we can have anywhere from 1-1024 VBucket UUID/High Seqno pairs. The number of pairs in the message can be calculated by dividing the body length by 16 (sizeof(VBucket UUID) + sizeof(HighSeqno)).

###Streams

Stream can be used in order to request a sequence of data from a given VBucket. After a stream has been successfully created the first packet that is seen by the client will be a stream start message. This packet structure is defined below.

    UPR Stream Start Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       52      |       00      |       00      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x52                 (Stream Start)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

An open stream will send packets in a series of snapshots. A snaphot is simply a series of packets that is guarenteed to contain a unique set of keys. Snapshots a signified by snapshot start and end messages. These messages are defined below.

    UPR Stream Snapshot Start Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       54      |       00      |       00      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x54                 (Snapshot Start)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

    UPR Stream Snapshot End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       55      |       00      |       00      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x55                 (Snaphot End)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Vbucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

After receiving an UPR stream start message the consumer will receive a series of UPR stream messsage that will specify mutations, deletes, and expirations.

    UPR Stream Mutation Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       56      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       00      |       00      |       0C      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x56                 (UPR Mutation)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000030           (48)
    Opaque         (12-15) : 0x0000002D           (45)
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
     0|       80      |       57      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       00      |       00      |       0C      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x57                 (UPR Deletion)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000029           (41)
    Opaque         (12-15) : 0x0000002D           (45)
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
     0|       80      |       58      |       00      |       05      |
      +---------------+---------------+---------------+---------------+
     4|       24      |       00      |       00      |       0C      |
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
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x58                 (UPR Expiration)
    Key Length     (2-3)   : 0x0005               (5)
    Extra Length   (4)     : 0x24                 (36)
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000029           (41)
    Opaque         (12-15) : 0x0000002D           (45)
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
     0|       80      |       59      |       00      |       00      |
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
    UPR Expiration Command
    Field        (offset) (value)
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x59                 (UPR Flush)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Vbucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000

Once a stream has finished streaming data or an error has occurred on that stream the consumer will be be notified with a stream end message.

#####Stream End Error Codes

The stream end message will contain a flags field in order to specify a reason for why the stream has ended. The error codes that may be received are shown below.

* OK (0x00) - The tap stream has finished without error.
* State Changed (0x01) - The state of the VBucket that is being streamed has changed to state that the consumer does not want to receive.

Examples of stream end packets are shown below.

    UPR Stream End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       53      |       00      |       00      |
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
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x80                 (Request)
	Opcode         (1)     : 0x53                 (UPR Stream End)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    Status         (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
	Flags          (24-27) : 0x00000000           (OK)

    UPR Stream End Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       53      |       00      |       00      |
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
    24|       00      |       00      |       00      |       01      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    UPR Header command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Request)
	Opcode         (1)     : 0x53                 (UPR Stream End)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Data Type      (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
    Flags          (24-27) : 0x00000001           (State Changed)


###Set VBucket

The Set VBucket message is used during the VBucket takeover process to hand off ownership of a VBucket between two nodes. The message format as well as the state values for this operation is below.

* Active (0x01) - Changes the VBucket on the consumer side to active state.
* Pending (0x02) - Changes the VBucket on the consumer side to pending state.
* Replica (0x03) - Changes the VBucket on the consumer side to replica state.
* Dead (0x04) - Changes the VBucket on the consumer side to dead state.

Below is the Set VBucket State packet:

    UPR Set VBucket Message

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       5A      |       00      |       00      |
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
    24|       01      |
      +---------------+

    Header breakdown
    UPR Set VBucket command
    Field        (offset) (value)
    Magic          (0)     : 0x81                 (Response)
	Opcode         (1)     : 0x5A                 (UPR Set VBucket)
    Key Length     (2-3)   : 0x0000
    Extra Length   (4)     : 0x00
    Request Type   (5)     : 0x00
    VBucket        (6-7)   : 0x000C               (12)
    Total Body     (8-11)  : 0x00000000
    Opaque         (12-15) : 0x0000002D           (45)
    Cas            (16-23) : 0x0000000000000000
	State		   (24)    : 0x01                 (Active)
