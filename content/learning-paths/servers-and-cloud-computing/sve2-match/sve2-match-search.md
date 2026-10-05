---
# User change
title: "Compare search performance using scalar and SVE2 MATCH on Arm Servers"

weight: 2

layout: "learningpathall"


---
## Introduction

Searching large arrays for specific values is a core task in performance-sensitive applications, from filtering records in a database to detecting patterns in text or images. On Arm Neoverse-based servers, SVE2 MATCH instructions unlock significant performance gains by vectorizing these operations. In this Learning Path, you’ll implement and benchmark both scalar and vectorized versions of search functions to see just how much faster workloads can run.

## What is SVE2 MATCH?

SVE2 (Scalable Vector Extension 2) is an extension to the Arm architecture that provides vector processing capabilities with a length-agnostic programming model. The MATCH instruction is a specialized SVE2 instruction that efficiently searches for elements in a vector that match any element in another vector.

## Set up your environment

To work through these examples, you need:

* An Arm-based cloud instance with SVE2 support or an Arm AGI CPU platform running Ubuntu 24.04
* GCC compiler with SVE support

Start by setting up your environment:

```bash
sudo apt-get update
sudo apt-get install -y build-essential gcc g++
```
An effective way to achieve optimal performance on Arm is not only through optimal flag usage, but also by using the most recent compiler version. This Learning path was tested with GCC 13 which is the default version on Ubuntu 24.04, but you can run it with newer versions of GCC as well.

Create a directory for your implementations:

```bash
mkdir -p sve2_match_demo
cd sve2_match_demo
```
## Understanding the problem

Your goal is to implement a function that searches for any occurrence of a set of keys in an array. The function should return true if any element in the array matches any of the keys, and false otherwise.

This type of search operation is common in many applications:

* **Database systems**: checking if a value exists in a column
* **Text processing**: finding specific characters in a text
* **Network packet inspection**: looking for specific byte patterns
* **Image processing**: finding specific pixel values

## Implementing search algorithms

To understand the alternatives, you can implement three versions of the search function:

### 1. Generic scalar implementation

Create a generic implementation in C that checks each element individually against each key. Open an editor of your choice and copy the code shown into a file named `sve2_match_demo.c`:

```c
#include <arm_sve.h>
#include <inttypes.h>
#include <stddef.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

int search_generic_u8(const uint8_t *hay, size_t n, const uint8_t *keys,
                      size_t nkeys) {
  for (size_t i = 0; i < n; ++i) {
    uint8_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
return 0;
}

int search_generic_u16(const uint16_t *hay, size_t n, const uint16_t *keys,
                       size_t nkeys) {
  for (size_t i = 0; i < n; ++i) {
    uint16_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
  return 0;
}
```

The `search_generic_u8()` and `search_generic_u16()` functions both return 1 immediately when a match is found in the inner loop.

### 2. SVE2 MATCH implementation

Now create an implementation that uses SVE2 MATCH instructions to process multiple elements in parallel. 

Copy the code shown into the same source file:

```c
int search_sve2_match_u8(const uint8_t *hay, size_t n, const uint8_t *keys,
                         size_t nkeys) {
  if (nkeys == 0) return 0;
  const size_t VL = svcntb();
  if (nkeys > VL) return search_generic_u8(hay, n, keys, nkeys);
  svbool_t pg = svptrue_b8();
  uint8_t tmp[256];
  for (size_t i = 0; i < VL; ++i) tmp[i] = keys[i % nkeys];
  svuint8_t keyvec = svld1(pg, tmp);
  size_t i = 0;
  for (; i + VL <= n; i += VL) {
    svuint8_t block = svld1(pg, &hay[i]);
    if (svptest_any(pg, svmatch_u8(pg, block, keyvec))) return 1;
  }
  for (; i < n; ++i) {
    uint8_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
  return 0;
}

int search_sve2_match_u16(const uint16_t *hay, size_t n, const uint16_t *keys,
                          size_t nkeys) {
  if (nkeys == 0) return 0;
  const size_t VL = svcnth();
  if (nkeys > VL) return search_generic_u16(hay, n, keys, nkeys);
  svbool_t pg = svptrue_b16();
  uint16_t tmp[128];
  for (size_t i = 0; i < VL; ++i) tmp[i] = keys[i % nkeys];
  svuint16_t keyvec = svld1(pg, tmp);
  size_t i = 0;
  for (; i + VL <= n; i += VL) {
    svuint16_t block = svld1(pg, &hay[i]);
    if (svptest_any(pg, svmatch_u16(pg, block, keyvec))) return 1;
  }
  for (; i < n; ++i) {
    uint16_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
  return 0;
}
```

