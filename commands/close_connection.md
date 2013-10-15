
##Close Connection

The close connection command can be used to close an UPR Prodcuer or Consumer connection.

####Binary Implementation

    Close Connection Binary Request

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       80      |       B6      |       00      |       0C      |
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
    Close Connection command
    Field        (offset) (value)
    Magic        (0)    : 0x80                (Request)
    Opcode       (1)    : 0xB6                (Close Connection)
    Key length   (2,3)  : 0x000C              (12)
    Extra length (4)    : 0x08                (8)
    Data type    (5)    : 0x00                (Field not used)
    VBucket      (6,7)  : 0x0000              (Field not used)
    Total body   (8-11) : 0x00000014          (20)
    Opaque       (12-15): 0x00000000          (Field not used)
    CAS          (16-23): 0x0000000000000000  (Field not used)
	Conn. UUID   (24-31): 0x00000000deadbeef  (3735928559)
	Conn. Name   (32-43): "mystreamname"

    Close Connection Binary Response

    Byte/     0       |       1       |       2       |       3       |
       /              |               |               |               |
      |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
      +---------------+---------------+---------------+---------------+
     0|       81      |       B6      |       00      |       00      |
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
    Close Connection command
    Field        (offset) (value)
    Magic        (0)    : 0x81 	              (Response)
    Opcode       (1)    : 0xB6                (Close Connection)
    Key length   (2,3)  : 0x0000              (Field not used)
    Extra length (4)    : 0x00                (0)
    Data type    (5)    : 0x00                (Field not used)
    Status       (6,7)  : 0x0000              (Success)
    Total body   (8-11) : 0x00000000          (0)
    Opaque       (12-15): 0x00000000
    CAS          (16-23): 0x0000000000000000  (Field not used)

#####Extra Fields

**Connection Name**

Used to identify a specific connection.

**Connection UUID**

A unique UUID that can be used to differentiate different connections that share the same name, but exist at different periods in time.

#####Returns

A status code indicating whether or not the operation was successful

#####Errors

**PROTOCOL_BINARY_RESPONSE_KEY_ENOENT (0x01)**

If the connection cannot be closed because a connection does not exist for the given Connection Name and Connection UUID.

**PROTOCOL_BINARY_RESPONSE_EINVAL (0x04)**

If data in this packet is malformed or incomplete then this error is returned.

**PROTOCOL_BINARY_RESPONSE_EINTERNAL (0x84)**

If the connection failed to be closed for any other reason.

#####Use Cases

The Close Connection command can be used to ensure that a connection is actually closed on a node. This command can be used to prevent races in connection teardown/reconnect.


