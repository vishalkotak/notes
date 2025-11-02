#### active_bytes.all.peak
Peak total amount of GPU memory, measured in bytes, that was simultaneously occupied by active PyTorch tensors across all memory pools on the specified device.

**Approach**
torch.cuda.memory_stats(self.device)['active_bytes.all.peak']

**Details:**
- active_bytes (Metric): This refers to the amount of GPU memory **actively being used** to store the data of PyTorch **tensors**. It's essentially the memory occupied by tensors that are currently live and part of the computation or data handling.
- all (Pool): PyTorch's caching allocator often manages memory in different pools (e.g., `small_pool` for small allocations, `large_pool` for larger ones) to optimize performance. The `all` designation means this statistic is **aggregated across all these memory pools** for the given GPU device. It gives you the total picture rather than the usage within a specific pool.
- peak(Type): This indicates that the value represents the **maximum value (high-water mark)** that the `active_bytes.all` metric reached during the execution period being measured. The period starts when the program begins or when the peak statistics were last reset using `torch.cuda.reset_peak_memory_stats(device)`.

**Significance**
- This is often considered a more accurate measure of the _true_ memory requirement of your model and workload compared to `reserved_bytes`, which includes cached memory not currently in use.
- Comparing this value to the total memory of your GPU helps you understand how close your application came to exhausting memory solely due to tensor allocations.
- If you divide this value by `1024**3`, you get the `max_active_gib` (maximum active Gibibytes) value often reported in memory summaries.

**Percentage calculation**:

100 * (memory * (1024 ** 3)) / total_memory_on_the_device (torch.cuda.get_device_properties(self.device).total_memory)

### Inference Metrics

Time to First Token (TTFT): Time it takes for a user to start getting a response from a model after submitting their prompt. Determined by the time taken by the model to process the user's input and generate first completion token.

Time Per Output Token (TPOT) / Time Between Tokens (TBT)

Latency: Overall time it takes to generate the complete response for a user. It can be calculated using the above two metrics and is a key metric for assessing the overall responsiveness of the model

Throughput: A measure on how may tokens the model can output in a given time span. This is typically measured using tokens per second.

Goal: Minimize TTFT, Maximize Throughput, Optimize TPOT for the best overall performance.

Model Flop Utilization (MFU):

Model Bandwidth Utilization (MBU): 

Benchmarking LLM's:
- LLMPerf
###  GPU operator
https://www.youtube.com/watch?v=f-FKoMHqRoI

![[Pasted image 20250415144241.png]]

![[Pasted image 20250415144328.png]]

![[Pasted image 20250415144932.png]]

GPU operator is sent to K8 management nodes through the control plane.
### DGCM
**DGCM** stands for **Data Center GPU Manager**. It's a suite of tools from NVIDIA primarily used for monitoring and managing NVIDIA GPUs in servers and clusters.
#### Telemetry Setup

**Prometheus:** This is very common in Kubernetes environments. The DGCM components are often configured to run an **exporter**. This exporter exposes the collected GPU metrics on a specific network port (like `/metrics`). A Prometheus server deployed in the cluster is then configured to "scrape" (pull) the data from these exporters periodically. The operator helps deploy and configure these exporters.
### Single Node GPU training run

```
torchrun
--nproc_per_node=${NGPU} \
--rdzv_backend c10d \
--rdzv_endpoint="localhost:0" \ 
--local-ranks-filter ${LOG_RANK} \
--role rank \
--tee 3
./ootf/workloads/base.py \
--job.config_file ${CONFIG_FILE} $overrides
```

- CONFIG_FILE: Model.toml file
- nproc: Stands for number of processes
- per_node: Means "on_each_machine" or "on_each_node"
- In distributed data-parallel training, you typically want one worker node per GPU you intend to use on that machine
- Command execution details:
	- Starts itself
	- Spawns 8 independent child processes on the same machine
	- Each of the child processes will execute your script
	- LOCAL_RANK: A unique ID for each process on that specific node ranging from 0 to (nproc_per_node - 1)
	- RANK: A unique ID for each process across all nodes (if you are running mult-node). For single node training, RANK is the same as LOCAL_RANK