The SVE MATCH implementation with the `search_sve2_match_u8()` and `search_sve2_match_u16()` functions provide an efficient vectorized search approach with these key technical aspects:
   - Uses SVE2's specialized MATCH instruction to compare multiple elements against multiple keys in parallel
   - Leverages hardware-specific vector length through `svcntb()` for scalability
   - In the case where the number of keys, `nkeys`, exceeds the hardware-specific vector length, the implementation falls back to the generic scalar version.
   - Prepares a vector of search keys that's compared against blocks of data
   - Processes data in vector-sized chunks with early termination when matches are found. Stops immediately when any element in the vector matches.
   - Falls back to scalar code for remainder elements

### 3. Optimized SVE2 MATCH implementation

In this next SVE2 implementation you will add loop unrolling and prefetching to further improve performance. 

Copy the code shown into the same source file:

```c
int search_sve2_match_u8_unrolled(const uint8_t *hay, size_t n, const uint8_t *keys,
                                 size_t nkeys) {
  if (nkeys == 0) return 0;
  const size_t VL = svcntb();
  if (nkeys > VL) return search_generic_u8(hay, n, keys, nkeys);

  svbool_t pg = svptrue_b8();
  
  // Prepare key vector
  uint8_t tmp[256] __attribute__((aligned(64)));
  for (size_t i = 0; i < VL; ++i) tmp[i] = keys[i % nkeys];
  svuint8_t keyvec = svld1(pg, tmp);
  
  size_t i = 0;
  // Process 4 vectors per iteration
  for (; i + 4*VL <= n; i += 4*VL) {
    // Prefetch data ahead
    __builtin_prefetch(&hay[i + 16*VL], 0, 0);
    
    svuint8_t block1 = svld1(pg, &hay[i]);
    svuint8_t block2 = svld1(pg, &hay[i + VL]);
    svuint8_t block3 = svld1(pg, &hay[i + 2*VL]);
    svuint8_t block4 = svld1(pg, &hay[i + 3*VL]);
    
    svbool_t match1 = svmatch_u8(pg, block1, keyvec);
    svbool_t match2 = svmatch_u8(pg, block2, keyvec);
    svbool_t match3 = svmatch_u8(pg, block3, keyvec);
    svbool_t match4 = svmatch_u8(pg, block4, keyvec);
    
    if (svptest_any(pg, match1) || svptest_any(pg, match2) || 
        svptest_any(pg, match3) || svptest_any(pg, match4))
      return 1;
  }
  
  // Process remaining vectors one at a time
  for (; i + VL <= n; i += VL) {
    svuint8_t block = svld1(pg, &hay[i]);
    if (svptest_any(pg, svmatch_u8(pg, block, keyvec))) return 1;
  }
  
  // Handle remainder
  for (; i < n; ++i) {
    uint8_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
  return 0;
}

// Optimized 16-bit version with unrolling
int search_sve2_match_u16_unrolled(const uint16_t *hay, size_t n, const uint16_t *keys,
                                  size_t nkeys) {
  if (nkeys == 0) return 0; 
  const size_t VL = svcnth();
  if (nkeys > VL) return search_generic_u16(hay, n, keys, nkeys);
  svbool_t pg = svptrue_b16();
    
  // Prepare key vector
  uint16_t tmp[128] __attribute__((aligned(64)));
  for (size_t i = 0; i < VL; ++i) tmp[i] = keys[i % nkeys];
  svuint16_t keyvec = svld1(pg, tmp);
    
  size_t i = 0;
  // Process 4 vectors per iteration
  for (; i + 4*VL <= n; i += 4*VL) {
    // Prefetch data ahead
    __builtin_prefetch(&hay[i + 16*VL], 0, 0);

    svuint16_t block1 = svld1(pg, &hay[i]);
    svuint16_t block2 = svld1(pg, &hay[i + VL]);
    svuint16_t block3 = svld1(pg, &hay[i + 2*VL]);
    svuint16_t block4 = svld1(pg, &hay[i + 3*VL]);

    svbool_t match1 = svmatch_u16(pg, block1, keyvec);
    svbool_t match2 = svmatch_u16(pg, block2, keyvec);
    svbool_t match3 = svmatch_u16(pg, block3, keyvec);
    svbool_t match4 = svmatch_u16(pg, block4, keyvec);

    if (svptest_any(pg, match1) || svptest_any(pg, match2) ||
        svptest_any(pg, match3) || svptest_any(pg, match4))
      return 1;
  }

  // Process remaining vectors one at a time
  for (; i + VL <= n; i += VL) {
    svuint16_t block = svld1(pg, &hay[i]);
    if (svptest_any(pg, svmatch_u16(pg, block, keyvec))) return 1;
  }

  // Handle remainder
  for (; i < n; ++i) {
    uint16_t v = hay[i];
    for (size_t k = 0; k < nkeys; ++k)
      if (v == keys[k]) return 1;
  }
  return 0;
}
```

