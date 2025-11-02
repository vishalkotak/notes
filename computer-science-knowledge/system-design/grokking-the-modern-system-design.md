
## Key Value Store

#### Requirements
###### Functional requirements
- Configurable service
- Ability to always write (High availability)
- Hardware heterogeneity
###### Non Functional requirements
- Scalability
- Fault Tolerance
#### API Design
- get function:
	- get(key)
- put function:
	- put(key, value)
#### Ensure Scalability and Replication
##### Add Scalability
Consider we have *m* nodes and we want to split keys across those nodes. The common functionality is to execute the following: hash(key) % *m* = node_index.

**Disadvantages**:
- Adding or deleting a node changes *m* causing rebalancing
##### Consistent Hashing
Consistent hashing is a technique for distributing keys across nodes in a cluster by mapping both keys and nodes to a circular hash space, ensuring that each key is assigned to the nearest node on the ring, while minimizing the amount of data redistribution when nodes are added or removed.

**Disadvantages**
- Some nodes can become hotspots leading to non-distribution across a single server
###### Use Virtual Nodes
In consistent hashing, virtual nodes are logical subdivisions of a physical node that are distributed across the hash ring to improve load balancing and fault tolerance.

**Advantages**
- Supports heterogeneity because nodes with more computation capability can have more virtual nodes
- If a node goes down, other nodes can take the load responsibility because virtual nodes are divided across the ring
##### Data Replication

###### Primary Secondary Approach
- Similar to master replica architecture
- Master servers the write requests and replica serves the read requests

**Disadvantages**
- Master going down can make our requirements not be met as writing to the kv store will take a hit
###### Peer to Peer Approach
- All nodes can act as primary and replicate data to other nodes
- Both read and write is allowed on all nodes

**Details**
- Apply a consistent hash function to the key to determine its position on the hash ring
- Move clockwise from the key's position to find the first node on the ring; this node becomes the **coordinator**
- The coordinator replicates the key to the next n-1 nodes clockwise on the ring
- To avoid placing replicas on the same physical node, the preference list skips virtual nodes belonging to the same physical node
#### Versioning Data and Achieving Configurability
##### Data Versioning
Data needs to be replicated across multiple nodes. But the nodes can have network partition and become out of sync. In order to check for the latest value, we need to find the ordering of updates. The simplest approach would be to use timestamps. But the issue with timestamps is that clocks can become out of sync in distributed systems leading to issues with ordering. We can use **Vector Clocks** to solve this issue.
###### Vector Clock details: [[vector-clocks]]
**Disadvantages**:
- Vector clock size == Count(Nodes). Too many nodes can make the size of vector clocks huge taking up bandwidth (during transmission), metadata size
- Node addition and removal can cause Vector clock size to change
- If two events happened concurrently, we will not be able to determine ordering
##### Configurability
![[Pasted image 20250119114838.png]]
#### Enable Fault Tolerance and Failure Detection

##### Handle Temporary Failures

###### Usual scenario
- A write is sent to the leader
- The leader replicates the data on the followers and receives acknowledgement
**Disadvantages**
- Leader might be down or unreachable causing leader election and slowing down the system
**Solution**: Sloppy Quorum 
- System prioritizes availability over consistency
- If the leader is unavailable, the system maintains a preference list. A healthy node is selected from the list
- The node writes the data in temporary storage using a mechanism called hinted handoff
- When the original leader comes back online, the fallback node sends data to the leader

## Content Delivery Network (CDN)


## Distributed Cache
A distributed cache is a caching system where multiple cache servers coordinate to store frequently accessed data. Caches use the locality of reference principle.
#### Writing policies
![[Pasted image 20250119174630.png]]
#### Eviction policies
- Least Recently Used (LRU)
- Most Recently Used (MRU)
- Least Frequently Used (LFU)
- Most Frequently Used (MFU)
#### Cache Invalidation
- Done through Time to Live (TTL)
- Approaches:
	- Active expiration: Background thread removes record that have crossed TTL
	- Passive expiration: Methods checks for TTL during data access
