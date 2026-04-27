---
title: Run memory access analysis with Performix and MCP

description: Learn how to profile memory behavior in a C++ particle simulation on Arm Linux using the Performix Memory Access recipe through the Arm MCP Server.

minutes_to_complete: 45

who_is_this_for: This is an introductory topic for intermediate C++ developers with basic cache and virtual-memory knowledge who want to diagnose cache and TLB bottlenecks with Arm Performix and MCP.

learning_objectives:
  - Explain how L1 cache behavior, translation lookaside buffer misses, and page walks affect runtime
  - Build and run the orbiting galaxies workload without using helper build scripts
  - Use Arm MCP to run the Performix Memory Access recipe on a remote Arm target
  - Identify data layout risks in an Array-of-Structures implementation before optimization

prerequisites:
  - Basic C++ development experience
  - Basic understanding of cache hierarchy and virtual memory translation
  - Access to a remote Arm Linux server with Arm MCP Server configured
  - "Arm Performix installed (see placeholder install guide: [Install Arm Performix](https://learn.arm.com/install-guides/performix-placeholder/))"
  - Git, CMake, a C++ compiler toolchain, and Python 3 installed
  - Familiarity with terminal-based workflows

author: Kieran Hejmadi

### Tags
skilllevels: Introductory
subjects: Performance and Architecture
armips:
  - Neoverse
tools_software_languages:
  - Arm Performix
  - MCP
  - C++
  - Linux perf
  - CMake
operatingsystems:
  - Linux

further_reading:
  - resource:
      title: Find Code Hotspots with Arm Performix
      link: https://learn.arm.com/learning-paths/servers-and-cloud-computing/cpu_hotspot_performix/
      type: learning-path
  - resource:
      title: Optimize application performance using Arm Performix CPU microarchitecture analysis
      link: https://learn.arm.com/learning-paths/servers-and-cloud-computing/performix-microarchitecture/
      type: learning-path
  - resource:
      title: Automate x86-to-Arm application migration using Arm MCP Server
      link: https://learn.arm.com/learning-paths/servers-and-cloud-computing/arm-mcp-server/
      type: learning-path
  - resource:
      title: Arm Performix
      link: https://developer.arm.com/servers-and-cloud-computing/arm-performix
      type: website

### FIXED, DO NOT MODIFY
# ================================================================================
weight: 1                       # _index.md always has weight of 1 to order correctly
layout: "learningpathall"       # All files under learning paths have this same wrapper
learning_path_main_page: "yes"  # This should be surfaced when looking for related content. Only set for _index.md of learning path content.
---

Memory-bound code can look simple and still run slowly. In this Learning Path, you profile a particle simulation where most of the cost comes from memory traffic, not arithmetic.

You focus on three memory hierarchy signals:

- **L1 cache efficiency**: low L1 hit rates force demand loads to wait on deeper cache levels or DRAM.
- **TLB pressure**: when the working set spans many pages, translation lookaside buffer misses increase translation overhead.
- **Page walks**: after a TLB miss, hardware must walk page tables, which adds latency before data access can complete.

The workload uses an Array-of-Structures layout where each particle is 64 bytes but only part of each particle is used in the hot loop. This can inflate bandwidth demand and reduce effective cache utilization.

Before you begin the hands-on steps, install Arm Performix using this placeholder guide link: [Install Arm Performix](https://learn.arm.com/install-guides/performix-placeholder/).

## Manual baseline approach with Linux perf

Before using Performix, you can inspect memory behavior manually with Linux `perf`.

```bash
# Example baseline run
perf stat -d -d -d ./build/baseline

# Example sampled profile (function-level)
perf record -g ./build/baseline
perf report
```

Manual `perf` is useful, but it requires event selection, interpretation, and correlation across multiple outputs. The Performix Memory Access recipe packages this workflow and returns memory-focused metrics in one run. In this Learning Path, you execute that recipe through Arm MCP so an AI agent can orchestrate the run and return structured output.