The main highlights of this implementation are:
   - Processes four vectors per iteration instead of just one and stops immediately when any match is found in any of the four vectors.
   - Uses prefetching (`__builtin_prefetch`) to reduce memory latency
   - Leverages the `svmatch_u8` and `svmatch_u16` instructions to efficiently compare each element against multiple keys in a single operation
   - Aligns memory to 64-byte boundaries for better memory access performance

## Benchmarking framework

To compare the performance of the three implementations, use a benchmarking framework that measures the execution time of each implementation. You will also add helper functions for membership testing that are needed to setup the test data with controlled hit rates.

Copy the code below into the bottom of the same source code file:

```c
// Timing function
static inline uint64_t nsec_now(void) {
  struct timespec ts;
#if defined(CLOCK_MONOTONIC_RAW)
  clock_gettime(CLOCK_MONOTONIC_RAW, &ts);
#else
  clock_gettime(CLOCK_MONOTONIC, &ts);
#endif
  return (uint64_t)ts.tv_sec * 1000000000ULL + ts.tv_nsec;
}

// ---------------- helper: membership test for RNG fill ----------------------
static int key_in_set_u8(uint8_t v, const uint8_t *keys, size_t nkeys) {
  for (size_t k = 0; k < nkeys; ++k)
    if (v == keys[k]) return 1;
  return 0;
}
static int key_in_set_u16(uint16_t v, const uint16_t *keys, size_t nkeys) {
  for (size_t k = 0; k < nkeys; ++k)
    if (v == keys[k]) return 1;
  return 0;
}

// Fill such that P(match) ~= p.
static void fill_bytes_lowhit(uint8_t *buf, size_t n, const uint8_t *keys,
                              size_t nkeys, double p) {
  for (size_t i = 0; i < n; ++i) {
    if (drand48() < p) {
      buf[i] = keys[rand() % nkeys];
    } else {
      uint8_t v;
      do { v = (uint8_t)rand(); } while (key_in_set_u8(v, keys, nkeys));
      buf[i] = v;
    }
  }
}
static void fill_u16_lowhit(uint16_t *buf, size_t n, const uint16_t *keys,
                            size_t nkeys, double p) {
  for (size_t i = 0; i < n; ++i) {
    if (drand48() < p) {
      buf[i] = keys[rand() % nkeys];
    } else {
      uint16_t v;
      do { v = (uint16_t)rand(); } while (key_in_set_u16(v, keys, nkeys));
      buf[i] = v;
    }
  }
}

// Main benchmarking function
int main(int argc, char **argv) {
  const size_t len        = (argc > 1) ? strtoull(argv[1], NULL, 0) : (1ull << 26);
  const int    iterations = (argc > 2) ? atoi(argv[2])              : 5;
  const double hit_prob   = (argc > 3) ? atof(argv[3])              : 0.001;

  printf("Haystack length : %zu elements\n", len);
  printf("Iterations      : %d\n", iterations);
  printf("Hit probability : %.6f (%.4f %% )\n\n", hit_prob, hit_prob * 100.0);

  // Initialize data and run benchmarks...
  srand48(42);
  srand(42);
    
  // Align memory to 64-byte boundary for better performance
  uint8_t  *hay8  = aligned_alloc(64, len);
  uint16_t *hay16 = aligned_alloc(64, len * sizeof(uint16_t));
  
  const uint8_t keys8[]   = {0x13, 0x7F, 0xA5, 0xEE, 0x4C, 0x42, 0x01, 0x9B};
  const uint16_t keys16[] = {0x1234, 0x7F7F, 0xA5A5, 0xEEEE, 0x4C4C, 0x4242};
  const size_t NKEYS8  = sizeof(keys8)  / sizeof(keys8[0]);
  const size_t NKEYS16 = sizeof(keys16) / sizeof(keys16[0]);

  fill_bytes_lowhit(hay8,  len, keys8,  NKEYS8,  hit_prob); 
  fill_u16_lowhit(hay16, len, keys16, NKEYS16, hit_prob);

  uint64_t t_gen8 = 0, t_sve8 = 0, t_sve8_unrolled = 0;
  uint64_t t_gen16 = 0, t_sve16 = 0, t_sve16_unrolled = 0;
    
  for (int it = 0; it < iterations; ++it) {
    uint64_t t0;
      
    t0 = nsec_now();
    volatile int r = search_generic_u8(hay8, len, keys8, NKEYS8); (void)r;
    t_gen8 += nsec_now() - t0;
  
#if defined(__ARM_FEATURE_SVE2)
    t0 = nsec_now();
    r = search_sve2_match_u8(hay8, len, keys8, NKEYS8); (void)r;
    t_sve8 += nsec_now() - t0;

    t0 = nsec_now();
    r = search_sve2_match_u8_unrolled(hay8, len, keys8, NKEYS8); (void)r;
    t_sve8_unrolled += nsec_now() - t0;
#endif

    t0 = nsec_now();
    r = search_generic_u16(hay16, len, keys16, NKEYS16); (void)r;
    t_gen16 += nsec_now() - t0;

#if defined(__ARM_FEATURE_SVE2)
    t0 = nsec_now();
    r = search_sve2_match_u16(hay16, len, keys16, NKEYS16); (void)r;
    t_sve16 += nsec_now() - t0;

    t0 = nsec_now();
    r = search_sve2_match_u16_unrolled(hay16, len, keys16, NKEYS16); (void)r;
    t_sve16_unrolled += nsec_now() - t0;
#endif
  }
// ---------- latency results ----------
  printf("Average latency over %d iterations (ns):\n", iterations);
  printf("  generic_u8       : %.2f\n", (double)t_gen8 / iterations);
#if defined(__ARM_FEATURE_SVE2)
  printf("  sve2_u8          : %.2f\n", (double)t_sve8 / iterations);
  printf("  sve2_u8_unrolled : %.2f\n", (double)t_sve8_unrolled / iterations);
  printf("  speed‑up (orig)  : %.2fx\n", (double)t_gen8 / t_sve8);
  printf("  speed‑up (unroll): %.2fx\n\n", (double)t_gen8 / t_sve8_unrolled);
#else
  printf("  (SVE2 path not built)\n\n");
#endif
  printf("  generic_u16      : %.2f\n", (double)t_gen16 / iterations);
#if defined(__ARM_FEATURE_SVE2)
  printf("  sve2_u16         : %.2f\n", (double)t_sve16 / iterations);
  printf("  sve2_u16_unrolled: %.2f\n", (double)t_sve16_unrolled / iterations);
  printf("  speed‑up (orig)  : %.2fx\n", (double)t_gen16 / t_sve16);
  printf("  speed‑up (unroll): %.2fx\n", (double)t_gen16 / t_sve16_unrolled);
#else
  printf("  (SVE2 path not built)\n");
#endif
// ---------- throughput results ----------
  const double elems_total = (double)len * iterations;
  printf("\nThroughput (million items/second):\n");
  double tp_gen8  = elems_total / (t_gen8 / 1e9) / 1e6;
  printf("  generic_u8       : %.2f Mi/s\n", tp_gen8);
#if defined(__ARM_FEATURE_SVE2)
  double tp_sve8 = elems_total / (t_sve8 / 1e9) / 1e6;
  double tp_sve8_unrolled = elems_total / (t_sve8_unrolled / 1e9) / 1e6;
  printf("  sve2_u8          : %.2f Mi/s\n", tp_sve8);
  printf("  sve2_u8_unrolled : %.2f Mi/s\n", tp_sve8_unrolled);
  printf("  speed‑up (orig)  : %.2fx\n", tp_sve8 / tp_gen8);
  printf("  speed‑up (unroll): %.2fx\n\n", tp_sve8_unrolled / tp_gen8);
#else
  printf("  (SVE2 path not built)\n\n");
#endif
  double tp_gen16 = elems_total / (t_gen16 / 1e9) / 1e6;
  printf("  generic_u16      : %.2f Mi/s\n", tp_gen16);
#if defined(__ARM_FEATURE_SVE2)
  double tp_sve16 = elems_total / (t_sve16 / 1e9) / 1e6;
  double tp_sve16_unrolled = elems_total / (t_sve16_unrolled / 1e9) / 1e6;
  printf("  sve2_u16         : %.2f Mi/s\n", tp_sve16);
  printf("  sve2_u16_unrolled: %.2f Mi/s\n", tp_sve16_unrolled);
  printf("  speed‑up (orig)  : %.2fx\n", tp_sve16 / tp_gen16);
  printf("  speed‑up (unroll): %.2fx\n", tp_sve16_unrolled / tp_gen16);
#else
  printf("  (SVE2 path not built)\n");
#endif

  free(hay8);
  free(hay16);
  return 0;
}
```

