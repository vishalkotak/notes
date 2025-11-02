## Introduction to Distributed systems
- A distributed system is a system where components are located on different networked computers which communicate and coordinate their actions by passing messages to one another
- **Performance**: Degree to which a software system meets its objectives for timeliness
- **Scalability**: Ability of the system to handle a growing amount of work or ability to enlarge itself
- **Availability**: Probability that a system works as expected
#### Fallacies of distributed system
![[Pasted image 20250120071534.png]]
#### Properties of distributed system
- **Network asynchrony** is a property of communication networks that cannot provide strong guarantees around delivering events, e.g., a maximum amount of time a message requires for delivery
- **Partial failures** are the cases where only some components of a distributed system fail
- **Concurrency** is the execution of multiple computations at the same time, and potentially on the same piece of data.
#### Correctness
- **Safety property**: Defines something that can never happen in the system (Ex: Over cannot go beyond max temperature)
- **Liveness property**: Defines something that must eventually happen in the system (Ex: Oven needs to reach the desired temperature)
#### Synchrony assumptions (Timing)
- Synchronous
	- Message latency no greater than the upper bound
	- Nodes execute algorithm at a given speed
- Partially synchronous
	- Synchronous for some time and not for some time
- Asynchronous 
	- Messages can be delayed arbitrarily
	- Node can pause execution
	- No timing guarantees 
#### Node behavior
- Crash Stop (Fail Stop)
	- A node is faulty if it crashes
	- After crashing it never recovers
- Crash recovery (Fail Recovery)
	- A node might fail losing data in memory
	- It will be back up again recovering data from disk
- Byzantine (Fail Arbitrarily)
	- A node that deviates from its intended behavior
	- This might crash or cause malicious behavior
#### Avoiding multiple deliveries of messages
- Idempotent
	- Example: Adding a value to a set
	- Not Example: A counter
	- Strategy: Send a unique identifier with each request. Need to control both the client and server to check for duplicate identifiers
#### Delivery semantics
- Exactly-Once
- At-Most-Once
- At-Least-Once
- Difference between delivery and processing of messages
- Delivery happens at the hardware level vs processing can be done on the application level to give an illusion of semantics
#### Timeouts
- Mechanism to detect failure by introducing artificial bounds on delivery of messages in an asynchronous system
- Tradeoffs between small and large timeout
- Failure detectors can be used separately with the below properties
	- Completness
	- Accuracy
## Basic Concepts and Theorems
#### Scalability
- Achieved through partitioning
- Partition types:
	- Vertical partitioning
	- Horizontal partitioning
		- Range partitioning
		- Hash Partitioning
		- Consistent Hashing
#### Replication
- Consists of storing the same piece of data across multiple nodes 
- Pessimistic replication
	- Gives the illusion that there is only one node with all the data i.e. all replicas are in sync
- Optimistic replication
	- Replicas may be out of sync but will be eventually in sync if there are no updates for time being
- Leader-Follower replication
	- Failover in case of leader failover
		- Manual
		- Automated
- Multi-Leader replication
	- All nodes can accept writes. This can lead to conflicts
	- Conflict resolution strategies:
		- Exposing to clients
		- Last-write wins
		- Causality tracking
#### Safety guarantees in distributed systems
- Atomicity
- Consistency
- Isolation
#### ACID Properties
- Atomicity
- Consistency
- Isolation
- Durability
#### CAP Theorem
- Consistency means that every successful read request receives the result of the most recent write request.
- Availability means that every request receives a non-error response, without any guarantees on whether it reflects the most recent write request.
- Partition tolerance means that the system can continue to operate despite an arbitrary number of messages being dropped by the network between nodes due to a **network partition**.
- In a system there can always be network partitions so we can either have CP or AP
####  PACELC theorem
- In the case of a _network partition_ (P), the system has to choose between _availability_ (A) and _consistency_ (C) but _else_ (E), when the system operates normally in the absence of network partitions, the system has to choose between _latency_ (L) and _consistency_ (C).
-  AP/EL
- CP/EL
- AP/EC
- CP/EC
#### Consistency Models
- Linearizability
	- All reads after a write will get the latest value
- Sequential consistency
	- If there are 3 events, A, B on client 1 and C on client 2, then all clients will see the ordering of events as A, B, C or C A B
- Casual consistency
	- If A leads to B on client 1 and C executes on client 2, then each client can see A, B, C or C A B (Since A leads to B, they are consistent)
- Eventual consistency
	- The system might not return the latest data but will eventually lead to correct values