### H100 vs GB200 Comparison

| **Feature**                              | **8x H100 System (e.g., DGX H100)**  | **GB200 NVL72 System**                                        | **Key Difference Highlight**                                 |
| ---------------------------------------- | ------------------------------------ | ------------------------------------------------------------- | ------------------------------------------------------------ |
| System Scale                             | Single Server                        | Single Rack (Multiple Trays)                                  | Server level vs. integrated Rack level                       |
| GPU Architecture                         | Hopper (H100)                        | Blackwell (B200)                                              | Newer, more powerful Blackwell architecture                  |
| Number of GPUs                           | 8                                    | 72                                                            | 9x more GPUs in a single NVL72 domain                        |
| Number of CPUs                           | Typically 2x x86 (e.g., Intel Xeon)  | 36x Grace (Arm-based)                                         | Tightly integrated Grace CPUs per Superchip vs. separate x86 |
| Core Unit                                | 8x H100 SXM GPUs                     | 36x GB200 Superchips (1 CPU + 2 GPUs each)                    | Superchip design integrates CPU+GPU tightly                  |
| GPU Interconnect                         | 4th Gen NVLink (within server)       | 5th Gen NVLink (via NVLink Switch Fabric across rack)         | Faster, rack-scale NVLink fabric connecting all 72 GPUs      |
| GPU-to-GPU Bandwidth (Aggregate per GPU) | 900 GB/s (within server)             | 1.8 TB/s (within NVLink domain)                               | 2x higher bandwidth per GPU across the entire system         |
| Total HBM Memory                         | 640 GB (8 * 80GB HBM3)               | 13.5 TB (72 * 192GB HBM3e)                                    | ~21x more total high-bandwidth memory                        |
| Memory Type                              | HBM3                                 | HBM3e                                                         | Faster HBM3e memory standard                                 |
| AI Performance (FP8)                     | ~32 PFLOPS (8 * 4 PFLOPS)            | ~720 PFLOPS (72 * 10 PFLOPS)                                  | ~22.5x higher FP8 performance (inference focus)              |
| AI Performance (FP4)                     | N/A (Hopper focus is FP8/FP16)       | ~1440 PFLOPS (72 * 20 PFLOPS)                                 | Massive FP4 performance for inference (Blackwell feature)    |
| AI Performance (FP16/BF16)               | ~16 PFLOPS (8 * 2 PFLOPS)            | ~360 PFLOPS (72 * 5 PFLOPS)                                   | ~22.5x higher FP16/BF16 performance (training focus)         |
| Networking (External)                    | Typically 8x 400Gb/s OSFP ports      | Integrated high-density fabric connectivity (e.g., CX8-based) | Designed for massive scale-out with integrated fabric        |
| Cooling                                  | Primarily Air-cooled (can be liquid) | Liquid Cooled                                                 | High density necessitates liquid cooling                     |
| Approx. Power                            | ~10-12 kW per server                 | ~120 kW per rack                                              | Significantly higher density and power per rack              |
| Primary Use Case                         | Large Model Training, HPC            | Massive LLM Inference & Training, Exascale HPC                | Extreme scale for the largest AI models and simulations      |
### CX7 vs CX8 difference
| Feature                      | ConnectX-7 (CX7)                                                        | ConnectX-8 (CX8)                                                                                | Key Difference Highlight                      |
| ---------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------- |
| Maximum Data Rate (Per Port) | 400 Gb/s                                                                | 800 Gb/s                                                                                        | 2x increase in maximum per-port speed         |
| Supported Protocols          | NDR InfiniBand, Ethernet (up to 400GbE)                                 | Expected: >NDR InfiniBand, Ethernet (up to 800GbE)                                              | Support for next-generation 800G standards    |
| PCI Express Interface        | PCIe Gen 5.0 (x16 or x32 depending on model)                            | Expected: PCIe Gen 6.0                                                                          | Upgrade to next PCIe generation for bandwidth |
| Port Density/Form Factor     | Typically 1 or 2 ports (OSFP, QSFP112)                                  | Expected: 1 or 2 ports (using next-gen 800G optics like OSFP or QSFP-DD800)                     | Adoption of new optics for 800G speeds        |
| In-Network Computing         | Advanced acceleration engines (RDMA, RoCE, GPUDirect Storage, ASAP²)    | Expected: Enhanced acceleration engines, potentially lower latency, improved AI/HPC collectives | Further advancements in offload capabilities  |
| Security Features            | Hardware Root of Trust, Connection Tracking, IPsec/MACsec/TLS offload   | Expected: Enhanced security features, potentially aligned with BlueField-4 capabilities         | Continued evolution of security offloads      |
| DPU Integration              | Integrated into BlueField-3 DPUs                                        | Expected: Integration into future BlueField-4 DPUs                                              | Aligned with the next generation of DPUs      |
| Target Platforms             | Hopper GPU architecture (e.g., H100 systems), current HPC & AI clusters | Blackwell GPU architecture (e.g., GB200 systems), next-gen AI supercomputers                    | Designed for the bandwidth needs of Blackwell |
| Power Efficiency             | Baseline for 400G generation                                            | Expected: Improved performance-per-watt over CX7                                                | Focus on efficiency alongside speed increase  |
| Availability (Apr 2025)      | Widely Available                                                        | Announced; expected with Blackwell platform rollout (sampling/early availability)               | CX7 is current gen; CX8 is next-gen           |
### Metrics from DGCM exporter

