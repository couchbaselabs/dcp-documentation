###Snapshot Marker (opcode 0x56)

Sent by the producer to tell the consumer that a new snapshot is being
sent. A snaphot is simply a series of commands that is guarenteed to
contain a unique set of keys.

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
       0| 0x80          | 0x56          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x09          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
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
      36| 0x00          | 0x00          | 0x00          | 0x08          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+

    UPR_SNAPSHOT_MARKER command
    Field           (offset) (value)
    Magic           (0)    : 0x80
    Opcode          (1)    : 0x56
    Key length      (2,3)  : 0x0000
    Extra length    (4)    : 0x14
    Data type       (5)    : 0x00
    Vbucket         (6,7)  : 0x0000
    Total body      (8-11) : 0x00000014
    Opaque          (12-15): 0xdeadbeef
    CAS             (16-23): 0x0000000000000000
      Start Seqno   (24-31): 0x0000000000000000
      End Seqno     (32-39): 0x0000000000000008
      Snapshot Type (40-43): 0x00000001 (disk)

Snapshot Type is defined as:

* 0x00000000 (memory) - Specifies that the snapshot contains in-meory items only.
* 0x00000001 (disk) - Specifies that the snapshot contains on-disk items only.
