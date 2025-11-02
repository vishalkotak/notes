#### Steps when a message arrives at the broker

###### Broker receives the Produce Request
- Broker's network thread reads the incoming request
- The request is then placed into a request queue for processing
###### I/O Thread Processes the Request
- An I/O thread processes the request from the queue
- The broker validates the following:
	- Topic existence
	- Partition validity
	- Producer ID and Sequence Number (For idempotence and exactly-once) semantics
###### Messages appending to Log Segment
- Message Log Structure
	- Each partition is append only log
	- Logs are divided into segments (1 GB)
- Appending Process
	- Message is assigned an offset (Monotonically Increasing Number)
	- Append to the active segment file
- Disk Write:
	- Kafka uses zero-copy to write effectively
	- OS page cache is utilized for faster writes
###### Message Indexing
- Offset Index: Maps offset to file positions
- Time Index: Maps timestamps to offsets
###### Data Replication
- The leader first writes to its local copy
- Follower broker pull the data from the leader. They acknowledge when they have replicated the message
- Controlled by the acks configuration. Based on which the producer is notified
- Acknowledgements consists of the partition number, offset and any error codes if applicable
#### Difference between Controllers and Brokers

![[Pasted image 20250224132243.png]]
- Active controller is a broker but not every broker is a controller
#### Steps when Producer connects to the bootstrap server

- The producer selects one of the server from the bootstrap server list and establishes a TCP connection
- The producer sends a metadata request to the server to obtain:
	- List of brokers
	- Partition information for target topic(s)
	- Leader broker for each partition
- The producer receives the MetaDataResponse with the above fields
- The producer sets up a record accumulator for batching messages. Each partition has its own buffer in the accumulator.
- Based on the partition logic, the producer decides which partition to send to. It directly communicates with the leader of that partition.
- If the partition leader changes, a NOT_LEADER_FOR_PARTITION error occurs, the producer:
	- Invalidates old metadata
	- Requests new metadata from the cluster to re-route messages
#### Consumer Partition
- If a broker records a message as consumed immediately every time it handed out over the network, then if the consumer fails to process the message, the message can be lost.
- In order to solve this problem, many messaging systems will ask the consumers to acknowledge the message. But what if the consumer fails after processing during acknowledgement. This can lead to duplication. Also, performance challenge is introduced because brokers need to keep track of state of messages.
- Kafka maintains the offset per partition per consumer so it is just one number that is being tracked.
#### Timeline of replication from leader to follower
- Follower sends FetchRequest (offset=1000)
- Leader responds with Request (100, 101, 102), High Watermark = 102
- Follower appends logs, updated Log End Offset = 103, High Watermark = 102
- In-Sync replica updated since follower caught up
#### Concept of In-Sync replicas
![[Pasted image 20250224172246.png]]
#### Kafka Producer Routing Strategies
- Default Partitioner
	- Key-Based Hashing - Mumur2 hashing algorithm
	- Round-Robin  - If no key is provided
- Round Robin Partitioner
- Uniform Sticky Partitioner
	- Assigns messages to a sticky partition until a batch size or time limit is reached. Designed to reduce latency by minimizing the number of brokers the producer needs to talk to. 
- Custom Partitioner
#### Kafka Consumer Routing Strategies
- Range (Default)
- Round Robin
- Sticky
- Cooperative Sticky
	- Consider 3 consumers - C1, C2, C3
	- Broker detects that C1 is down and partitions need to be rebalanced
	- Traditional system will stop all consumption increasing downtime
	- In the newer approach, C2 and C3 can continue processing partitions that are not going to be rebalanced as they are associated with a sticky session. This causes downtime only for the partitions from C1.
- Custom