- Value can be of any type
- Large objects that need to be stored as values should be stored in a different object store with references to the storage stored as value

Functional Requirements:
- Configurable Service: Higher availability and allow to choose different models of consistency
- Ability to always write
- Hardware irrelevant

Non Functional Requirements
- Scalable
- Available
- Fault Tolerant

Assumptions
- Auth is sorted
- Data Center is available
- HTTPS is utilized

Operations
- GET
- PUT
#### Scalability

- Each request will have a KEY
- We will compute the hash value of the KEY
- Then we find the remainder by taking KEY % number of nodes = X
- We will send the request to Node - X

Above operation will lead to lot of swaps if the number of nodes changes. Thus, we will go for a better solution in the name of consistent hashing.

- We assume of a conceptual ring of hashes from 0 to n - 1 where n is the number of available hash values
- We use each node's ID to calculate it's hash and map it to the ring
- We apply the same process for the request but the next node in the circle will end up processing the request

Any change in the number of nodes will only affect requests between two nodes. But this can also lead to non uniform distribution of load across nodes.

![[Pasted image 20231122113824.png]]

**Virtual Nodes**

- Instead of one hash function, we will use three hash functions to map the node id to three different spots
- Request will be hashed using the same hash function

![[Pasted image 20231122120922.png]]

**Data Replication**

- Primary-Secondary Approach (Master Slave)
- Peer to Peer Approach

Each data item will be replicated at n hosts, where n is a parameter configured per instance of the key-value store. 

![[Pasted image 20231122121818.png]]

