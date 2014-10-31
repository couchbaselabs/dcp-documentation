#Concepts

DCP is build on four major architectural concepts that allow the protocol to be used to build a rich set of applications. Below are the major concepts.

###Order

The first major concept is order and it is the foundation for the rest of the major architectural concepts. Order is important because it allow applications to reason about causality of data or in other words allows an application to know if an operation occurred before or after another operation. DCP achieves order through the use of sequence numbers and failover logs. Sequence numbers are used to keep order on a single node and failover logs are used to resolve ording conflicts during failure scenarios.

###Restartability

Restartability is an important feature for any replication protocol that often has to deal with large amounts of data. In any distributed system components crash and when your moving large portions of data applications want to be able to resume from exactly where they left off and not have to resend any data in the case of a dropped connection. Due to the DCPs ordering mechanism restartability is possible from any point and means the server won't send any data that the application has already received even if the application has been disconnected for an extended period of time.

###Consistency

Some applications can only make decisions about the data they receive if they have a consistent view of the data in the database. DCP provides consistency through snapshots and allow applications know that they have seen all mutations in the database up to a certain sequence number.

###Performance

DCP provides high throughput and low latency by keeping all of the most recent data that needs to be replicated in memory. This means that most DCP connection will never have to read data off of disk.