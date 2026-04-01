---
title: Conclusion
weight: 6

### FIXED, DO NOT MODIFY
layout: learningpathall
---


Run the microbenchmark sweep to compare baseline versus accelerated generation across input sizes:

```bash
make clean
cmake -S . -B build-baseline -DBUILD_TESTS=ON
cmake -S . -B build-apl -DBUILD_TESTS=ON -DUSE_APL=ON

cmake --build build-baseline --target sweep_microbench_baseline
cmake --build build-apl --target sweep_microbench_with_apl
```

``bash
./build-baseline/tests/sweep_microbench_baseline
```

```output
Generating Distribution of size 256 = 174 us
Generating Distribution of size 512 = 280 us
Generating Distribution of size 1024 = 554 us
Generating Distribution of size 2048 = 1117 us
Generating Distribution of size 4096 = 2231 us
Generating Distribution of size 8192 = 4429 us
Generating Distribution of size 16384 = 8908 us
Generating Distribution of size 32768 = 17818 us
```

```bash
./build-apl/tests/sweep_microbench_with_apl
```

```output
Generating Distribution of size 256 = 58 us
Generating Distribution of size 512 = 59 us
Generating Distribution of size 1024 = 102 us
Generating Distribution of size 2048 = 186 us
Generating Distribution of size 4096 = 370 us
Generating Distribution of size 8192 = 709 us
Generating Distribution of size 16384 = 1431 us
Generating Distribution of size 32768 = 2785 us
```

As data size grows, acceleration from OpenRNG and Arm Performance Libraries should become greater because the the overhead of using openRNG compared to the standard library `std::mt19937` is a smaller portion. 

## Conclusion

You completed a full optimization workflow on Arm:

- Built and measured a baseline C++ workload
- Used Arm Performix Code Hotspots to find the highest-cost function class
- Accelerated random generation with OpenRNG and Arm Performance Libraries
- Improved flame graph clarity by rebuilding OpenRNG with debug symbols
- Verified performance gains with a size sweep microbenchmark

You are now ready to apply the same profile-first optimization method to your own Arm workloads.
