- The structure of a CUDA C program reflects the coexistence of a _host_ (CPU) and one or more _devices_ (GPUs) in the computer. Each CUDA C source file can have a mixture of host code and device code. By default, any traditional C program is a CUDA program that contains only host code. The device code includes functions, or _kernels_, whose code is executed in a data-parallel manner.
- When a kernel function is called, a large number of threads are _launched_ on a device to execute the kernel. All the threads that are launched by a kernel call are collectively called a grid. When all threads of a grid have completed their execution, the grid terminates, and the execution continues on the host until another grid is launched.
#### Threads
 - In CUDA, the execution of each thread is sequential as well. A CUDA program initiates parallel execution by calling kernel functions, which causes the underlying runtime mechanisms to launch a grid of threads that process different parts of the data in parallel.

![[Pasted image 20250409222854.png]]
#### Device Global Memory and Data Transfer
- Devices are often hardware cards that come with their own dynamic random-access memory called device _global_ memory, or simply global memory

![[Pasted image 20250409223434.png]]

![[Pasted image 20250409225909.png]]

Reference:
https://www.udemy.com/course/cuda-parallel-programming-on-nvidia-gpus-hw-and-sw/learn/lecture/34013876#content
#### CUDA Programming
- Compute Unified Device Architecture
- Parallel Computing platform and application programming interface (API)
- GPGPU (General Purpose Computing on Graphic Processing Units)
- Based on the C Programming language
- CUDA compiler is called NVCC
##### Host and Device
![[Pasted image 20250413150349.png]]

![[Pasted image 20250414173721.png]]
#### ConnectX-7
- High performance Smart Network Interface Card (Smart NIC)
- Fast Network Connectivity (up to 400Gb/s) for servers
- Includes hardware accelerators to offload specific network related tasks (like RDMA/RoCE), GPU Direct, TLS/IPsec encryption/decryption from the main CPU improving efficiency and performance
#### BlueField-3
- Data Processing Unit (DPU)
- Integrates ConnectX-7 network controller as its network frontend
- Capabilities include:
	- Powerful ARM Processor Cores (16 ARM Cortex - A78 cores)
	- Significant Onboard Memory (DDR)
	- Dedicated accelerated engines for storage (NVMe-oF)
	- Ability to run its own operating system (like Linux or specialized distributions like NVIDIA DOCA) independently from the host server.
##### Hardware architecture
![[Pasted image 20250413150420.png]]
Divided across 4 layers
- Top Layer: GPU
	- Second Layer: Multiple Streaming Processors (SM)
		- Third Layer: Each SM contains multiple partitions
			- Fourth Layer: Core (Ex: FP32)
##### Software-Hardware relationship
![[Pasted image 20250413150727.png]]

Each task can be considered as a task and each SM can be considered as a worker. We need a unit (master) that can allocate the blocks to the SM. This unit is called as the **GigaThread** unit. 

![[Pasted image 20250413150941.png]]

If GigaThread assigns the block for a single Streaming Multiprocessor, we will have only one partition assigned to the block. This leads to wastage of partitions inside the SM. Here comes **Warp**.

![[Pasted image 20250413151212.png]]

Each block will be divided into Warp so that we can use the partitions offered by the Streaming Multiprocessors. If we have more Warps than partitions, we will execute multiple warps on a single partition. In order to manage Warps across partitions, we need a manager and every partition will have a **Warp Scheduler.**

![[Pasted image 20250413151544.png]]

**A Warp typically consists of 32 threads by default.** 
#### Numbers

| Property                            | H100                                                  | B200                       |
| ----------------------------------- | ----------------------------------------------------- | -------------------------- |
| Max number of threads per block     | 1024                                                  | 1024                       |
| Number of Streaming MultiProcessors | Chip: 144<br><br>Product:<br>SXM5 - 132<br>PCIe - 114 | 208<br><br>104 per die x 2 |
| Number of threads per SM            | 2048                                                  | 2048                       |
| Number of concurrent wraps          | 64                                                    | 64                         |
| Number of thread blocks per SM      | 32                                                    | 32                         |
|                                     |                                                       |                            |
#### Vector Addition
![[Pasted image 20250413182728.png]]
![[Pasted image 20250413183135.png]]

![[Pasted image 20250413183204.png]]
- Case 1
	- Blocks = 2
	- Threads = 1024
	- Since one block runs on one SM, we will be using 2 SM's for the execution. But H100 has 144 SM. So we are using 2 out of 144 SM. Thus, our GPU utilization is bad.
	- This metric is called as **GPU Utilization.** The above use case has pretty bad GPU Utilization
- Actions taken to increase utilization
	- We need to increase the number of blocks to utilize the SM effectively. The minimum number of threads we need to keep for a block = 32 as it is equal to 1 warp. So 2048 / 32 = 64 blocks. So we will be using 64 SM, way better than 2 from the above case.

#### Runtime API's
- https://docs.nvidia.com/cuda/cuda-runtime-api/index.html
- Sample Code: https://github.com/hamdysoltan/CUDA_Course/blob/main/section4_001_runtime_apis.cu
- High level interface to CUDA
- Harness the power of NVIDIA GPU's
- Managing GPU devices, memory allocation and execution of parallel kernels
- Example:
	- GPU limits such as maximum number of threads per block
