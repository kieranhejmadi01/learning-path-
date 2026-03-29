---
title: Challenge brief
weight: 2
layout: challengeall
---

## Define the challenge

Train a model using a non-restrictively licensed dataset, apply quantization-aware training, and deploy the result to an Arm-powered mobile target such as an Android phone or emulator.

## Suggested scope

Consider real-time applications such as:

- Sign language recognition for accessibility
- Visual anomaly detection in manufacturing
- Personal health and activity monitoring from camera feeds

The final quantized model should be suitable for sharing with the Hugging Face community and, where relevant, for inclusion in the Arm space on Hugging Face.

## Skills you can practice

- Quantization-aware training
- Model deployment on Arm mobile devices
- Android integration for AI inference
- Reproducible model publishing

## Assessment criteria

Submissions are assessed using broad criteria that balance model quality, deployability, and value for Arm-powered mobile systems.

#### Use of Arm technology

Your submission should show that the trained and quantized model runs successfully on an Arm-powered mobile target. Strong entries demonstrate that deployment choices are appropriate for real Arm mobile devices rather than only desktop evaluation.

#### Model quality and suitability

Assessors look for models that are both useful and well matched to the chosen task. Strong submissions show that quantization-aware training preserves practical quality while keeping the model efficient enough for mobile use.

#### Engineering quality and robustness

Your implementation should be reliable and well organized across training, export, and deployment. Strong entries show clear workflows, sensible validation steps, and robust handling of model conversion or runtime issues.

#### Documentation and reproducibility

Your work should be easy to understand, run, and evaluate. Strong submissions provide dataset information, training settings, deployment steps, and enough detail for reviewers to reproduce the quantized model and mobile inference flow.
