#Open Connection (opcode 0x50)

Sent by an external entity to a producer or a consumer to create a logical channel.

The request:

* Must have an 8 byte extras section
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

The sequence number should be incremented by one each time you try to connect with a given name, so that the receiving side may decide to terminate connections with a lower sequence number.

Flags are specified as a bitmask in network byte order with the following bits defined:

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

    DCP_OPEN command
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

###Returns

A status code indicating whether or not the operation was successful.

###Errors

**PROTOCOL_BINARY_RESPONSE_EINVAL (0x04)**

If data in this packet is malformed or incomplete then this error is returned.

**(Disconnect)**

If the connection could not be created due to an internal error. Check the server logs if this happens.