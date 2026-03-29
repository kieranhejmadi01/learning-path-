---
title: Challenge brief
weight: 2
layout: challengeall
---

## The challenge

Create an intelligent automation tool that supports package porting for domains such as bioinformatics pipelines with Nextflow or statistics workflows with R.

The outcome should go beyond simple shell scripting. Your solution should use dependency graph analysis and automation to identify unported packages, recommend or generate fixes, evaluate build results, and help contributors upstream support.

## Suggested scope

Aim to support tasks such as:

- Identifying packages that lack native Arm support
- Tracing recursive dependency failures
- Recommending or auto-generating build recipes
- Retrying builds intelligently after failures
- Generating pull requests when the tool is confident in a fix
- Suggesting techniques such as SSE2NEON when packages rely on x86 intrinsics

## Skills you can practice

- Package ecosystem analysis
- Dependency graph reasoning
- Build automation and CI design
- Porting workflows for Arm systems

## Assessment criteria

Submissions are assessed using broad criteria that reflect technical depth, practical impact, and value to the Arm software ecosystem.

#### Use of Arm technology

Your submission should show a clear benefit for Arm-based platforms. Strong entries demonstrate how the tool improves package porting, validation, or upstream contribution workflows for Arm systems rather than offering a generic automation script.

#### Impact on package porting

Assessors look for solutions that help solve meaningful bottlenecks in real package ecosystems. Strong submissions identify useful targets, improve developer efficiency, or make it easier to move packages from unsupported to working on Arm.

#### Code quality and robustness

Your implementation should be maintainable, reliable, and well structured. Strong entries handle dependency failures clearly, provide sensible automation boundaries, and produce results that developers can trust.

#### Documentation and reproducibility

Your project should be straightforward to evaluate and reuse. Clear documentation, reproducible workflows, and concise evidence of successful package analysis or porting help reviewers validate the submission efficiently.
