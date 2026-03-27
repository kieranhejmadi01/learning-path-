---
title: Challenge brief
weight: 2
layout: challengeall
---

## Define the challenge

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
