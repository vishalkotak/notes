**Indexing**

**Build a searchable index**

The simplest way to build a searchable index is to a assign a unique id to each document and store it in a database table.

![[Pasted image 20231123075047.png]]
On each search request, we will have to traverse all the documents and count the occurrence of the search string in each document.

**Inverted Index**

An inverted index is a HashMap like data structure that employs a document term matrix. The document term matrix identifies all the unique words and performs deletions on frequently occurring words such as "to", "they".

![[Pasted image 20231123075529.png]]
- A lit of documents in which each term appears
- A list that counts the frequency with which the term appears in each document
- A two dimensional list that pinpoints the position of the term in each document. 

Advantages:
- Full-text searches
- Faster as the mapping is already available

Disadvantages:
- Storage overhead
- Maintenance cost to add/delete/update the index

**Indexing on a centralized system**

![[Pasted image 20231123080254.png]]

Disadvantages:
- Single point of failure
- Server overload due to large number of queries
- Large size of the index

**Distributed Index Hight Level Design**

![[Pasted image 20231123080441.png]]

- Crawler - Collects content from the intended resource
- Indexer - Fetches the document from the distributed storage and indexes these documents using MapReduce
- Distributed Storage: Store documents and index
- User: Enters search query
- Searcher: Parses the search string that contains multiple words and returns the most matched results to the user

Partitioning
- Document Partitioning
- Term Partitioning

![[Pasted image 20231123081522.png]]

Indexing
- Documents collected by the crawler
- Cluster splits the documents into cluster of n machines
- Indexing algorithm is running on all machines to create a small inverted index on each machine's local storage

Searching
- Run parallel queries on each tiny inverted index
- Mapping lists are merged by the merger generally using sort ranking based on frequency

**Replication Factor and Replica Distribution**

Three replicas of the same partition will be sent to three different nodes. Nodes will compute index of the same partition three times. Load balancer can chose one of the three copies of each partition.

**Problems associated with the proposed design**

- Colocating indexing and searching: There can be a imbalance of indexing and searching on the same node
- Index recompilation: We will run indexing across three different replicas of the same partition which seems expensive

We can have a node that acts as a primary replica and performs indexing. Once indexing is complete, it can share it with the other nodes that contain the same replica.

**Separating the indexing and searching**

![[Pasted image 20231123083130.png]]

**Indexing using Map Reduce**

![[Pasted image 20231123083432.png]]


