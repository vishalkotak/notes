# Google File System (GFS)

### Problems with traditional file systems
- A **single-node file system** is a system that runs on a single computer and manages the storage attached to it. A single server has limited resources like storage space, processing power, as well as I/O operations that can be performed on a storage disk per second.
- The **network-attached storage** **(NAS) system** consists of a file-level storage server with multiple clients connected to it over a network running the network file system (NFS) protocol. This system can also suffer from throughput issues while accessing large files from a single server.
- The **storage area network (SAN) system** consists of a cluster of commodity storage devices connected to each other, providing block-level data storage to the clients over the network. However, these systems are difficult to manage because of the complexity of the second network — the Fiber Channel (FC). To set up the Fiber Channel, we need dedicated host bus adapters (HBAs) to be deployed on each host server, switches, and specialized cabling.
- ![[Pasted image 20250202112831.png]]
### GFS Requirements
###### Functional Requirements:
- Data Storage
- Data Retrieval
###### Non-Functional Requirements
- Scalability
- Availability
- Fault Tolerance
- Durability
- Easy operational management
- Performance optimization (increased throughput)
- Relaxed consistency (no strong consistency)
### Architecture
- A GFS cluster consists of two major types of components– a manager node and a large number of chunkservers.
- Each file is split into fixed-size chunks.
- The manager assigns each chunk a 64-bit globally unique ID and assigns chunkservers where the chunk is stored and replicated.
- A **manager** is like an administrator that manages the file system metadata, including namespaces, file-to-chunk mapping, and chunk location.
- Chunk size is 64 MB
### Design Pattern
- Each machine runs a Linux file system to manage the storage attached to it
- Each chunk is 64 MB in size and is identified with a unique ID called a **chunk handle**.
- GFS stores three replicas for each chunk by default for reliability.
- All this communication between the manager and the chunkservers is done through **heartbeat** messages.
- GFS uses a monitoring system that runs outside the cluster to monitor the machines inside the cluster
- The metadata size per file and chunk was very small; less than 64 bytes are required to store a chunk's metadata and the same for the file name and other file-specific information.
- The chunk metadata includes chunk ID, chunk's current version, and chunk's replica locations while the file metadata includes the file name (stored in a prefix-compressed form), file ownership and permission information, and file-to-chunk mapping.
- The manager keeps itself up to date by asking the chunkservers about the chunks they hold. This is done via regular heartbeat messages between the manager and the chunkservers.
### Create a file
The manager acquires two types of locks to guarantee the atomicity and correctness of file creation operation.
- A **read lock** is acquired on the directory name (full path to the directory) to make sure that the directory in which the file is being created is not deleted or renamed by another client during the file creation operation.
- A **write lock** is acquired on the file name (full path to the file) to serialize the creation of two or more files with the same name.
- We can acquire a read lock on a region that is already read-locked but we can't acquire a write lock on a region that is already write-locked. Operations that just need read lock can run concurrently.

In operating systems, there are four necessary conditions if a deadlock happens:

- **Bounded resources:** Only a finite number of client requests can access a resource concurrently.
- **No preemption:** Once a lock is acquired, its ownership can only be changed by the thread that acquired the lock.
- **Waiting while holding locks:** When a client needs multiple locks, it acquires one, then waits for the next ones to be acquired by keeping the previously locked resources.
- **Circular waiting:** There is a circular wait between different threads.

GFS primarily targets circular waiting. With its well-defined locking order, it never lets a circular waiting happen and avoids deadlocks.
### Calculations
![[Pasted image 20250202121904.png]]![[Pasted image 20250202121927.png]]
### Writing data to the file
- n a **random write operation**, the client provides the offset at which the data should be written. 
- In an **append operation**, the data is written at the end of the file at the GFS chosen offset.
- During the write operation, the manager gives one of the replica on the lease. This acts as the primary node for the write and others are secondary.
- We need a primary because if we use multi-leader approach to accept writes, then it would require us to provide consensus because writes might arrive in different order across replicas.
### Edge cases when writing data
- If the file has a chunk that is less than 64 MB, the manager will return the chunk handle, chunk servers. The client will send data to the primary replica and append the data to the last chunk.
- If the file's last chunk is already 64 MB, it will inform the client that the chunk is full. The client can send a request to the manager to create a new chunk. After receiving the details, the client can send data for the new chunk.
- If the file's last chunk has partial space available, the chunk server might write data that can be accommodated to the chunk after informing the client to split the data and send partial chunk. Also, the client needs to get the manager to create a new chunk and store second part of data on the new chunk.
### Delete a file
- The file system implements a service called **garbage collection**. This service deletes the chunks but responds to the client immediately after marking the deleted file in the namespace tree maintained at the manager node.
- The chunkservers share the chunk handles for the chunks that they hold with the manager through heartbeat messages regularly. If the manager doesn't find a chunk handle in the metadata, it informs the chunkserver about it, and the chunkservers delete such chunks.
### Snapshot file/directory
- A **snapshot** operation creates a copy of a file or a directory tree almost immediately and at a low cost.
- Consider if someone performed a write operation on a file while a snapshot was also being taken then the snapshot will result in an incorrect copy of the data. The concurrent snapshot operations might result in inconsistent data among the copies of the same file. We need to stop all the write operations for the file/directory to perform the snapshot operation.
- GFS used **Copy-On-Write** procedure where if one chunk handle have two references due to snapshots, the write operation on the chunk might cause the snapshot to see the change too. In order to avoid this, before writing data to the chunk, we change the chunk handle, replicate the data across the chunk servers and then execute the original write operation.
### States of a file region after data mutation
- **Consistent**: A file region is consistent if a client sees the same data on all replicas after a mutation.
- **Inconsistent**: A file region is inconsistent if a client sees different data on multiple replicas.
- **Defined**: After mutations to a file region (possibly concurrent), if the region has properly changed data such that applications can parse and read it as per application-level record format, that region is **defined**.
- **Undefined**: After mutations are made to a file region (possibly concurrent), if the region hasn’t properly changed data such that applications can parse and read it as per application-level record format, that region is **undefined**.
### Consistency assurance by GFS
- **Serial success**: The mutations were carried out in a sequence, one after the other, and successfully completed on all replicas.
- **Concurrent success**: The mutations were carried out concurrently (more than one mutation is being executed at the same time on one specific region) and successfully completed on all replicas.
- **Failure** means that the mutation failed to execute either on the primary replica or the secondary replicas, even after retries. If it fails to execute at the primary and is not therefore forwarded to the secondary replicas, then the file region will remain consistent. If the mutation has succeeded at the primary replica but failed at any of the secondary replicas, it will leave the file region inconsistent.
- ![[Pasted image 20250202131256.png]]
### Metadata
- Metadata server stores data in memory for performance. Some parts of it are stored on disk too for durability. The part that is not persistently stored can be rebuilt if needed. At times, such data is called soft state.
- Metadata such as namespaces and file-to-chunk mappings only occur at the manager level. We can't determine this metadata from anywhere else if it has been lost as a result of the failure and the manager node's restart.
- The location of each chunk's replicas doesn't need to be stored permanently on the manager node because the manager can determine this information from chunkservers through heartbeat messages.
- For metadata, the GFS provides a strong consistency model so that the system works normally in case of a single manager's failure.