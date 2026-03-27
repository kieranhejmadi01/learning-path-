---
title: "Build video and audio provenance workflows on Arm"
minutes_to_complete: 30240
author:
    - Arm Developer Program
weight: 1
subjects: Security
skilllevels: Advanced
tools_software_languages:
    - Python
    - C2PA
    - PyTorch
operatingsystems:
    - Linux
    - macOS
    - Windows
challenge: true
multi_challenge: false
multichallenge_part: false
layout: challengeall
---

## Overview

Generative AI has made it much easier to manipulate images, video, and audio, which creates a trust gap for viewers and organizations. Provenance frameworks such as C2PA help address this by attaching signed, machine-verifiable metadata that explains how media was created, edited, or analyzed.

In this challenge, you build an AI-augmented audio or video capture and provenance system on an Arm-powered device. Your solution should combine on-device inference with transparent provenance metadata so others can inspect how media was processed.
