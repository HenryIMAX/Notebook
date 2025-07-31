# 性能分析技术基础 Profiling

## Introduction

 **Profile Categories：** 

- System
- Program

**系统自带的性能分析工具：**

- cpufp
- core-to-core-latency
- STREAM：测量内存带宽
- cpu-micro-benchmark：用于逆向CPU微架构的工具

## CPU Profiling Tool

### Perf

**Common commands:**

- perf top
- perf report
- perf record
- perf stat

```bash
perf record -F 99 -g cycles.instructions.cache-miss
```

### Vtune

**Analysis Types:**

- hostports
- threading
- memory access
- micro-architecture

## GPU Profiling Tool

### Nsight Systems

### Nsight Compute

以dgemm_gpu为例：

```bash
mkdir -p nsight_results
ncu --set full --export nsight_results/matmul_profile.ncu-rep --force-overwrite ./dgemm_gpu
```

## Roofline Model

computing platform: **computational capability(FLOPS)** and **memory bandwidth**



