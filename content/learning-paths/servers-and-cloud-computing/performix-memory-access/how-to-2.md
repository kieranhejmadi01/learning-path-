---
title: Inspect with Performix
weight: 4

### FIXED, DO NOT MODIFY
layout: learningpathall
---

## Inspect the particle data structure

Start by inspecting the baseline particle model in `src/baseline/particle.hpp`.

This is a naive baseline implementation intended for learning and comparison:

```cpp
struct Particle {
    float x, y, z;                   // position      (12 bytes) 
    float vx, vy, vz;                // velocity      (12 bytes)
    float mass, charge, temperature; // properties    (12 bytes) 
    float pressure, energy, density; //               (12 bytes) 
    float spin_x, spin_y, spin_z;    //               (12 bytes)
    float pad;                       // padding        (4 bytes)
};
```

The ownership container in the same file is:

```cpp
class ParticleOwner {
    // Stores particle references used by the simulation.
    std::vector<Particle*> particles_;
};
```

The update loop in `src/baseline/baseline.cpp` then runs position updates:

```cpp
for (int iter = 0; iter < iters; ++iter) {
    update_positions(particles.data(), NUM_PARTICLES, dt);
}
```

This baseline design can create avoidable memory overhead:

- Pointer chasing can hurt locality compared to contiguous storage.
- The hot loop only needs a subset of particle fields, so full-struct loads can waste bandwidth.

Before you optimize anything, profile and measure.

## Run the Performix Memory Access Recipe

Open the Performix GUI on your local machine.

![](./setup.png)
![](./performix_before_optimizations.png)
![](./codex_prompt.png)