#### Hash Function
- Locate the cache server where the data might be located
- Locate the record within the cache server
#### Linked List
- Doubly Linked List is the most favored implementation of LRU cache
#### Requirements
##### Functional
- Insert 
- Retrieve
##### Non Functional
- High performance (Consistent hashing, RAM storage, LRU eviction, read replicas)
- Scalability (Shards)
- High availability (Replication in data and out of data centers)
- Consistency
#### Detailed Design
##### Maintaining cache server list
- Configuration file deployed with each server - Complex deployment process, Monitoring needs to be done on the cache client layer
- Configuration file is stored in a centralized server - Deployment process is easy but monitoring is missing
- Configuration service - Handles the server list and also monitors the cache servers
##### Improving availability
- Shards
- Consistent hashing with virtual nodes
##### Internals of cache server
- Hash Table
- Doubly Linked List
##### Concurrent access to the hash table and doubly linked list
- Limited Locking: Locking certain parts of the data structures
- Offline eviction: Writes to the structures are not directly executed. Only when the cache miss is going up
- Generally DLL are pretty good with concurrency
#### Memcached
- Key-Value store with keys and values being strings. Data needs to be serialized into strings before storing
- Servers follow a **shared-nothing architecture**. No data sharing, synchronization or communication between servers
- High Throughput and Low latency. O(1) deterministic query speed
- Uses multi-threading
#### Redis
- Data Structure Store: Keys and Values can be of multiple data type
- Database: Persistence offered through **Append Only File** (AOF) and **Redis database (RDB) snapshot**
- Message broker: Can act as a message broker
- Replication
- Automatic failover
- Also is memcached compatible
- Decouples control and data plane
- Doesn't provide strong consistency due to asynchronous replication
- Single threaded
##### Redis Cluster
- Redis Sentinel
- Automatic sharding - primary and secondary nodes
- Cluster Manager detects failures and performs automatic failovers
##### Pipelines in Redis
- Uses client-server model. Client is blocked until the server responds with data
- Uses pipelining to speed up the process. This is done by combining multiple requests from the client side
- Reduces Round Trip Time (RTT)
- Server processes these requests independently 

![[Pasted image 20250119191459.png]]
## Distributed Message Queue

#### Advantages
- Improved performance
- Better reliability
- Granular scalability
- Easy decoupling
- Rate limiting
- Priority Queue
#### Functional Requirements
- Queue creation
- Send message
- Receive message
- Delete message
- Delete queue
#### Non Functional Requirements
 - Durability
 - Scalability
 - Availability
 - Performance
#### Strategies for ordering of messages in queue
- Best-effort ordering
	- The service will put the messages in the order they are received and not the order generated on the client side
- Strict Ordering
	- Messages are placed in the queue in the generation order on the client
	- How is the ordering guaranteed?
		- Sequence numbers when messages arrive on the server. This doesn't solve the ordering based on client side
		- Causality-based sorting: Every message sent by the client will have a timestamp. The issue with timestamp is clocks can go out of sync
		- Use time stamp based on synchronized clocks: We can synchronize the clocks based on NTP. If multiple clients send messages at the same time, we can use a client_id associated with it
#### Sorting
- Once messages arrive on the server, we need to sort them
- This will increase latency and decrease throughput
#### Managing Concurrency
- First Approach: Lock the queue when producer/consumer is put/extracting message from the queue
	- Slow performance
- Second Approach:
	- Producers and consumers send messages on a common port
	- OS queues these requests in the order they arrive
	- A single application thread dequeues these requests from the buffer and updates the queue:
		- put request: Adds the message to the queue
		- extract request: Removes and delivers a message from the queue
#### High Level Design
![[Pasted image 20250119214114.png]]
#### Message Replication
###### Primary-Secondary Replication
- Responsibility of primary host is to receive requests for the queue and handle replication
- We will have an internal cluster manager that maps queues to nodes as well as handles leader failure
###### Peer-To-Peer replication
- Any node can act as a primary for a queue and receive requests. 
- An external cluster manager such as Zookeeper is introduced for mapping queue to cluster 
##### Dead Letter Queue
- Handles messages that aren't consumed after maximum number of processing attempts made by the consumer
## Distributed Monitoring

#### Types of monitoring
- Server-side errors
- Client-side errors
#### Metrics
- Metrics define what should we measure and what units will be appropriate
- Collection of metrics should incur minimal performance penalty
##### Metrics Fetching Strategies
- Push
- Pull

**Code Instrumentation**: Embedding logging or monitoring code in our applications to collect information of interest.
#### Alerting
- Part of monitoring system that responds to changes in metric values and takes action
- Contains of two components:
	- A metric based threshold
	- Action to take when values fall outside the permitted range
#### Server Monitoring

**Single data center**
![[Pasted image 20250119154157.png]]

**Multi-data center**
![[Pasted image 20250119154301.png]]
#### Client-side monitoring system
##### Prober
- Run a different service across different infrastructure to send requests to the service. This can be used to monitor the reachability to our service
- Disadvantages:
	- Too many probers need to be deployed to get a good picture
	- This doesn't represent user behavior
