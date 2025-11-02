**Remote Procedure Call** provide an abstraction of a local procedure call to developers by hiding the complexities of packing and sending function arguments to the remote server, receiving the return values and managing network retries.

![[Pasted image 20231122074822.png]]
#### Consistency Models

Consistency: In distributed systems, consistency may mean many things. One is that each replica node has the same view of data in a given point of time. The other is that each request gets the value of the recent write.

![[Pasted image 20231122075236.png]]

Eventual consistency is the weakest consistency model. This consistency model ensures that all replicas converge on a final value after a finite time and when no more writes are coming in. This ensures high availability. Example: DNS, Cassandra.

Casual Consistency works by categorizing operations into dependent and independent operations. Casual consistency preserves the order of the casually related operations.

![[Pasted image 20231122080101.png]]

Sequential Consistency: Sequential consistency is stronger than the casual consistency model. It preserves ordering specified by each client's program. No order maintained through a global clock.

Strict consistency or linearizability is the strongest consistency model. This model ensures that the read request from the previous replicas will get the latest value. *Quorum based replication???*
#### Spectrum of failure models

![[Pasted image 20231122081040.png]]

Fail Stop: A node in the distributed system halts permanently. However, the other nodes can still detect that node by communicating with it.

Crash: In this type of failure, a node in the distributed system halts silently and the other nodes can't detect that the node has stopped working.

Omission failure: Node fails to send or receive messages. There are two types of omission failures:
- Send omission failure: Node does not respond to the incoming request
- Receive omission failure: Node fails to receive the request and doesn't acknowledge it

Temporal failures: Node generates correct results but it is too late to be useful. Example: bad algorithms, a bad design strategy or loss of synchronization of processor clocks.

Byzantine failure: Node exhibits random behavior like transmitting arbitrary messages at arbitrary times, producing wrong results or stopping midway. This is mostly due to a malicious attack. (Most challenging to deal with.)
#### Availability

Availability is the percentage of time that some service or infrastructure is accessible to clients and is operated upon under normal conditions.

*A (in percent) = (Total Time - Amount of Time Service Was Down)/Total Time * 100*
#### Reliability

Probability that a service will perform its function for a specified time. Specifies how the service performs under varying operating conditions.

*Mean Time Between Failure (MTBF) = 
(Total Elapsed Time - Sum of Downtime) / Total Number of Failures

Mean Time To Repair (MTTR) = Total Maintenance Time / Total Number of Repairs*

- Strive for higher MTBF and lower MTTR.
- Availability is driven by time loss, frequency and impact of failures drive the measure of reliability
- A is the function of R. Value of R can change independently, the value of A depends on R.
#### Scalability

- Vertical Scaling
- Horizontal Scaling
#### Maintainability

Maintainability (M) is the probability that the service will restore its functions within a specified time of fault tolerance. Gives us insight into the systems's capability to undergo repairs and modifications while it's operational.
#### Fault Tolerance

Fault tolerance refers to a system's ability to execute persistently even if one or more components fail. 

**Replication**: We can replicate both services and data. 
![[Pasted image 20231122092436.png]]

**Checkpointing**: Technique that saves the system's state in stable storage when the system state is consistent. 

![[Pasted image 20231122092558.png]]