**Common Metrics Exposed by dcgm-exporter:**

**GPU Utilization (%)**

- `dcgm_gpu_utilization` (Gauge): GPU core utilization percentage. (Maps to `DCGM_FI_DEV_GPU_UTIL`)
- `dcgm_mem_copy_utilization` (Gauge): Memory controller utilization percentage. (Maps to `DCGM_FI_DEV_MEM_COPY_UTIL`)
- `dcgm_enc_utilization` (Gauge): Video Encoder utilization percentage. (Maps to `DCGM_FI_DEV_ENC_UTIL`)
- `dcgm_dec_utilization` (Gauge): Video Decoder utilization percentage. (Maps to `DCGM_FI_DEV_DEC_UTIL`)

**Memory**

- `dcgm_fb_free` (Gauge): Free framebuffer memory (in MiB). (Maps to `DCGM_FI_DEV_FB_FREE`)
- `dcgm_fb_used` (Gauge): Used framebuffer memory (in MiB). (Maps to `DCGM_FI_DEV_FB_USED`)
- `dcgm_fb_total` (Gauge): Total framebuffer memory (in MiB). (Maps to `DCGM_FI_DEV_FB_TOTAL`)
    - _Note:_ Some configurations might expose `dcgm_frame_buffer_free`, `dcgm_frame_buffer_used`, `dcgm_frame_buffer_total` in MB instead.
- `dcgm_bar1_used` (Gauge): Used BAR1 memory (in MiB). (Maps to `DCGM_FI_DEV_BAR1_USED`)
- `dcgm_bar1_free` (Gauge): Free BAR1 memory (in MiB). (Maps to `DCGM_FI_DEV_BAR1_FREE`)
- `dcgm_bar1_total` (Gauge): Total BAR1 memory (in MiB). (Maps to `DCGM_FI_DEV_BAR1_TOTAL`)

**Temperature (°C)**

- `dcgm_gpu_temp` (Gauge): GPU core temperature. (Maps to `DCGM_FI_DEV_GPU_TEMP`)
- `dcgm_mem_temp` / `dcgm_memory_temp` (Gauge): GPU memory temperature. (Maps to `DCGM_FI_DEV_MEMORY_TEMP`)
- `dcgm_slowdown_temp` (Gauge): Slowdown temperature limit. (Maps to `DCGM_FI_DEV_SLOWDOWN_TEMP`)

**Power (Watts / Joules)**