#### Categorization of consistency models
- Too many consistency models causes confusion
- Broad categories are:
	- Strong consistency models - Compromises on availability (liearizability)
	- Weak consistency models - Preserves availability by compromising consistency
#### Anomalies(Important)

| Name                         | Description                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dirty Writes                 | A **dirty write** occurs when a transaction overwrites a value that was previously written by another transaction that is still in-flight and has not been committed yet.<br>![[Pasted image 20250120110131.png]]<br><br>                                                                                                                               |
| Dirty Reads                  | A **dirty read** occurs when a transaction reads a value that has been written by another transaction that has not yet been committed.<br>![[Pasted image 20250120110258.png]]                                                                                                                                                                          |
| (Fuzzy) Non-Repeatable reads | A **fuzzy or non-repeatable read** occurs when a value is retrieved twice during a transaction (without it being updated in the same transaction), and the value is different.<br>![[Pasted image 20250120110433.png]]                                                                                                                                  |
| Phantom reads                | A **phantom read** occurs when a transaction does a predicate-based read, and another transaction writes or removes a data item matched by that predicate while the first transaction is still in flight. If that happens, then the first transaction might be acting again on stale data or inconsistent data.<br>![[Pasted image 20250120110545.png]] |
| Lost updates                 | A **lost update** occurs when two transactions read the same value and then try to update it to two different values. The end result is that one of the two updates survives, but the process executing the other update is not informed that its update did not take effect.<br>![[Pasted image 20250120110705.png]]                                   |
| Read skew                    | A **read skew** occurs when there are integrity constraints between two data items that seem to be violated because a transaction can only see partial results of another transaction.<br>![[Pasted image 20250120110820.png]]                                                                                                                          |
| Write skew                   | A **write skew** occurs when two transactions read the same data, but then modify disjoint sets of data.<br>![[Pasted image 20250120110952.png]]                                                                                                                                                                                                        |
#### Isolation levels (Important)

| Name               | Description                                                                                                                                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Serializability    | It essentially states that two transactions, when executed concurrently, should give the same result as though executed sequentially.                                                                                                            |
| Repeatable Read    | It ensures that the data once read by a transaction will not change throughout its course.                                                                                                                                                       |
| Snapshot Isolation | It guarantees that all reads made in a transaction will see a consistent snapshot of the database from the point it started, and the transaction will successfully commit if no other transaction has updated the same data since that snapshot. |
| Read Committed     | It does not allow transactions to read data that has not yet been committed by another transaction.                                                                                                                                              |
| Read Uncommitted   | It is the lowest isolation level and allows the transaction to read uncommitted data by other transactions.                                                                                                                                      |
#### Isolation levels and prevented anomalies
![[Pasted image 20250120111643.png]]
#### Consistency and Isolation

###### Similarities
- Both models offer strict guarantees
###### Differences
- Consistency models can be applied to single-object operations while isolation models affect multiple rows
- Consistency models deal with the concept of time especially sequence of operations
###### Strict serializability
- Combination of linearizability and serializability
## Distributed Transactions
#### Achieving Atomicity
###### 2-Phase Commit (2PC)
- Consists of two roles:
	- The **coordinator** is responsible for coordinating the different phases of the protocol
	- The **participants** correspond to all the nodes that participate in the transaction
- The _coordinator_ and the _participants_ make use of a **write-ahead-log**, where they persist their decisions during the various steps so that they can recover in case of a failure.
- The coordinator also uses a **timeout** when waiting for the responses from the first phase. If the timeout expires, the coordinator interprets this timeout as a No vote and considers the node as failed.
- On the other hand, the _participants_ do not apply any timeouts while waiting for the coordinator’s messages, since that could lead to participants reaching different conclusions due to timing issues.
- Handling failures:
	- Failure of participant in the voting phase
	- Failure of participant in the commit phase
	- Network failures
- Blocking nature of 2PC
- Usage of 2PC:
	- eXtended Architecture (XA)
	- Participant must implement the interface of the resource manager
	- Coordinator must implement the interface of the transaction manager
- Safety property is achieved. Liveness might be compromised if coordinator fails
###### 3-Phase Commit (3PC)
- Splitting the first round (voting phase) into 2 sub-rounds, where the coordinator first communicates the votes result to the nodes, waits for an acknowledgment, and then proceeds with the commit or abort message.
- Benefits:
	- Coordinator is not the single point of failure
	- A participant taking over can commit the transaction if it receives a **prepare-to-commit**, knowing that all the participants have voted “Yes”. If it does not receive a prepare-to-commit, it can abort the transaction, knowing that no participant has committed, without all the participants receiving a prepare-to-commit message first.
