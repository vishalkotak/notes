#### Standard Queue
- Unlimited throughput
- Unlimited number of messages in the queue
- Default retention: **4 days, max of 14 days**
- Low Latency **(< 10 ms on publish and receive)**
- Limitation of **256KB per message**
- Can have duplicate messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)
- Message is persisted until consumer deletes it
##### Producing Messages
- Produced to SQS using the *SendMessageAPI*
##### Consuming Messages
- Receives up-to **10 messages** at a time
###### Ideal Setup
- Track metric on SQS Queue - QueueLength
	- *Approximate number of messages*
- Alarm for breach
- Use AutoScaling Groups on EC2 instances and scale the number of workers
#### Visibility Timeout
- After a message is polled by one consumer, it becomes invisible to other consumers
- By default, the message timeout is **30 seconds**
- After the message visibility timeout is over, the message is visible in SQS for consumption
- Consumer can call ChangeVisibility API to get more time
#### Long Polling
- Decreases the number of API calls
- Wait time = **1 sec to 20 sec**
- Can be enabled at the queue level or API request *WaitTimeSeconds*
#### FIFO Queue
- Limited throughput: 300 msg/sec without batching, 3000 msgs/s with batching
- Exactly once semantics (by removing duplicates using Deduplication ID)
- Messages are processed in order by consumer
- Ordering of messages by MessageGroupID (All messages in the same group are ordered)
