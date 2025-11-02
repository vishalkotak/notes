https://www.hellointerview.com/learn/system-design/deep-dives/consistent-hashing

Started with one database

![[Pasted image 20250411135843.png]]

Load spiked and partitioned the data across three nodes

![[Pasted image 20250411135908.png]]

**Approach 1 : Simple Modulo Hashing**
- Issue 1: New instance of database is added

![[Pasted image 20250411140050.png]]
- Issue 2: A database instance is removed

![[Pasted image 20250411140121.png]]
#### Consistent Hashing
![[Pasted image 20250411140210.png]]

**Fix 1: Adding a node**
![[Pasted image 20250411140313.png]]

**Fix 2: Removing a node**
![[Pasted image 20250411140328.png]]
#### Hot Spots
![[Pasted image 20250411140355.png]]
#### Consistent Hashing in Real World
- Redis
- Cassandra
- DynamoDB
- Content Delivery Networks