- Network failures can make the system violate safety property to achieve liveness property
###### Quorum-based commit protocol
- This protocol leverages the concept of a _quorum_ to ensure that different sides of a partition do not arrive at conflicting results.
- The protocol establishes the concept of a **commit quorum (VC)** and an **abort quorum (VA)**.
- Requirement: VA + VC > V
- Sub-protocols in quorum-based commit protocol used in different cases:
	- The **commit** protocol, which is used when a new transaction starts
	- The **termination** protocol, which is used when there is a network partition
	- The **merge** protocol, which is used when the system recovers from a network partition
###### Commit Protocol
- Very similar to the 3PC protocol
- The only difference is that the coordinator waits for VC​ number of acknowledgments at the end of the third phase to proceed with committing the transaction.
###### Termination Protocol
- A (surrogate) coordinator will be selected from amongst the _participants_ with a leader election
- If there is at least one participant that has committed (or aborted), the coordinator commits (or aborts) the transaction, maintaining the _atomicity_ property.
- If there is at least one participant with prepare-to-commit state, and VC participants waiting for the results, the coordinator sends prepare-to-commit message
- If there is no participant with prepare-to-commit state, and VA participants waiting for the results, the coordinator sends prepare-to-abort message
###### Merge protocol
- Satisfies the safety property but not the liveliness property completely (Better than 3PC)
![[Pasted image 20250126132835.png]]
###### Saga
- Associated with transactions that are long lived such as an e-commerce order
- Isolation might not be required for these long lived transactions
- The _saga_ is a sequence of transactions T1​, T2, …, TN​ that can be interleaved with other transactions with atomicity$
- Each transaction Ti is associated with compensation transaction (Ci) for rollbacks
###### Benefits![[Pasted image 20250126133629.png]]
###### Cases where isolation is required
- Example: E-commerce item where you don't want an inventory of 1 to be distributed to 2 consumers. One consumer might not get the item
###### Providing isolation at the application layer
- Semantic lock
- Commutative updates
- Reordering the structure of saga
	- Re-order the saga structure so that a transaction called a **pivot transaction** delineates a boundary between transactions that can fail and those that can’t.
	- This transaction could have serious consequences if another concurrent _saga_ reads this increase in the balance, but then the previous transaction is rolled back. Moving this transaction after the _pivot transaction_ means that it will never be rolled back, since only all the transactions after the _pivot transaction_ can succeed.
## Consensus
#### Formal definition
- Assume we have a distributed system that consists of k nodes (n1, n2, …, nk​), where each one can propose a different value vi. _Consensus_ is the problem of making all these nodes agree on a single value.
- The following properties must also be satisfied:
	- **Termination**: Every non-faulty node must eventually decide.
	- **Agreement**: The final decision of every non-faulty node must be identical.
	- **Validity**: The agreed value must have been proposed by one of the nodes.
#### Use cases of consensus
- Leader election
- Distributed locking
- Atomic broadcast
#### FLP (Fischer, Lynch, Paterson)
- It is **impossible to achieve deterministic consensus in an asynchronous distributed system if even a single node can fail**.
#### Paxos Algorithm
- This algorithm guarantees that the system will come to an agreement on a single value and tolerate the failure of any number of nodes (potentially all of them) as long as more than half the nodes work properly at any time.
- Roles:
	- Proposer: A _proposer_ is responsible for proposing values (potentially received from clients’ requests) to the _acceptors_ and trying to persuade them to accept their value to arrive at a common decision.
	- Acceptor: An _acceptor_ is responsible for receiving these proposals and replying with their decision on whether this value can be chosen or not.
	- Learners: The _learners_ are responsible for learning the outcome of the consensus, storing it (in a replicated way) and potentially acting on it, by either notifying clients about the result or performing actions.
- Phases:
	- 1(a): Prepare request sent to acceptors with a unique proposal number (e.g., `cat(i++, node_number)` to generate unique numbers).
	- 1(b): Acceptor returns either a success message with the highest value it has seen before the current value OR returns a failure message with the value higher than the current value to give a hint to the proposer
	- 2(a): Proposer sends accept requests with the chosen value (or the highest proposal value returned in Phase 1).
	- 2(b): Acceptors confirm acceptance of the proposal.
	- Send the updates to learners about the chosen value
