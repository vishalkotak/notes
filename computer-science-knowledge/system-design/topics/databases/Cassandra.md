#### When to use Cassandra
- Big data
- Rapidly changing data
- Large number of writes
- Availability / Uptime important
- Horizontal Scaling important
- Geographical distribution
#### Characteristics

- **No master slave**, **leaderless**
- No separate config server
- Data is replicated across nodes
- Different Nodes, Data Centers, Geographies
- **Eventual Consistent**
- CAP Tradeoff
	- **High availability over consistency**
- **Hash Ring: Data is partitioned around a ring**
- Determined by partition key
- **Cassandra performs an upsert instead of insert**

![[Pasted image 20250411153423.png]]

#### Cassandra Tables
- Columnar tables
- Rows represent things of a type
- Typically narrow (i.e. not too many columns)
- Joins not possible
- Denormalizing is okay
- No foreign key constraints
- Not all rows will be in the same partition
#### Terms
- **Keyspaces**:  Keyspaces are basically data containers, and can be likened to "databases" in relational systems like Postgres or MySQL. They contain many tables.
- **Table** - A table is container for your data, in the form of rows.
- **Row** - A row is a container for data.
- **Column** - A column contains data belonging to a row.
- **Partition Key** - One or more columns that are used to determine what partition the row is in.
- **Clustering Key** - Zero or more columns that are used to determine the sorted order of rows in a table.
#### Partition Key
- Used to split tables
- Each partition gets mapped to a node
- Chose a partition key wisely
- Consistent hashing is used with the key to distribute the load
- What is a good partition key?
	- Uniquely identifies the data
	- Spreads data evenly across partitions
	- Limit multiple node access for common fetches
	- Not the primary key
#### Primary Key
- Uniquely identifies the row
- Can be one or more columns
- If you have one column as the primary key, then it becomes the partition key
#### Clustering column example
- Similar to sort key in DynamoDB
- Difference is you have multiple keys in Clustering column
#### Replication