- `dcgm_power_usage` (Gauge): Current GPU power draw in Watts. (Maps to `DCGM_FI_DEV_POWER_USAGE`)
- `dcgm_total_energy_consumption` (Counter): Total energy consumed since driver load/boot in milliJoules (mJ). (Maps to `DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION`)
- `dcgm_power_management_limit` (Gauge): Current power limit in Watts. (Maps to `DCGM_FI_DEV_POWER_MGMT_LIMIT`)

**Clocks (MHz)**

- `dcgm_sm_clock` (Gauge): Current SM (Streaming Multiprocessor) clock frequency. (Maps to `DCGM_FI_DEV_SM_CLOCK`)
- `dcgm_mem_clock` (Gauge): Current memory clock frequency. (Maps to `DCGM_FI_DEV_MEM_CLOCK`)
- `dcgm_video_clock` (Gauge): Current video encoder/decoder clock frequency. (Maps to `DCGM_FI_DEV_VIDEO_CLOCK`)

**Errors and Violations**

- `dcgm_xid_errors` (Gauge): Value/timestamp of the last XID (GPU internal) error encountered. (Maps to `DCGM_FI_DEV_XID_ERRORS`)
- `dcgm_pcie_replay_counter` (Counter): Total count of PCIe replays (errors). (Maps to `DCGM_FI_DEV_PCIE_REPLAY_COUNTER`)
- `dcgm_ecc_sbe_volatile_total` (Counter): Total Single-Bit volatile ECC errors. (Maps to `DCGM_FI_DEV_ECC_SBE_VOL_TOTAL`)
- `dcgm_ecc_dbe_volatile_total` (Counter): Total Double-Bit volatile ECC errors. (Maps to `DCGM_FI_DEV_ECC_DBE_VOL_TOTAL`)
- `dcgm_ecc_sbe_aggregate_total` (Counter): Total Single-Bit aggregate (persistent) ECC errors. (Maps to `DCGM_FI_DEV_ECC_SBE_AGG_TOTAL`)
- `dcgm_ecc_dbe_aggregate_total` (Counter): Total Double-Bit aggregate (persistent) ECC errors. (Maps to `DCGM_FI_DEV_ECC_DBE_AGG_TOTAL`)
- `dcgm_retired_pages_sbe` (Counter): Count of retired pages due to single-bit errors. (Maps to `DCGM_FI_DEV_RETIRED_SBE`)
- `dcgm_retired_pages_dbe` (Counter): Count of retired pages due to double-bit errors. (Maps to `DCGM_FI_DEV_RETIRED_DBE`)
- `dcgm_retired_pages_pending` (Gauge): Count of pages pending retirement. (Maps to `DCGM_FI_DEV_RETIRED_PENDING`)
- `dcgm_nvlink_crc_flit_error_count_total` (Counter): NVLink flow-control CRC error count. (Maps to `DCGM_FI_DEV_NVLINK_CRC_FLIT_ERROR_COUNT_TOTAL`) _Requires specific config_
- `dcgm_nvlink_crc_data_error_count_total` (Counter): NVLink data CRC error count. (Maps to `DCGM_FI_DEV_NVLINK_CRC_DATA_ERROR_COUNT_TOTAL`) _Requires specific config_
- `dcgm_nvlink_replay_error_count_total` (Counter): NVLink replay error count. (Maps to `DCGM_FI_DEV_NVLINK_REPLAY_ERROR_COUNT_TOTAL`) _Requires specific config_
- `dcgm_nvlink_recovery_error_count_total` (Counter): NVLink recovery error count. (Maps to `DCGM_FI_DEV_NVLINK_RECOVERY_ERROR_COUNT_TOTAL`) _Requires specific config_
- `dcgm_row_remap_failure` (Gauge): Whether remapping of rows has failed (0 or 1). (Maps to `DCGM_FI_DEV_ROW_REMAP_FAILURE`)
- `dcgm_uncorrectable_remapped_rows` (Counter): Number of remapped rows for uncorrectable errors. (Maps to `DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS`)
- `dcgm_correctable_remapped_rows` (Counter): Number of remapped rows for correctable errors. (Maps to `DCGM_FI_DEV_CORRECTABLE_REMAPPED_ROWS`)
- _Throttling Violations (Counters, in microseconds - often commented out by default):_ Power, Thermal, Sync Boost, Board Limit, Low Utilization, Reliability. (e.g., `DCGM_FI_DEV_POWER_VIOLATION`)

