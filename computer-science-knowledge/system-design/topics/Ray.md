- **Ray** is a distributed computing framework that makes it easy to scale Python applications from a single machine to a cluster.
- **Ray Data** is a high-level library for distributed data processing and loading, focusing on ML/AI data workflows.
- Ray Data can be used standalone or integrated with other Ray libraries and ML frameworks.
#### Core Terms
- **Cluster**: A set of machines/nodes managed by Ray, where tasks are distributed.
- **Driver/Head Node**: The main process that coordinates tasks in Ray.
- **Worker Nodes**: Processes that execute tasks (Ray tasks or actor tasks).
- **Ray Dataset**: The fundamental data abstraction in Ray Data (distributed collection of data blocks).
#### Dataset and DatasetPipeline
- A **Ray Dataset** is partitioned into **blocks**, each of which can be processed in parallel.
- Operations on datasets (like `.map()`, `.filter()`) can be **lazy**; Ray builds a DAG (directed acyclic graph) of transformations.
- **DatasetPipeline** is used for streaming data in an iterative fashion, useful for training large models on large datasets that don’t fit into memory.
#### Blocks
- Blocks are the fundamental units of data parallelism in Ray Data.
- Each dataset is split into blocks that can be processed in parallel across the cluster.
#### Transformations, Operations and DAG's
- `map` transforms each row or record one at a time.
- `map_batches` processes a batch of records as a single block; can speed up performance for vectorized operations (e.g., NumPy, Pandas).
- DAG execution means Ray Data will schedule tasks across the cluster, but you can control when to execute via `.materialize()`, `.take()`, etc.
#### Operators and Plans

![[Pasted image 20250307071404.png]]

Ray uses a two phased approach:
- Writing a program using the DataSet API, Ray Data first builds a logical plan - a high level description of what operations to perform
- When execution begins, it converts into a physical plan that specifies how to execute these operations

Logical Plan: Consist of logical operations that describe what operations to perform
Physical Plan: Consist of physical operators describing how to execute the operation

When execution begins, Ray Data optimizes the logical plan, then translate it into a physical plan - a series of operations that implement the actual data transformation.

Physical operations work by:
	- Taking a stream of block references
	- Performing their operation
	- Outputting another stream of block references
#### Streaming Execution Model

![[Pasted image 20250307072422.png]]

Ray Data uses a streaming execution model to efficiently process large datasets. Instead of loading the whole dataset in memory, Ray Data can process data in a streaming fashion through a pipeline of operations. Operations are connected in a pipeline, with each operator output queue feeding directly into the input queue of the next downstream operator. 
#### Task Dependencies
![[Pasted image 20250307105413.png]]
![[Pasted image 20250307105428.png]]
![[Pasted image 20250307105443.png]]
- The second task y2_id will not be executed until the first task has completed executing
- If the two tasks are scheduled on different machines, the output of the first (the value corresponding to x2_id) will be copied over the network to the machine where the second task is scheduled.
#### Actors
- A remote function is side effect free i.e. they operate on inputs and produce outputs, but they don't change the state of the worker they execute on.
- Actors are different. When we instantiate an actor, a brand new worker is created, and all the methods that are called on the actor are executed on the same worker.
- This means that with a single actor, no parallelism can be achieved because calls to actor's methods will be executed one at a time. However multiple actors can be created and methods can be executed in parallel.



#### Questions:
- When creating a large dataset, how does Ray Data decide the number of blocks, and can you control it?
- What is the difference between Ray and Spark? What benefit does Ray offer that Spark does not?
- What factors influence how data is partitioned and how many blocks are created when reading a dataset?

#### How Apple Uses Ray?

Reference: https://www.youtube.com/watch?v=ZCRZQVt-r3g&t=1459s
#### Why Ray?
- Unified Computing API and Platform for Data Processing, Training, Tuning and Serving
- Scale from Laptop to Data Center
- Strong support of Model Serving including support for vLLM

![[Pasted image 20250309211116.png]]
#### Challenges
- Resource Sizing
- Auto Scaling
- GPU Utilization

![[Pasted image 20250309211455.png]]
![[Pasted image 20250309211638.png]]
![[Pasted image 20250309211748.png]]
![[Pasted image 20250309212129.png]]
![[Pasted image 20250309212247.png]]
- KubeRay creates the Ray cluster (head + worker nodes) but it does not decide where or when to run these resources.
- Yunikorn as a scheduler decides which nodes in the Kubernetes cluster should run the Ray head and worker pods
- It prioritizes jobs, ensuring fair resource allocation and optimizing cluster utilization.

![[Pasted image 20250309213031.png]]
![[Pasted image 20250309213145.png]]
![[Pasted image 20250309213244.png]]
![[Pasted image 20250309213336.png]]
![[Pasted image 20250309213435.png]]
![[Pasted image 20250309213534.png]]


Reference: https://www.youtube.com/watch?v=G8Di0d1_XDg&t=903s

![[Pasted image 20250309215629.png]]
![[Pasted image 20250309215834.png]]
![[Pasted image 20250309220037.png]]
![[Pasted image 20250309223206.png]]
![[Pasted image 20250309223303.png]]
![[Pasted image 20250309223415.png]]
![[Pasted image 20250309223502.png]]

Reference: https://www.youtube.com/watch?v=1vGb0nn5n0o

![[Pasted image 20250309223714.png]]
![[Pasted image 20250309223833.png]]
![[Pasted image 20250309224125.png]]
![[Pasted image 20250309224227.png]]
![[Pasted image 20250309224418.png]]
![[Pasted image 20250309225023.png]]
![[Pasted image 20250309225142.png]]

Reference: https://www.youtube.com/watch?v=9S5WznGnIpE
![[Pasted image 20250309222716.png]]
![[Pasted image 20250309222837.png]]

Reference:https://www.youtube.com/watch?v=oH9pJavu-PU&t=48s (Ray, a Unified Distributed Framework for the Modern AI Stack | Ion Stoica)

![[Pasted image 20250309225751.png]]
![[Pasted image 20250309225828.png]]