###### Dueling proposes
![[Pasted image 20250126145004.png]]
- Consensus is never achieved as two nodes are fighting for leader election
- Solution: Before proposing a new value after a rejection, have exponential backoff
###### Paxos handling Partial Failures
![[Pasted image 20250126145418.png]]
###### Towards running multiple instances of Paxos
- A single paxos instance can decide on a single value which has limited applications
- Consistent selection of values is needed we need to be running multiple instances of Paxos running independently and in parallel but have to be numbered
###### Paxos returning current state of the system (?)
- Clients might need to get past values
- Typically done by querying acceptors or learners without affecting their state
###### Read Operation
- Returns the decision of the previously completed instances alongside write operations that start new instances of the protocol by sending the request to the leader
- The leader cannot use the local copy as a new proposal might have started.
- Performs a read from majority of nodes.
###### Leader leases
- A node can take a lease by running the Paxos instance and no other node can challenge it
- Read operations can go to this node that occur during the lease duration
- Clock skew must be taken into account
#### Multi-Paxos
- Under stable conditions:
	- After a successful proposal i.e. distinguished proposer, leader skips Phase 1 and directly executes Phase 2 using the value from the last proposal
	- This increases efficiency as it does not have to execute Phase 1
- Under failure conditions:
	- Nodes detect leader failure using heartbeat and start a new election i.e. proposal
	- Uses the regular Paxos protocol for the same
#### Difference between Distributed Transactions and Consensus Problem

| Consensus                                    | Distributed Transactions                                  |
| -------------------------------------------- | --------------------------------------------------------- |
| Every non-faulty node reaches the same value | Every node including faulty reaches the same value        |
| Less strict vote requirements such as quorum | Strict vote requirements such as all nodes should say yes |
| Value will be proposed by one node           | Value should be proposed by all nodes (commit/abort)      |
#### Time

###### Atomic Clock
- An _atomic clock_ is one of the most accurate timekeeping devices. It uses the frequency of electronic transitions in certain atoms to measure time.
#### Total and Partial Ordering
###### Total Ordering
- Given a set of events, it will provide a single order for all the elements in the set.
###### Partial Ordering
- Binary relation to compare only some of the elements of a set with each other.
###### Total ordering events in a single-node
- It’s easy and intuitive to determine a _total ordering_ of events. The main reason is that there is a single actor where all the events happen, so this actor can impose a _total order_ on these events as they occur.
- For any two events (e1​, e2​), one starts after the other finishes.
- Clock errors don’t matter since there is only one clock.
###### Partial Ordering events in distributed systems
- Each node assigns unique timestamps using local clocks.
- Clocks run at different rates and have errors, complicating comparisons of timestamps.
- Clock errors across nodes must be accounted for, making total ordering harder.
#### The Concept of Causality
Refer Martin Klepmann Lecture Notes
#### Lamport Clocks
Refer Martin Klepmann Lecture Notes
#### Vector Clocks
Refer Martin Klepmann Lecture Notes
#### Version Vectors
- Version vectors are a data structure used in distributed systems to track updates to replicated data. 
- They help in detecting conflicts and maintaining causality across replicas.
- `C1`, `C2`, ..., `CN` are the counters for each node.
- Each counter represents the number of updates made by the respective node.
###### Operations on Version Vectors
- Increment:
	 - A node increments its counter whenever it performs a local update.
	```
   Before: {Node1: 1, Node2: 0}
   After Node1 Update: {Node1: 2, Node2: 0}
   ```
- Merge:
	 - Combine two version vectors by taking the element-wise maximum:
   ```
   A = {Node1: 1, Node2: 0}
   B = {Node1: 0, Node2: 2}
   Merged: {Node1: 1, Node2: 2}
   ```
-  Comparison:
	 - To compare two version vectors `A` and `B`:
		   - `A == B`: All counters in `A` are equal to those in `B`.
		   - `A <= B`: All counters in `A` are less than or equal to those in `B`.
		   - `A || B` (Concurrent): Some counters in `A` are less, and some are greater.
###### Handling Conflicts
- Last-Write-Wins (LWW):
	 - Choose the update with the most recent timestamp or predefined rule.
- Merge Updates:
	 - Combine the updates logically.
	 - Example: Combine items in a shopping cart.
- Present to Application:
	 - Allow the application to resolve conflicts manually.
###### Advantages
- Tracks causality across replicas.
- Lightweight compared to other techniques (e.g., full logs).
- Ensures eventual consistency.
###### Limitations
- Fixed size: Requires one counter per node.
- Difficult to manage with dynamic node membership (e.g., adding/removing nodes).