## Compiling and Running

You can now compile a binary that is portable across Armv9-A systems with SVE2, while tuning it for a specific Neoverse version:

{{%notice Please Note%}}

If building a binary tuned for a Neoverse V3-based systems such as Arm AGI CPU or AWS Graviton 5, you will need `gcc` version 15 or greater to use the `-mtune=neoverse-v3` option.

{{%/notice%}}

{{< tabpane code=true >}}
{{< tab header="tune for Neoverse V2" >}}
gcc -O3 -march=armv9-a+sve2 -mtune=neoverse-v2 sve2_match_demo.c -o sve2_match_demo
{{< /tab >}}
{{< tab header="tune for Neoverse V3" >}}
gcc -O3 -march=armv9-a+sve2 -mtune=neoverse-v3 sve2_match_demo.c -o sve2_match_demo
{{< /tab >}}
{{< /tabpane >}}

Run the benchmark on a dataset of 65,536 elements (2^16) with a 0.001% hit rate for 10,000 iterations:

```bash
./sve2_match_demo $((1<<16)) 10000 0.00001
```

The output on the AGI CPU, using the Neoverse V3 build, is similar to:

```output
Haystack length : 65536 elements
Iterations      : 10000
Hit probability : 0.000010 (0.0010 % )

Average latency over 10000 iterations (ns):
  generic_u8       : 176900.54
  sve2_u8          : 1823.45
  sve2_u8_unrolled : 1490.96
  speed‑up (orig)  : 97.01x
  speed‑up (unroll): 118.65x

  generic_u16      : 35878.13
  sve2_u16         : 6356.75
  sve2_u16_unrolled: 5182.28
  speed‑up (orig)  : 5.64x
  speed‑up (unroll): 6.92x

Throughput (million items/second):
  generic_u8       : 370.47 Mi/s
  sve2_u8          : 35940.72 Mi/s
  sve2_u8_unrolled : 43955.64 Mi/s
  speed‑up (orig)  : 97.01x
  speed‑up (unroll): 118.65x

  generic_u16      : 1826.63 Mi/s
  sve2_u16         : 10309.68 Mi/s
  sve2_u16_unrolled: 12646.17 Mi/s
  speed‑up (orig)  : 5.64x
  speed‑up (unroll): 6.92x
```

