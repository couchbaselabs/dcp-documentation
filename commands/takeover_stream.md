
##Takeover Stream

The Takeover Stream command is used to create a takeover stream for a given VBucket or to convert a normal stream to a takeover stream.

####Binary Implementation

    Takeover Stream Binary Request

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       B5      |       00      |       0C      |
      +---------------+---------------+---------------+---------------+
     4|       08      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       14      |
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
    32|       6D      |       79      |       73      |       74      |
      +---------------+---------------+---------------+---------------+
    36|       72      |       65      |       61      |       6D      |
      +---------------+---------------+---------------+---------------+
    40|       6E      |       61      |       6D      |       65      |
      +---------------+---------------+---------------+---------------+


    Header breakdown
    Takeover Stream command
    Field        (offset) (value)
    Magic        (0)    : 0x80                (Request)
    Opcode       (1)    : 0xB5                (Takeover Stream)
    Key length   (2,3)  : 0x000C              (12)
    Extra length (4)    : 0x08                (8)
    Data type    (5)    : 0x00                (Field not used)
    VBucket      (6,7)  : 0x0000              (Field not used)
    Total body   (8-11) : 0x00000014          (20)
    Opaque       (12-15): 0x00000000          (Field not used)
    CAS          (16-23): 0x0000000000000000  (Field not used)
	Stream UUID  (24-31): 0x00000000deadbeef  (3735928559)
	Stream Name  (32-43): "mystreamname"

    Takeover Stream Binary Response

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       B5      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
     4|       10      |       00      |       00      |       03      |
      +---------------+---------------+---------------+---------------+
     8|       00      |       00      |       00      |       10      |
      +---------------+---------------+---------------+---------------+
    12|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    16|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    20|       00      |       00      |       00      |       A7      |
      +---------------+---------------+---------------+---------------+
    24|       00      |       00      |       00      |       07      |
      +---------------+---------------+---------------+---------------+
    28|       4E      |       82      |       A9      |       7A      |
      +---------------+---------------+---------------+---------------+
    32|       00      |       00      |       00      |       00      |
      +---------------+---------------+---------------+---------------+
    36|       00      |       00      |       00      |       0E      |
      +---------------+---------------+---------------+---------------+

    Header breakdown
    Takeover Stream command
    Field        (offset) (value)
    Magic        (0)    : 0x81 	              (Response)
    Opcode       (1)    : 0xB5                (Takeover Stream)
    Key length   (2,3)  : 0x0000              (Field not used)
    Extra length (4)    : 0x00                (0)
    Data type    (5)    : 0x00                (Field not used)
    Status       (6,7)  : 0x0000              (Success)
    Total body   (8-11) : 0x00000000          (0)
    Opaque       (12-15): 0x00000000
    CAS          (16-23): 0x0000000000000000  (Field not used)

#####Extra Fields

**Stream UUID**

A unique UUID that can be used to differentiate different streams that share the same name, but exist at different periods in time.

#####Returns

A status code indicating whether or not the operation was successful

#####Errors

**PROTOCOL_BINARY_RESPONSE_EINVAL (0x04)**

If data in this packet is malformed or incomplete then this error is returned.

**PROTOCOL_BINARY_RESPONSE_NOT_MY_VBUCKET (0x07)**

If the consumer does not have the given VBucket then this error is returned.

**PROTOCOL_BINARY_RESPONSE_EINTERNAL (0x84)**

If the stream failed to be created for any other reason. This most likely means that the UPR handshake failed for some reason.

#####Use Cases

The Takeover Stream command is used in the final step of a VBucket takeover which handles transfering the last of the items from the old VBucket to the new VBucket.


