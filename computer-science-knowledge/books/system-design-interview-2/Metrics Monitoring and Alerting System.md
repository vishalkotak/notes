# High-level requirements and assumptions:

## Functional
- 100 million active users
- 1000 server pools * 100 machines per pool * 100 metrics per machine = 10 million metrics
- 1 year data retention
## Non-Functional
- Scalability
- Reliability
- Low Latency
- Flexibility
# High-level design

## Fundamentals

### Components
- Data Collection
- Data transmission
- Querying
- Alerting
- Visualization

![[Pasted image 20231021152442.png]]
### Data Model
#### Example 1

| Name  | Value  |
|----------|----------|
| metric_name | cpu.load |
| labels | host:631, env: prod |
| timestamp | 1234567890 |
| value | 0.29 |
#### Example 2
Example of line protocol: CPU.load host=webserver01, region=us-west 1234567890 50. This is followed by Prometheus and OpenTSDB

| Name | Type |
| ---| --- |
| A metric name | String |
| A set of tags/labels | List of <key:value> pairs |
| An array of values and their timestamps | An array of <value, timestamp> pairs |
### Data Access Patterns
- Write heavy as telemetry is being appended
- Read traffic can be bursty based on alerting and querying
### Data Storage Systems
#### Option 1 - SQL
- SQL tables are not optimized for time-series data. Example: Computing the moving average in a rolling time window requires complicated querying
- We need support for labels/tags and querying might require index on each tag
- SQL tables don't perform well under heavy write loads. Why??
- Significant tuning would be required
#### Option 2 - NoSQL
- Cassandra and BigTable can be used for time-series data but that would require deep expertise of NoSQL to devise a scalable schema for effectively querying and storing time-series data
#### Option 3 - Time Series DB (Example: InfluxDB, Open TSDB, Prometheus)
- Optimized for time-series data
- Huge volumes of data with lesser servers
- Custom querying much simpler than SQL
- Support for data retention and aggregation

- Open TSDB - Requires Hadoop cluster maintenance - Not recommended
- InfluxDB and Prometheus - In-memory caching and On-Disk storage, Handles performance quite well

Additionally, InfluxDB builds indexes on labels to facilitate the fast lookup of time-series data by labels. Recommendations on labels includes low-cardinality (having a small set of possible values)
# Design Deep Dive

## Metrics Collection

### Pull Model
#### Steps:
- Metric collector fetches configuration metadata of service endpoints from Service Discovery. Metadata include pulling interval, IP addresses, timeout and retry parameters.
- The metric collector pulls metrics data via a pre defined HTTP endpoint (for example: /metrics). To expose the endpoint, a client library usually needs to be added to the service. 
- The metric collector registers a change event notification with Service Discovery to receive an update whenever the service endpoints change. 
#### Cons:
- Single metric collector might not be sufficient. We need an array of instances
- Multiple collectors can bring an issue of trying to pull data from same server causing duplication
- To avoid this issue, we can use consistent hashing ???
- Creating a TCP connection from the metric collector to the server instance might require changes to firewall.
### Push Model
A collection agent also known as long-running software that collects metrics from the services running on the server and pushes metrics periodically to the metrics collector. Optionally, the servers can first store data locally before pushing it out but this can lead to data loss if the server crashes before pushing metrics.

A load balancer can be added before the metrics collector instances to manage load. Major con of the push model is a requirement for some kind of authentication for the server looking to push metrics.
## Metrics Transmission
When moving data from the metrics collector to the time-series database, there is a chance of data loss if the database is unavailable. To mitigate this problem, we can have a queuing service such as Kafka between the metrics collector and the database.
### Pros:
- Highly reliable and scalable
- Decoupling
- Configure the partition based on metrics
- Priority to metrics that are important
### Cons
- Managing Kafka is hard
- Facebook's Gorilla is highly available with node failures accounted for
## Querying Service
Comprises of a cluster of query servers, which access the time-series database and handles requests from the visualization or alerting system.
### Pros
- Decoupling to allow for changes to storage layer
- Caching can be provided on this layer
### Cons
- Introduces additional abstraction
- Many databases offer plugins for the same
## Storage layer

### Choose a time-series db carefully
### Space optimization 
- Compression and encoding
- Downsampling
- Cold Storage
## Alerting System
- Config files are stored on a cache server
- Alert manager fetches the config files from the cache server
- Alert manager calls the querying service at pre-defined intervals
- Alerts are stored in a key-value store with their state
- Eligible alerts are inserted into Kafka and sent at least once
- Alert consumers pull data from Kafka
- Alert consumers can decide how alerting needs to be done and sent across appropriate mechanisms such as pager, email, etc.
## Visualization System
- Example: Grafana


