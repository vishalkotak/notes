## Chapter 6: Partitioning

The main reason for wanting to partition data is scalability. 
#### Partitioning and Replication
- Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes. Even though every record belongs to one partition, it may be stored on several nodes for fault tolerance
#### Partitioning of Key-Value data
- If partitioning is unfair, so that some partitions have more data or queries than others, it is called skewed
- A partition with disproportionately high load is called as a hot spot
#### Partitioning by Key Range
- Assign a continuous range of keys (from some min to max) to each partition
- The range of keys are not necessarily evenly spaced
- Within each partition, the keys are kept in a sorted order making range based querying extremely easy
- Can lead to hotspots. For example if timestamp is used as the partition key, and our process is writing data for the current day, all write queries will go to the same partition
#### Partitioning by Hash of Key
- Chose a suitable hash function to be executed on keys
- You can then assign each partition a range of hashes (rather than range of keys) and every hash that falls within the hash range, will be stored on that partition
- This will divide the load evenly but we lose the ability to do range scans
- Databases achieve a balance by having a composite key where the first part of the key is hashed to get the partition and the remaining part is used to sort the data inside the partition
- If the requests are coming for the same key, hash based partition can also lead to hot spots
#### Skewed workloads and Relieving hot spots
- You can have a random number appended on the start or end of the key to split the hashes across nodes but the searching will be difficult
- You don't have to do this for all the keys only the ones that are hot keys
#### Partitioning and Secondary Indexes
- Above approaches used primary key for partitioning
- Not all queries are focussed on the primary key
- Secondary indexes can be used for querying but the challenge of finding the partition is introduced
- Approaches for secondary index based partitioning:
	- Document-based partitioning
	- Term-based partitioning
#### Partitioning Secondary Indexes by Document
![[Pasted image 20250126091102.png]]
- Each partition is completely separate; each partition maintains its own secondary index covering only the documents in that partition
- Writing to the database requires finding the partition based on key, and then dealing with the partition that will hold the document
- This approach follows the concept of *local index* where every partition maintains its own index
- Approach to querying is called scatter-gather as we don't know which partition contains the data so we have to send the query to all partitions to find documents matching the filters on the secondary index
- This has a slowness problem introduced due to the tail latency because of the slowest partitions
#### Partitioning Secondary Indexes by Term
![[Pasted image 20250126091144.png]]
- Instead of a local index, you create a global index. Since the node containing the global index becomes the bottleneck, we can partition the global index across nodes
- Called as term-based index because the term we are looking for determines the partition of the index where terms are all the words in the document
- The index partitions can be done based on ranges or hashes as discussed above
- Read is faster now because you know which partition to go to
- Write is slower because we need to update the global index and the write can affect multiple partitions
#### Rebalancing partitions
- Needed because of the following:
	- Query throughput increases
	- Dataset size increases
	- Machine fails
#### Strategies for rebalancing
###### How not to do it: mod N
- When number of nodes changes, it can lead to too many movements across the nodes
###### Fixed Number of partitions
- Have more partitions than number of nodes. Consider 1000 partitions for 10 nodes so every node will have 100 partitions 
- When rebalancing needs to be done, move the whole partitions across nodes. The number of partitions nor the assignment of keys to partitions do not change
###### Dynamic Partitioning
- Start with a single partition and increase the number of partitions once the partition grows in size beyond a threshold
- Each node can handle multiple partitions
- This can work both with key-range based partitioning and hash-range based partitioning
###### Partitioning proportionally to nodes
- Have a fixed number of partitions per node
- Partition size increases as data increases but the number of nodes remain unchanged
- When you add a new node, it randomly choses a fixed number of existing partitions and then takes ownership of one half of each of the split partitions while leaving the other half of the partition in place
#### Operations: Automatic or Manual Rebalancing
- Prefer a balance where the rebalancing is automatic but a manual operator has to commit before the rebalancing takes effect
#### Request Routing
- Three approaches for routing requests:
	- Allow clients to connect to any node. If data is not on that node, it can route the request to the node containing the data
	- Send all requests to a routing layer which is aware of the partitions
	- Client should be aware of the partitions and send the request accordingly 
- In all the above approaches, some component needs to keep track of the changes
- Use coordination service such as Zookeeper. Zookeeper maintains the authoritative mapping of partitions to nodes 
- Alternative approach is to use gossip protocol to share updates with other nodes. And use the first approach above for routing. More complexity on node but no new component
## Chapter 7: Transactions

## Chapter 10: Batch Processing

Three types of services exist today:
- Request / Response services
- Batch processing (offline systems)
- Stream processing systems (near real-time systems)
#### Batch Processing with Unix Tools

Problem: Consider large log file which captures HTTP request and response headers and we need to find the top 5 URL's.

Approach 1: Unix commands using chain of commands
- Commands such as sort on Linux distribute sorting functionality across cores for large data 
- If we use SSTables and LSM trees, we can perform merging and compaction and reduce the size of data. Also, we can use merge sort that has sequential access patterns on disk
	
Approach 2: Custom program using hashmap
- If the files are small where the hashmap can fit in memory, then this is a good option


