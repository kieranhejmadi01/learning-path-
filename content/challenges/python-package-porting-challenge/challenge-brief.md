---
title: Challenge brief
weight: 2
layout: challengeall
---

## The challenge

Advance the Windows on Arm Python ecosystem by validating and optimizing third-party packages so they build and run successfully on `win_arm64`.

## Suggested scope

Aim to complete work such as:

- Turn at least five amber projects green on [Windows Arm64 Wheels](https://tonybaloney.github.io/windows-arm64-wheels/)
- Identify packages without `win_arm64` wheels
- Diagnose porting bugs such as x86-specific intrinsics or unsupported toolchain assumptions
- Create patches that allow packages to build and run correctly
- Collaborate with maintainers to upstream your fixes

## Skills you can practice

- Python packaging
- Continuous integration testing
- Porting native extensions to Arm
- Collaboration with upstream open source maintainers

## Assessment criteria

Submissions are assessed using broad criteria that reflect ecosystem impact, technical quality, and effectiveness on Windows on Arm.

#### Use of Arm technology

Your submission should make a clear contribution to the Windows on Arm Python ecosystem. Strong entries demonstrate that packages build, install, or run correctly on `win_arm64`, with fixes that are directly relevant to Arm users and maintainers.

#### Ecosystem impact

Assessors look for work that improves real packages and benefits the broader community. Strong submissions unblock multiple packages, turn visible ecosystem gaps into working solutions, or help move fixes upstream.

#### Code quality and maintainability

Your changes should be clear, reliable, and appropriate for long-term use. Strong entries include well-scoped fixes, sensible CI or packaging updates, and good evidence that the package behaves correctly after the changes.

#### Documentation and reproducibility

Your project should make the porting work easy to review and repeat. Clear notes on the issue, the fix, the validation steps, and any upstream coordination help reviewers understand both the problem and the outcome.
