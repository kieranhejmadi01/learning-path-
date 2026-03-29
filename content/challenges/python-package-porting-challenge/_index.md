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
community_attempts:
  - link: "https://github.com/arm/mcp"
    title: "Arm Model Context Protocol (MCP) Server"
    author: "Arm Limited and Contributors"
    summary: "An MCP server providing AI assistants with tools and knowledge for Arm architecture development, migration, and optimization."
challenge: true
multi_challenge: false
multichallenge_part: false
layout: challengeall
---

## Overview

Windows on Arm brings Arm performance and efficiency to more desktop systems, but developers still hit package ecosystem gaps. Many widely used Python packages do not ship `win_arm64` wheels, which forces users to rebuild from source and often exposes architecture-specific issues.

In this challenge, you help improve the Windows on Arm Python package ecosystem by validating packages, fixing compatibility issues, and upstreaming support for Arm-native binaries.
