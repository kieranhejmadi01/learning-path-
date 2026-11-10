---
title: Author a Compute Flow brick variant
description: Implement a reusable Compute Flow brick variant for image segmentation and validate its model outputs and runtime compatibility.

minutes_to_complete: 45
who_is_this_for: Developers who want to integrate a model into a reusable Compute Flow brick variant and share it across applications and with other developers.
learning_objectives:
    - Understand steps required to author a custom brick variant.
    - Export and convert an image segmentation model for compute flow compatibility.
    - Implement inference code and run the custom brick. 
prerequisites:
    - Access to an Apple® silicon Mac, Linux® arm64 system, or Android® mobile phone.
    - Basic understand and experience with compute flow.
    - Basic C++ and Python knowledge and familiarity with terminal commands.
author: Kieran Hejmadi
generate_summary_faq: true
rerun_summary: false
rerun_faqs: false

skilllevels: Advanced
subjects: ML
armips: []
tools_software_languages:
    - Compute Flow
    - C++
    - Python
    - ONNX Runtime
operatingsystems:
    - macOS
    - Linux
    - Android

### Cross-platform metadata only
shared_path: true
shared_between:
    - laptops-and-desktops
    - mobile-graphics-and-gaming
    - servers-and-cloud-computing

further_reading:
    - resource:
        title: Compute Flow repository
        link: https://github.com/ARM-software/compute-flow
        type: documentation
    - resource:
        title: OneFormer brick implementation
        link: https://github.com/kieranhejmadi01/OneFormer-Brick
        type: documentation
    - resource:
        title: Define a Compute Flow brick API
        link: https://github.com/ARM-software/compute-flow/blob/main/docs/design/brick_api.md
        type: documentation
    - resource:
        title: Brick runtime interface
        link: https://github.com/ARM-software/compute-flow/blob/main/docs/design/brick_runtime_interface_v1.md
        type: documentation

### FIXED, DO NOT MODIFY
weight: 1
layout: "learningpathall"
learning_path_main_page: "yes"
---