Cassandra has two different "replication strategies" it can employ: [NetworkTopologyStrategy](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html#network-topology-strategy) and [SimpleStrategy](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html#simple-strategy).

- **NetworkTopologyStrategy** is the strategy recommended for production and is data center / rack aware so that data replicas are stored across potentially many data centers in case of an outage. It also allows for replicas to be stored on distinct racks in case a rack in a data center goes down. The main goal with this configuration is to establish enough physical separate of replicas to avoid many replicas being affected by a real world outage / incident.
- **SimpleStrategy** is a simpler strategy, merely determining replicas via scanning clockwise (this is the one we discussed above). It is useful for simple deployments and testing.

Below is Cassandra CQL for specifying different replication strategy configurations for a keyspace:

```
---3 replicas
ALTER KEYSPACE hello_interview WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };

-- 3 replicas in data center 1, 2 replicas in data center 2
ALTER KEYSPACE hello_interview WITH REPLICATION = {'class' : 'NetworkTopologyStrategy', 'dc1' : 3, 'dc2' : 2};
```
#### Partition Rings
- Data distributed across nodes by partition key
- Partition key is run through a hash function
- Nodes are responsible for a range of hash values
- Nodes can be added or removed at any time
- After changes, nodes re-adjust and distribute the range of hash values
#### Understanding consistency levels and quorum

![[Pasted image 20250329104620.png]]

![[Pasted image 20250329104606.png]]

![[Pasted image 20250329121631.png]]
#### Storage Model
**Commit Log** - This basically is a [write-ahead-log](https://en.wikipedia.org/wiki/Write-ahead_logging) to ensure durability of writes for Cassandra nodes.
 **Memtable** - An in-memory, sorted data structure that stores write data. It is sorted by primary key of each row.
 **SSTable** - A.k.a. "Sorted String Table." Immutable file on disk containing data that was flushed from a previous Memtable.
#### Understanding Write Path

Write Path - What Happens (Immediately)
- Time stamp is attached
- Write is written to commit log
- Write is written to memtable (in memory)
- Completion of a write

![[Pasted image 20250329124422.png]]

What happens (background)
- Memtable is flushed to disk when full
- Written to a file called SSTable
- New MemTable is created in memory

SSTable is Immutable
- Updates new writes
- Deletes are new writes (tombstone)
- Indicates data doesn't exist anymore
- Tombstone writes come with timestamp too
#### Read Path

- Check MemTable and return
- Fetches data from (possibly) multiple SSTables - Uses bloom filter to check membership
- Reads SSTables in order from newest to oldest to find the last data of the row
- **Compaction**
	- Reduction of bloat on SSTables
	- Merges data based on timestamp, remove data that is marked with tombstone
- **SSTable Indexing**
- Read Repair: https://cassandra.apache.org/doc/stable/cassandra/operating/read_repair.html#read-repair-example
#### Gossip
- Nodes manage **generation** and **version numbers** for each node
- Generation is timestamp when the node was bootstrapped
- Version is logical value that increments every second
- Across these clusters these values form a vector clock
- Cassandra gossips with a probabilistic bias towards seed nodes
#### Seed Nodes
- Designated by Cassandra to bootstrap the cluster and serve as guaranteed hotspots for gossip so all nodes are communicating across the cluster
- Eliminates the possibility that sub-clusters of nodes emerge because information happens to not reach the entire cluster
- Always discoverable via off the shelf service discovery mechanisms
#### Fault Tolerance
- Phi Accural Failure Detector
- Node gossip does not respond, Cassandra failure logic will stop sending writes to that node
- Will never consider a node truly down unless the Cassandra System Administrator decommissions the node 
#### Hinted HandOfff
- Coordinator attempting to write a record to offline node, temporarily stores the data in order for the write to proceed
- Temporary data is called hint
- Is useful for short downtime. If the downtime is high, node needs to be rebuilt or undergo read repairs as hints have less lifespan

![[Pasted image 20250329125413.png]]
#### Advanced features
- Storage Attached Indexes (SAI) 
	- Global secondary indexes on columns
	- Search performance poor as compared to using the partition key but better than scanning across all nodes
	- Can be used to reduce denormalization for less frequent queries
- Materialized Views
	- Similar to SQL
- Search Indexing
	- Integration with Elastic Search
#### Examples
##### Discord chat messages
Discord’s use of **Cassandra** for storing message data is a great example of **query-driven schema design**. Since users typically access **recent messages** in a **reverse chronological order**, Discord initially used a schema with `channel_id` as the **partition key** and `message_id` (a **Snowflake ID**, not a timestamp) as the **clustering key**. This allowed efficient reads within a single partition.

However, **high-volume channels** led to **large partitions**, which degraded Cassandra's performance. To address this, Discord introduced a **bucket** (representing 10-day intervals based on a custom epoch) as part of the **partition key**, creating a new schema:

`PRIMARY KEY ((channel_id, bucket), message_id)`

This limited partition size, avoided monotonically growing partitions, and aligned with typical access patterns. The approach showcases how understanding **access patterns, partition sizing**, and **Cassandra internals** leads to scalable schema design.
##### TicketMaster
Ticketmaster's **ticket browsing UI** prioritizes availability and performance over strict consistency, making it a good fit for **Cassandra**. Users often browse seats for an event but only a few proceed to checkout, so the system must quickly respond to many read-heavy queries.

```
CREATE TABLE tickets (
  event_id bigint,
  seat_id bigint,
  price bigint,
  PRIMARY KEY (event_id, seat_id)
);
```

**Problem**: Large partitions for events with thousands of seats cause performance issues during queries like ticket counts or price totals.

```
CREATE TABLE tickets (
  event_id bigint,
  section_id bigint,
  seat_id bigint,
  price bigint,
  PRIMARY KEY ((event_id, section_id), seat_id)
);
```

- Adds `section_id` to the **partition key**, aligning with the UI where users browse by section.
- This reduces partition size and spreads load across nodes.

**How do we now show ticket data for the entire event?**

```
CREATE TABLE event_sections (
  event_id bigint,
  section_id bigint,
  num_tickets bigint,
  price_floor bigint,
  PRIMARY KEY (event_id, section_id)
);
```

- Supports UI needs like showing **aggregated ticket availability per section**.
- Prefers **denormalization** over costly aggregation.
- Trades off **strict consistency** for **efficient access**, acceptable due to UI tolerances (e.g., showing “100+ tickets”).

#### Commands

```
cqlsh> describe cluster

Cluster: Test Cluster
Partitioner: Murmur3Partitioner
Snitch: DynamicEndpointSnitch
```
###### Create Table
```
-- Primary key with partition key a, no clustering keys CREATE TABLE t (a text, b text, c text, PRIMARY KEY (a));

-- Primary key with composite partition key a + b, clustering key c CREATE TABLE t (a text, b text, c text, d text, PRIMARY KEY ((a, b), c));

-- Primary key with partition key a, clustering keys b + c CREATE TABLE t (a text, b text, c text, d text, PRIMARY KEY ((a), b, c));

-- Primary key with partition key a, clustering keys b + c (alternative syntax) CREATE TABLE t (a text, b text, c text, d text, PRIMARY KEY (a, b, c));
```
###### Select query based on Primary Key

```
SELECT * from person WHERE id='1';

# Result
```
###### Select query based on column

```
SELECT * from person WHERE name = 'Vishal';

**InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"**
```
###### Inserting record

```
INSERT INTO person (first_name, last_name, id) VALUES ('Anku', 'Kotak', '2');

# Result
```
###### Inserting record based on Primary Key that already exists

```
INSERT INTO person (first_name, last_name, id) VALUES ('Hinal', 'Kotak', '2');

# It will overwrite the above inserted record
```
