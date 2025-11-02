![[Pasted image 20250420113959.png]]
![[Pasted image 20250420114010.png]]
### Deep Dives
#### Live Comments
- Server Side Events (SSE)
- Web Sockets
#### Support Millions of Concurrent Views
##### Horizontal Scaling with Load Balancer and Pub-Sub (Bad Solution)
- All Service instance will maintain a list of connections for evert video id.
![[Pasted image 20250420114720.png]]
- When we get a comment for a video id, we send the comment to all connections
- Not all connections for a video will be on the same service instance
- Introduce a Pub/Sub system. Anytime a new comment is added we will send it to the Pub/Sub system. All subscribers will receive the post and send the comment for that video id
![[Pasted image 20250420114943.png]]
###### Cons
- Wastage of resources
##### Pub/Sub partitioned into Channels per Video
![[Pasted image 20250420115018.png]]
Though we will not be sending it to all partitions, what if a service has a small distribution of connections of all videos. Thus, it will subscribe to every channel
##### Force the viewers of a single video on the same service instance
##### Scalable Dispatcher Instead of Pub/Sub
![[Pasted image 20250420115419.png]]

Example: [[Streaming a Million Likes Per Second]]