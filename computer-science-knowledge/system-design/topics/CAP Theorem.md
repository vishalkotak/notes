- **Consistency**: All nodes see the same data at the same time. When a write is made to one node, all subsequent reads from any node will return that updated value.
- **Availability**: Every request to a non-failing node receives a response, without the guarantee that it contains the most recent version of the data.
- **Partition Tolerance**: The system continues to operate despite arbitrary message loss or failure of part of the system (i.e., network partitions between nodes).
#### Use Cases Consistency
- Ticket Booking Systems
- E-commerce Inventory
- Financial Systems
#### Use Cases Availability 
- Social Media
- Content Platforms (like Netflix)
- Review Sites (like Yelp)
#### Prioritizing Consistency
- Distributed Transactions
- Single-Node Solutions
- Technology Choices
	- Postgres, MySQL
	- Google Spanner
	- DynamoDB with strong consistency
#### Prioritizing Availability
- Multiple Replicas
- Change Data Capture (CDC) - Send data to object storage
- Technology Choices
	- Cassandra 
	- DynamoDB
	- Redis
#### Advanced CAP Theorem Considerations
- You need a balance of both in the same system
- For this ticketing system, I'll prioritize consistency for booking transactions but optimize for availability when users are browsing and viewing events.
- For this dating app, I'll prioritize consistency for matching but optimize for availability when users are viewing profiles.
#### Different types of Consistency
- Strong
- Causal
- Eventual
- Read Your Own Writes
