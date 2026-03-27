---
title: "Port Python packages to Windows on Arm"
minutes_to_complete: 30240
author:
    - Arm Developer Program
weight: 1
subjects: Libraries
skilllevels: Advanced
tools_software_languages:
    - Python
    - CI
    - C++
operatingsystems:
    - Windows
challenge: true
multi_challenge: false
multichallenge_part: false
layout: challengeall
---

## Overview

Windows on Arm brings Arm performance and efficiency to more desktop systems, but developers still hit package ecosystem gaps. Many widely used Python packages do not ship `win_arm64` wheels, which forces users to rebuild from source and often exposes architecture-specific issues.

In this challenge, you help improve the Windows on Arm Python package ecosystem by validating packages, fixing compatibility issues, and upstreaming support for Arm-native binaries.
