- Follow / Following Service
- Most common choice is a graph database
##### Why is Graph database not a good choice?
- We are not using any sophisticated algorithm
- Managing a graph DB is painful
- DB is expensive
- Pagination is very tricky
##### Example
![[Pasted image 20250406124611.png]]

![[Pasted image 20250406124704.png]]
##### Querying
![[Pasted image 20250406124748.png]]
- Followers of B
	- SELECT * FROM edges WHERE dest = B
		- Query needs to be sent to all shards, merged and return
- People B follows
	- SELECT * FROM edges WHERE src = B
		- Answered from 1 shard
##### Optimize above for both scenarios
![[Pasted image 20250406124947.png]]
![[Pasted image 20250406125004.png]]
##### Solution

![[Pasted image 20250406125106.png]]

![[Pasted image 20250406125122.png]]
