---
title: Get started
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

## Prepare your environment

Start on an Arm Linux system (aarch64), such as an AWS Graviton instance running Amazon Linux 2023. This learning path has been tested on an AWS Graviton 3 `c7g.metal` instance, which has 64 Neoverse V1 cores. 

Install the following packages, replacing the `dnf` package manager with the one for your linux distribution.

```bash
sudo dnf update -y
sudo dnf install -y git cmake gcc-c++ environment-modules python3 python3-pip
```

Next, [install Arm Perfomix](https://learn.arm.com/install-guides/performix/) on the remote Arm Linux target and the graphic user interface on your local machine - there is no need to use the command line interface (CLI). 

Install the prebuilt Arm Performance Libraries on your Arm Linux system using our [install guide](https://learn.arm.com/install-guides/armpl/). Follow the instructions and load the module file. Loading a module file configures your environment by setting paths and variables so the correct software and libraries are available, allowing programs to run with the dependencies they require. You can check you have successfully loaded the module file with the following command.  

```bash
module list
```

You should see the output below. If will need to load the environment module with the `module load <arm-performance-lib>` command.

```output
Currently Loaded Modulefiles:
 1) arm-performance-libraries
```

{{% notice Please Note %}}
When you open a new terminal, you will need to reload the modulefile. To simplify this, you can add the modulefile path to your `~/.bashrc` so modules can be loaded more easily.
```bash
echo "export MODULEPATH=$MODULEPATH:/opt/arm/modulefiles" >> ~/.bashrc
```
{{% /notice %}}

Clone and build the example project:

```bash
git clone https://github.com/arm-education/Data-Processing-Example.git
cd Data-Processing-Example
```

In the next section, you examine what the data-processing example does and run the visualization helper.
