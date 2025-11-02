### Requirements
- Able to calculate the number of active users on a site
	- For any given instant
	- For any possible time range
- Users may have multiple active sessions at the same time on a variety of different devices
### Estimates
### Components
#### Session Service
![[Pasted image 20250422125656.png]]
- When a user logs in, the session service can assign it a random id for the session
- Hit a session DB (partition by userId) to assign the next session id for that user
- User assigned to session service based on userId
- Users should ping the server to respond that they are still using the service and session needs to be kept active
- Use a background process to expire sessions that don't get heartbeats for
#### CDC
![[Pasted image 20250422130052.png]]
- We can capture the changes to the database and move it to Kafka sharded by UserId
- Flink can read these partitions and aggregate data based on our requirements

![[Pasted image 20250422134331.png]]

HyperLogLog can be stored in Redis for quick retrieval. We can store 1 minute session with count of unique users. The data will be an approximate as HLL are probabilistic data structures. For accurate results, we will query the start time and the end time slot of the requested user. 

We can have a time series database for the accurate counts so that you can query the ranges and get the results.

### Aprit Bhayani Course

https://drive.google.com/file/d/1C0-uhvdH-ITew1A9RbPGwVZu6NvF2cve/view

