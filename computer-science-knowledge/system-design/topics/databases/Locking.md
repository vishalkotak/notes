### Pessimistic Locking
- Concurrency control strategy used in database management systems
- Core Idea: Assumption that conflicts between concurrent transactions are likely to occur
#### Characteristics 
- Assume conflict will happen such as one transaction is writing data and other tries to read it
- Lock before access
- Exclusive Access (FOR UPDATE). No other transaction can read or write data
- Other transaction needs to wait until the first transaction is complete
- Prevents conflicts proactively
#### Advantages
- High Data Integrity
- Simpler Conflict Handling
- Effective in high contention
#### Disadvantages
- Reduced concurrency
- Potential for deadlocks
- Scalability issues
- Locks might be held for a long time
### Optimistic Locking
- Assumption that conflicts between concurrent transactions are relatively infrequent
- Instead of locking resources upfront to prevent conflicts, optimistic locking allows transactions to proceed without locks and only checks at the very end when a transaction tries to commit its changes
#### Characteristics 
- Assume no conflict
- Read without locking
- Perform work
- Validate at commit
	- Rechecks the version identifier of the data in the database
	- Compares the version identifier with the version identifier it recorded when it first read the data
- Retry on failure with the new version number
##### Versioning Implementation
- Version Numbers
- Timestamps
- State Comparison
#### Advantages
- Higher concurrency
- No deadlocks
- Better scalability 
#### Disadvantages
- Retry logic required if version numbers don't match
- Wasted work - If conflicts are frequent, transactions will often fail meaning the computational work performed is wasted
- Potential for starvation
- Complexity in Conflict Resolution
