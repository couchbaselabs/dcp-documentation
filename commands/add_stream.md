
##Add Stream

The Add Stream command is used to add VBuckets to an existing UPR stream. This command will block until the UPR handshake for the new stream has finished. This message can specify multiple VBucket streams to be added.

####Binary Implementation

    Add Stream Binary Request

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       B3      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       0C      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       18      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       DE      |       AD      |       BE      |       EF      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       0A      |       00      |       0B      |
      +---------------+---------------+---------------+---------------+
    36|       6D      |       79      |       73      |       74      |
      +---------------+---------------+---------------+---------------+
    40|       72      |       65      |       61      |       6D      |
      +---------------+---------------+---------------+---------------+
    44|       6E      |       61      |       6D      |       65      |
      +---------------+---------------+---------------+---------------+


    Header breakdown
    Add Stream command
    Field        (offset) (value)
    Magic        (0)    : 0x80                (Request)
    Opcode       (1)    : 0xB3                (Add Stream)
    Key length   (2,3)  : 0x0000              (Field not used)
    Extra length (4)    : 0x0C                (12)
    Data type    (5)    : 0x00                (Field not used)
    VBucket      (6,7)  : 0x0000              (Field not used)
    Total body   (8-11) : 0x00000018          (24)
    Opaque       (12-15): 0x00000000          (Field not used)
    CAS          (16-23): 0x0000000000000000  (Field not used)
	Conn. UUID   (24-31): 0x00000000deadbeef  (3735928559)
	VBucket      (32-33): 0x000A              (10)
	VBucket      (34-35): 0x000B              (11)
	Conn. Name   (36-47): "mystreamname"

    Add Stream Binary Response

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       B3      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       08      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       08      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       0A      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    28|       00      |       0B      |       00      |       07      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    Add Stream command
    Field         (offset) (value)
    Magic         (0)    : 0x81 	              (Response)
    Opcode        (1)    : 0xB3                (Add Stream)
    Key length    (2,3)  : 0x0000              (Field not used)
    Extra length  (4)    : 0x08                (8)
    Data type     (5)    : 0x00                (Field not used)
    Status        (6,7)  : 0x0000              (Success)
    Total body    (8-11) : 0x00000008          (8)
    Opaque        (12-15): 0x00000000
    CAS           (16-23): 0x0000000000000000  (Field not used)
	VBucket       (24-25): 0x000A              (10)
	Stream Status (26-27): 0x0000              (Success)
	VBucket       (28-29): 0x000B              (11)
	Stream Status (30-31): 0x0007              (Not My VBucket)


#####Extra Fields

**Connection Name**

Used to identify a specific connection.

**Connection UUID**

A unique UUID that can be used to differentiate different streams that share the same name, but exist at different periods in time.

#####Returns

A status code indicating whether or not the operation was successful

#####Errors

**PROTOCOL_BINARY_RESPONSE_KEY_EEXISTS (0x02)**

If the stream cannot be created because a stream already exists for the given VBucket.

**PROTOCOL_BINARY_RESPONSE_EINVAL (0x04)**

If data in this packet is malformed or incomplete then this error is returned.

**PROTOCOL_BINARY_RESPONSE_NOT_MY_VBUCKET (0x07)**

If the consumer does not have the given VBucket then this error is returned.

**PROTOCOL_BINARY_RESPONSE_EINTERNAL (0x84)**

If the stream failed to be created for any other reason. This most likely means that the UPR handshake failed for some reason.

#####Use Cases

The Add Stream command is used by applications that want to be able to ask a Couchbase node to initiate a stream with their application. The node that handles the Add Stream request will always be the consumer in an UPR stream.


