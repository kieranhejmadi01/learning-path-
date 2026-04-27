---
title: Background
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

## Refresher on CPU Memory Hierarchy

This learning path assumes you have a basic understanding of memory hierarchy in modern systems. This page is not meant to be an extensive explanation but a quick refresher of some of the memory concepts covered in this learning path. Modern CPUs use a cache hierarchy. You typically see:

- L1 data cache (`L1d`) and L1 instruction cache (`L1i`) per core
- L2 cache, often private per core
- Last-level cache (LLC), often an L3 cache shared across cores

You can inspect your system cache hierarchy with the following command:

```bash
lscpu | grep -i cache
```

Example output:

```output
L1d cache:                               4 MiB (64 instances)
L1i cache:                               4 MiB (64 instances)
L2 cache:                                64 MiB (64 instances)
L3 cache:                                32 MiB (1 instance)
```

In this example we are using an AWS Graviton 3 instance based on the Neoverse V1 architecture. Each core has its own L1 and L2 caches (64 instances), while L3 is shared (1 instance across the system).

L1 data cache is the first level your core checks for load and store data. If your hot loop has strong locality and good cache line utilization, many accesses resolve quickly in L1.

When L1 hit rate drops, your code waits on deeper levels (L2 or DRAM), and latency rises.

Working set size also matters. The working set is the amount of code and data your application actively touches over a period of execution. If the working set is larger than available cache capacity, cache misses increase because lines are evicted before reuse. 

## TLB misses and page walks

The translation lookaside buffer (TLB) caches virtual-to-physical page translations. A TLB miss means the processor must perform a page table walk to resolve the address before the data access can complete.

Frequent TLB misses and page walks increase memory access latency, especially for large working sets with poor spatial locality.

If data is not found in cache, accesses continue toward DRAM. DRAM has much higher latency than on-chip caches, so cache misses and translation overhead can dominate runtime even when the compute loop is simple.

Some applications do not fit neatly into cache by design, especially with large datasets or irregular access patterns. In multithreaded code, additional unintentional effects such as false sharing can also reduce cache efficiency. For a focused walkthrough, see:

- [Analyze false sharing with Arm SPE](https://learn.arm.com/learning-paths/servers-and-cloud-computing/false-sharing-arm-spe/)

## Why data layout changes these metrics

Modern CPUs depend heavily on the memory hierarchy—caches, TLBs, and hardware prefetchers—to achieve high performance. Compilers can reorder accesses and improve locality to some extent, but they cannot fix fundamentally poor data layout or access patterns. If your program has low spatial or temporal locality, scattered allocations, or excessive pointer indirection, it will incur more cache misses, TLB misses, and memory latency. In such cases, performance is limited not by compute, but by how efficiently data moves through the memory system.

## Summary

Memory access behavior can be complex, and optimization decisions need observability from a running workload rather than assumptions from source code alone.

Linux provides low-level performance tools, but there is a learning curve in both collecting the right data and interpreting the results correctly. The Performix Memory Access recipe exists to streamline that workflow and present memory-focused evidence in a structured way.

In the next section, you build the workload and prepare a reproducible run. Then you use Arm Performix through MCP to gather memory-access-specific evidence in one workflow.
