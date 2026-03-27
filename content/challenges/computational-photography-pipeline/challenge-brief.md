---
title: Challenge brief
weight: 2
layout: challengeall
---

## Define the challenge

Design and implement one of the following computational photography pipelines, or a closely related variant, on an Arm-based system:

- Night-mode imaging to improve low-light signal-to-noise ratio
- Portrait-mode imaging with realistic bokeh blur
- Neural ray denoising for rendering-related image data
- Generative AI pipelines such as style transfer or diffusion-based enhancement

## Suggested scope

Your project should aim to optimize both inference latency and perceived image quality. The technical criteria include:

- Use a quantized neural network, preferably below 16-bit precision
- Preferably use the ExecuTorch runtime with Arm Neural Technology and the ML extensions for Vulkan backend
- Run on an Arm-based CPU with SIMD support such as SVE or SME, or on a Neural Graphics capable GPU

## Skills you can practice

- Computational photography design
- Neural inference optimization
- SIMD and GPU-aware implementation on Arm
- Image quality evaluation