- API's function operated differently
	- You need to pass an argument to store the results
- Return an error state to investigate
- Saves time to get values such as number of SM's, number of thread per block
#### nvidia-smi (NVIDIA System Management Interface)
##### 3 Key Features
- Performance monitoring
	- Utilization
	- Memory Usage
	- Temperature
	- Power
- Settings Management
	- Controlling the clock speed
	- Power limits
- Device information querying
	- GPU Name
	- Driver Version
##### Commands
```
$nvidia-smi
```

No processes running on the GPU

![[Pasted image 20250413191350.png]]

Vector addition running on the GPU

![[Pasted image 20250413191603.png]]

Running nvidia-smi every 5 seconds

```
nvidia-smi -l 5
```

Getting specific metrics using nvidia-smi

```
nvidia-smi --query-gpu=gpu_name,driver_version --format=csv
```

Changing power limits of a GPU:
- i refers to the index of the GPU which starts from 0

```
nvidia-smi -i 0 -pl 200
```

Query and display detailed information specifically about the various clock speeds
- q stands for query
- d stands for display

```
nvidia-smi -q -d CLOCK
```

![[Pasted image 20250413192638.png]]
The clock speed is lower than the max clock in the above screenshot because NVIDIA will reduce the clock speed to save power. Higher clock speed = Higher power.

To check all the supported clocks that can be used

```
nvidia-smi -q -d SUPPORTED_CLOCKS
```

#### Occupancy
- Measure of utilization of the resources of the GPU
#### GPU's Occupancy and Latency Binding
![[Pasted image 20250414151734.png]]

Theoretical occupancy calculation = **Warp used in a kernel / max warps per SM**
Example:
- Assuming a kernel utilizes 48 warps - 48 / 48 = 100% occupancy
- Assuming a kernel utilizes 12 warps - 12 / 48 = 25% occupancy
#### Profiling a kernel

Command

```
ncu ./<compiled_cu_filename>
```
![[Pasted image 20250414153221.png]]
 
 Block limit per SM = 16
 Block size = 64 threads
 Total number of threads per SM = 16 * 64 = 1024
 Max threads per SM = 1536
 Utilization = 1024 / 1536 = 0.667 = 66.67%
##### Theoretical Occupancy
 - Refers to the ideal circumstances
 - Optimal conditions where there are enough independent tasks
	 - Without being bottlenecked by memory, computation and dependency
##### Actual Occupancy
When you have multiple warps sent to a partition, Warp Scheduler takes the responsibility of being the master assigning warps. The dispatcher will take the 
responsibility of assigning data to the cores based on the type of instruction. For example FMUL will go to FP32 cores, ISEPT will go to INT cores, etc.

![[Pasted image 20250414162427.png]]
##### Out of order execution
Consider a situation where we have the following instruction pipeline:
- Instruction 1
	- Instruction 1.1
- Instruction 2
 Instruction 1.1 is dependent on Instruction 1 which will take time. In order to effectively utilize CPU cycles, the CPU will perform out of order execution by executing Instruction 2. 
##### Cycles count

| Access data from data source | Number of cycles |
| ---------------------------- | ---------------- |
| L1 cache                     | 30               |
| L2 cache                     | 190-200          |
| Global memory                | 300              |
##### Achieved Occupancy

![[Pasted image 20250414164330.png]]

= Number of Active Cycles / Number of Total Cycles

- Computation is replicated for each of the 4 partitions within an SM
- The average of all petitions is calculated (occupancy / SM)
- This methodology is applied across all SMs.
- Overall achieved occupancy value is determined by averaging the values across all SM's.
#### Allocated Active Blocks per SM (AABS)

![[Pasted image 20250414165243.png]]
##### Max Block / SM
- No more than 32 blocks per SM can run concurrently 
##### Max Warps / SM
- Max warps that can be executed concurrently on a single SM

Regardless of the number of blocks per SM, number of warps per SM will dictate the number of blocks per SM. Warp limit controls how many active blocks can be allocated per SM.
###### Example
- 32 blocks per SM
- Each block contains 4 warps
- Total warps would be 32 * 4 = 128 warps
**Solution**
- The compiler will intervene with the following logic
- Decrease the allocated blocks per SM to 16
- So 16 * 4 = 64 warps per SM which meets our given criteria from the table above

- We can assign million blocks / GPU
- These blocks will be distributed across 108 SM
- We can assign more than 1000 blocks per SM
- But only 32 will run concurrently 
##### No more than Max registers / SM
- One block per SM with a size of 1024 threads
- Each thread requires 100 registers
- More than 100,000 registers/SM are required
- The limit is 65336 (This is a problem)
- We can't decrease the number of blocks per SM
- Decrease the number of threads per block
- Otherwise: Local memory will be used i.e. **Register Spilling**

Example 2:
- Consider the block size is 512 threads
- Assume that each thread requires 16 registers
- We can't control the allocated registers per thread
- Registers needed for the block = 512 * 16 = 8192 registers
- Required registers < Register limit
	- 65536 / 8192 = 8 blocks
#### Shared Memory Size / SM
