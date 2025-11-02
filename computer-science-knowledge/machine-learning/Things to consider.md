https://github.com/k8sgpt-ai/k8sgpt
https://github.com/Lightning-AI/litgpt
 

- NCCL overhead
- High tensor core utilization
- Watchdog
- FSDP (Fully Sharded Data Parallel)
	- torch.compile only focus on optimizing compute
	- Strategies
		- Auto bucketing 
		- Memory based prefetching

### Networking for AI

![[Pasted image 20250414114227.png]]

- Single PCI link that serves both the NIC and the connectivity through the GPU
- CPU is the link to the outside world in terms of the front end network and connectivity

![[Pasted image 20250414114516.png]]

Difference between B200 and H100 is twice the speed on the NVLink

![[Pasted image 20250414115025.png]]

Grace CPU is directly connected to the B200 GPU so we don't have the same contention on the PCIe bus. It uses the Chip to Chip connectivity system.

NVLink efficiency is not well documented but seems poor, so actual throughput peaks at 75% (raw) and can be lower again in NCCL. NVLink protocol is proprietary and the assumption is that the low level protocol that NVLink doesn't tends to not use large messages. This leads to a poor ratio between the headers and content.

![[Pasted image 20250414120602.png]]

- 4 GPU Compute Tray * 18 Compute Nodes = 72 GPU
- 2 B200 are connected to one Grace CPU
- External NVSwitch trays provide connectivity
- No NVLink connectivity within a single compute node
- If one of the GPU Node crashes, the 72 nodes stop working and we need to recover the model from the checkpoints

![[Pasted image 20250414123701.png]]

Further talks to listen to: https://www.nvidia.com/en-us/on-demand/session/gtc24-s62129/

### NVIDIA Collective Communications Library

**Purpose:** NCCL is a library that provides highly optimized routines for collective communication operations between multiple GPUs, both within a single machine (intra-node) and across multiple machines (inter-node).

**Collective Communications:** These are operations where multiple processors (in this case, GPUs) participate simultaneously in a communication pattern. Common examples implemented in NCCL include:

- `All-Reduce`: Combine data from all GPUs and distribute the result back to all GPUs.
- `Broadcast`: Send data from one GPU to all other GPUs.
- `Reduce`: Combine data from all GPUs onto a single root GPU.
- `All-Gather`: Gather data from all GPUs onto every GPU.
- `Reduce-Scatter`: Combine data from all GPUs and scatter portions of the result back to different GPUs.

### Peripheral Component Internet Express (PCIe)

![[Pasted image 20250415160730.png]]
![[Pasted image 20250415160851.png]]
![[Pasted image 20250415161858.png]]