You can experiment with different haystack lengths, iterations and hit probabilities.

```bash
./sve2_match_demo [length] [iterations] [hit_prob]
```

## Performance results on the AGI CPU

These measurements use a Neoverse V3-based AGI CPU running Ubuntu 24.04, with a 128-bit SVE vector length. The benchmark is compiled using `-O3 -march=armv9-a+sve2 -mtune=neoverse-v3`.

Each run uses 65,536 elements (2^16) and 10,000 iterations. For each hit probability, the latency tables report the median of the average latencies from five separate runs. Speedups are calculated from those median latencies. The example output is from one run, so its values differ from the medians. The runs use the host's existing settings without CPU pinning or host-wide tuning.

Performance on your system will vary with your hardware, compiler version, and build settings. Focus on the relative performance differences between the scalar, SVE2 MATCH, and unrolled implementations rather than the absolute latency values. The size of those relative differences can also vary across systems and compiler versions.

### Latency (ns per iteration) at different hit rates (8-bit)

| Implementation | 0% (no matches) | 0.001% | 0.01% | 0.1% | 1% |
|----------------|----------------|--------|-------|------|----|
| Generic scalar | 358,653.40 | 167,631.05 | 31,495.47 | 3,727.40 | 374.73 |
| SVE2 MATCH | 3,243.07 | 1,823.76 | 604.74 | 114.09 | 70.60 |
| SVE2 MATCH unrolled | 2,622.86 | 1,491.06 | 500.27 | 113.19 | 70.32 |

