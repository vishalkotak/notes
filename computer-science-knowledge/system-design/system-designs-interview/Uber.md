Arpit Reference: https://drive.google.com/file/d/1IhnHA9BeWwxhi2ENBBBKGDgWOZjL4AH4/view
Byte Byte Go: https://www.youtube.com/watch?v=M4lR_Va97cQ 
- This is a proximity service such as Yelp but has good explanation of Geohashes

Below image shows how locations will be compared against one another.

![[Pasted image 20250406123226.png]]
##### Approaches to hash location
![[Pasted image 20250406125856.png]]
##### Design

![[Pasted image 20250406123807.png]]

Redis for storing and query data
- In-Memory (Fast)
- Support Geo Queries
- Multi-Master and Multi-Replica setup
**Redis does not support EVAL LUA script execution on read replica**
	- Why? LUA scripts can contain update operations that cannot be executed in Read replica
	- If we fire EVAL on read replica, it will respond with a MOVED <IP_ADDRESS> of the Master replica
##### Availability with Low Latency
![[Pasted image 20250406124125.png]]
Instead of firing GEOSEARCH on one node/replica, split the query into smaller subqueries and fire on multiple nodes
- Even if one node is slow, other nodes respond
- Parallel execution makes computing quicker
- Send partial response if SLA breached
##### Hot Shard Problem
- Splitting queries into smaller regions addresses the concern
- Shard the data in such a way that peak load and low traffic regions are mixed well