##### Agent
- Prober embedded in the client application that sends service reports
- Collector, a service independent of the main service can collect these reports
- Collector needs to be on a different failure domain to not be impacted by the main service footprint
## Design YouTube

#### High Level Design
![[Pasted image 20250120151844.png]]
#### API Design

| Path                                                                                                        | Details                                                                                                    |
| ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| uploadVideo(user_id, video_file, category_id, title, description, tags, default_language, privacy_settings) |                                                                                                            |
| streamVideo(user_id, video_id, screen_resolution, user_bitrate, device_chipset)                             | user_bitrate: Bandwidth available on the client side. This will help in selected the correct quality video |
| searchVideo(user_id, search_string, length, quality, upload_date)                                           |                                                                                                            |
| viewThumbnails(user_id, video_id)                                                                           |                                                                                                            |
| likeDislike(user_id, video_id, like)                                                                        |                                                                                                            |
| commentVideo(user_id, video_id, comment_text)                                                               |                                                                                                            |
#### Database schema
![[Pasted image 20250120145351.png]]
#### Duplicate Videos
- Duplication can be solved with simple techniques like locality-sensitive hashing
- Complex techniques such as block matching algorithms (BMA) and phase correlation
#### Encode
- For any video segment with dynamic colors and high depth, we'll encode it differently from a video with fewer colors. This means that a not-so dynamic segment will be encoded such that it's compressed more to save additional storage space.
## Quora
#### High Level Design
![[Pasted image 20250120171235.png]]

#### API Design

| Name                                                                    | Details |
| ----------------------------------------------------------------------- | ------- |
| postQuestion(user_id, question, description, topic_label, video, image) |         |
| postAnswer(user_id, question_id, answer_text, video, image)             |         |
| upvote(user_id, question_id, answer_id)                                 |         |
| comment(user_id, answer_id, comment_text)                               |         |
| search(user_id, search_text)                                            |         |
|                                                                         |         |
#### Similar Problem: Reddit
![[Pasted image 20250120165934.png]]
## Google Maps

Reference: https://www.youtube.com/watch?v=jk3yvVfNvds

![[Pasted image 20250120191557.png]]
- Location Service: Pinpoints the user's current location
- Traffic Update Service: Monitors and updates real-time traffic conditions
- Map Update Service: Updates or adds new routes based on user data

![[Pasted image 20250120192154.png]]
- Map Service: Acts as the central hub, orchestrating various services to fulfill user requests
- Graph Processing Service: Calculates the shortest and most efficient routes between two points
- Segment Service: Contains details about segments with entry and exit points
- Third-Party Data Manager: Integrates and manages data from various third-party providers
- Historical Data Service: Stores and manages historical traffic patterns and travel times
#### API Design

| Function                                       | Description                                                                                                             |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| currLocation(location)                         | The `currLocation` function displays the user’s location on the map                                                     |
| findRoute(source, destination, transport_type) | The `findRoute` function helps find the optimal route between two points.                                               |
| directions(curr_location)                      | The `directions` function helps us get alerts in the form of texts or sounds that indicate when and where to turn next. |
![[Pasted image 20250120195945.png]]
## Twitter
#### High Level Design

![[Pasted image 20250121074436.png]]
#### API Design

| Path                                                                                                                                                     | Description                                                                                         |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| postTweet(user_id, access_type, tweet_type, content, tweet_length, media_field, post_time, tweet_location, list_of_used_hashtags, list_of_tagged_people) | The POST method is used to send the Tweet to the server from the user through the `/postTweet` API. |
| likeTweet(user_id, tweet_id, tweeted_user_id, user_location)                                                                                             | The `/likeTweet` API is used when users like public Tweets.                                         |
| replyTweet(user_id, tweet_id, tweeted_user_id, reply_type, reply_content, reply_length)                                                                  | The `/replyTweet` API is used when users reply to public Tweets.                                    |
| searchTweet(user_id, search_term, max_result, exclude, media_field, expansions, sort_order, next_token, user_location)                                   | When the user searches any keyword in the home timeline, the GET method is used.                    |
| viewHome_timeline(user_id, tweets_count, max_result, exclude, next_token, user_location)                                                                 |                                                                                                     |
| followAccount(account_id, followed_account_id)                                                                                                           |                                                                                                     |
| retweet(user_id, tweet_id, retweet_user_id)                                                                                                              |                                                                                                     |
