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

## Assessment criteria

Submissions are assessed using broad criteria that balance image quality, technical execution, and effective use of Arm platforms.

#### Use of Arm technology

Your submission should make meaningful use of Arm-based compute features such as SIMD, SME, SVE, Vulkan acceleration, or Arm-oriented runtimes. Strong entries explain how the implementation takes advantage of Arm hardware rather than treating Arm as only a target device.

#### Image quality and technical performance

Assessors look for a convincing balance between visual quality and runtime efficiency. Strong submissions demonstrate thoughtful trade-offs in latency, memory use, precision, and output quality for the selected photography task.

#### Novelty and design choices

Your project should show considered technical design, whether through model architecture, pipeline structure, optimization strategy, or evaluation method. Strong entries present a distinctive approach or a clearly justified improvement over a simpler baseline.

#### Documentation and reproducibility

Your work should be clear enough for others to understand and test. Strong submissions provide setup instructions, model details, evaluation steps, and enough context for reviewers to reproduce both performance and quality claims.
