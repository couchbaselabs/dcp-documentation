
#UPR Notifier Connection

In order to provide a Notification API though UPR we will introduce a new connection type called a Notifier Connection which will have different behavior than the typical Consumer and Producer Connections. The purpose of this connection will be to relay notifications about data becoming ready to a Consumer. The Consumer will show interest in particular VBuckets by sending Stream Requests to the server. The streams created by Stream Requests will remain open until new items are ready for the client to stream. The client can then create a seperate Producer Connection to the server to receive the new data.

###Message Flow

![Figure 1](images/notifier_connection.jpg)

1. The Consumer will establish a TCP connection with the server and send an Open Connection message specifying that the connection will be an UPR Notifier Connection. This is specified by supplying the UPR Notifier Connection Flag (0x02) in the flags field of the Open Connection messge.

2. The server will respond with a message indicating that the UPR Notifier Connection has been created.

3. The Consumer will then send a Stream Request Message to the server. This message will contain information for the Consumer about the last mutation seen by the Consumer for a particular VBucket. The Start Sequence Number will be used to specify the sequence number of the last mutation number seen and the Consumer should set the High Sequence Number and VBucket UUID to the latest entry in the Consumers failover log. The End Sequence number will not be used and can just be set to 0.

4. The server will return a message indicating that the Stream Request was successful.

5. The Stream will remain open until a mutation containing a sequence number higher than the one specified in the Start Sequence Number field of the Stream Request.

6. Once a higher sequence number than the sequence number specified in the Start Sequence Number field is received for the VBucket the stream was created for the server will send the Consumer an End Stream message and close the stream. This End Stream message will indicate that new items are ready on the server.