### Latency (ns per iteration) at different hit rates (16-bit)

| Implementation | 0% (no matches) | 0.001% | 0.01% | 0.1% | 1% |
|----------------|----------------|--------|-------|------|----|
| Generic scalar | 35,880.21 | 35,880.19 | 14,574.54 | 1,692.74 | 57.34 |
| SVE2 MATCH | 6,325.59 | 6,511.51 | 2,613.57 | 347.77 | 65.36 |
| SVE2 MATCH unrolled | 5,184.61 | 5,183.31 | 2,157.04 | 295.15 | 64.84 |

### Speedup versus generic scalar (8-bit)

| Hit rate | SVE2 MATCH | SVE2 MATCH unrolled |
|----------|------------|---------------------|
| 0% | 110.59x | 136.74x |
| 0.001% | 91.92x | 112.42x |
| 0.01% | 52.08x | 62.96x |
| 0.1% | 32.67x | 32.93x |
| 1% | 5.31x | 5.33x |

### Speedup versus generic scalar (16-bit)

| Hit rate | SVE2 MATCH | SVE2 MATCH unrolled |
|----------|------------|---------------------|
| 0% | 5.67x | 6.92x |
| 0.001% | 5.51x | 6.92x |
| 0.01% | 5.58x | 6.76x |
| 0.1% | 4.87x | 5.74x |
| 1% | 0.88x | 0.88x |

### Impact of hit rate on performance

For 8-bit data, SVE2 MATCH provides its largest speedups when matches are rare. At 0%–0.001% hit probabilities, the two vectorized implementations are about 92–137x faster than scalar. This falls to about 52–63x at 0.01%, 33x at 0.1%, and 5.3x at 1%.

For 16-bit data, the speedup is about 5.5–6.9x at 0%–0.001%, 5.6–6.8x at 0.01%, and 4.9–5.7x at 0.1%. At 1%, both SVE2 implementations are slower than scalar: their speedup is 0.88x, corresponding to about 13–14% higher latency. Frequent matches shorten the scan, reducing the benefit of vector processing relative to setup and timing overhead.

The hit probabilities describe data generation, not guaranteed match counts. With the fixed random seeds, the 16-bit array contains no matches at both 0% and 0.001%, so both cases scan the full array. The small latency differences between these cases reflect run variation. The basic 16-bit SVE2 implementation also varies more across repeated runs at these probabilities than the unrolled implementation.

Each run repeatedly searches the same generated arrays and stops at the first match. The printed throughput divides the full array length by elapsed time, even when the search terminates early. For nonzero hit probabilities, interpret it as a nominal array-length rate, not the number of elements actually inspected per second. Use latency to compare these early-exit searches.

### Benefits of loop unrolling

Unrolling improves the median latency for both element sizes from 0% through 0.1% hit probability, although the 8-bit gain at 0.1% is less than 1%. In the no-match case, unrolling increases the speedup from 110.59x to 136.74x for 8-bit data and from 5.67x to 6.92x for 16-bit data.

At 1%, the basic and unrolled versions differ by less than 1 ns for both element sizes. Treat them as effectively tied at this measurement scale; both 16-bit versions remain slower than scalar. These results support using unrolling for longer scans, but do not establish a benefit for every hit rate or isolate the contribution of prefetching.

### Applications of SVE2 MATCH

The SVE2 MATCH instruction can be applied to various real-world scenarios such as:

**Database Systems**

In database systems, MATCH can accelerate:
- String pattern matching in text columns
- Value existence checks in arrays
- Filtering operations in columnar databases

**Text Processing**

For text processing applications, MATCH can speed up:
- Character set membership tests
- Word boundary detection
- Special character identification

**Network Packet Inspection**

In network applications, MATCH can improve:
- Protocol header inspection
- Pattern matching in packet payloads
- Signature-based intrusion detection

**Image Processing**

For image processing, MATCH can accelerate:
- Color palette lookups
- Pixel value classification
- Image mask operations

## Conclusion

The SVE2 MATCH instruction provides a powerful way to accelerate search operations in byte and half word arrays. By implementing these optimizations on cloud instances with SVE2, you can achieve significant performance improvements for your applications.