**PCIe Throughput (Bytes/sec or KB/sec depending on exporter version/config)**

- `dcgm_pcie_tx_throughput` (Counter/Gauge): PCIe transmit throughput. (Maps to `DCGM_FI_PROF_PCIE_TX_BYTES` or older `DCGM_FI_DEV_PCIE_TX_THROUGHPUT`)
- `dcgm_pcie_rx_throughput` (Counter/Gauge): PCIe receive throughput. (Maps to `DCGM_FI_PROF_PCIE_RX_BYTES` or older `DCGM_FI_DEV_PCIE_RX_THROUGHPUT`)

**NVLink Throughput / Bandwidth**

- `dcgm_nvlink_bandwidth_total` (Counter): Total NVLink bandwidth counter across all lanes. (Maps to `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL`)
- `dcgm_prof_nvlink_rx_bytes` (Gauge): Rate of NVLink receive data in bytes/sec. (Maps to `DCGM_FI_PROF_NVLINK_RX_BYTES`) _Profiling metric_
- `dcgm_prof_nvlink_tx_bytes` (Gauge): Rate of NVLink transmit data in bytes/sec. (Maps to `DCGM_FI_PROF_NVLINK_TX_BYTES`) _Profiling metric_

**Performance States**

- `dcgm_pstate` (Gauge): Current performance state (P-State) from 0 (highest) to 15 (lowest). (Maps to `DCGM_FI_DEV_PSTATE`)

**Fan Speed**

- `dcgm_fan_speed` (Gauge): Fan speed percentage (0-100). (Maps to `DCGM_FI_DEV_FAN_SPEED`)

**Profiling Metrics (often need explicit enabling, measure activity ratios)**

- `dcgm_prof_sm_active` (Gauge): Ratio of cycles an SM has at least 1 warp assigned (%). (Maps to `DCGM_FI_PROF_SM_ACTIVE`)
- `dcgm_prof_sm_occupancy` (Gauge): Ratio of resident warps on an SM to the theoretical maximum (%). (Maps to `DCGM_FI_PROF_SM_OCCUPANCY`)
- `dcgm_prof_pipe_tensor_active` (Gauge): Ratio of cycles the Tensor Core (HMMA) pipes are active (%). (Maps to `DCGM_FI_PROF_PIPE_TENSOR_ACTIVE`)
- `dcgm_prof_dram_active` (Gauge): Ratio of cycles the DRAM interface is active sending/receiving data (%). (Maps to `DCGM_FI_PROF_DRAM_ACTIVE`)
- `dcgm_prof_pipe_fp64_active` (Gauge): Ratio of cycles the FP64 pipes are active (%). (Maps to `DCGM_FI_PROF_PIPE_FP64_ACTIVE`)
- `dcgm_prof_pipe_fp32_active` (Gauge): Ratio of cycles the FP32 pipes are active (%). (Maps to `DCGM_FI_PROF_PIPE_FP32_ACTIVE`)
- `dcgm_prof_pipe_fp16_active` (Gauge): Ratio of cycles the FP16 pipes are active (%). (Maps to `DCGM_FI_PROF_PIPE_FP16_ACTIVE`)
- `dcgm_prof_gr_engine_active` (Gauge): Ratio of time the graphics/compute engine is active (%). (Maps to `DCGM_FI_PROF_GR_ENGINE_ACTIVE`)

**vGPU Metrics (if applicable)**

- `dcgm_vgpu_license_status` (Gauge): vGPU license status (e.g., 0=Unlicensed, 1=Licensed, etc.). (Maps to `DCGM_FI_DEV_VGPU_LICENSE_STATUS`)
