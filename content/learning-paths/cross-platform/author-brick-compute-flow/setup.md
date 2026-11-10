---
title: Setup
description: Install Compute Flow and download the original OneFormer checkpoint and prepared model package.
weight: 3
layout: learningpathall
---

## Install the host tools

These commands assume you are natively building the brick on MacOS. The commands are broadly similar if using Linux and may require minor modification and have not been tested. Install the Xcode Command Line Tools and [Homebrew](https://brew.sh/) if they are not already available on your system. 

```bash
brew install git git-lfs uv
git lfs install
```

## Create your workspace

From a directory of your choice, create a workspace for the repositories and generated files.

```bash
mkdir -p oneformer-example
cd oneformer-example

export CF="$PWD/compute-flow"
export BRICK_REPO="$PWD/OneFormer-Brick"
export HF_REPO="$PWD/OneFormer-HF"
export WORK="$PWD/work"
export COMPUTE_FLOW_HOME="$WORK/state"

mkdir -p "$WORK/output"
```

## Install Compute Flow

Clone the SDK's into the workspace and create its virtual environment:

```bash
git clone https://github.com/ARM-software/compute-flow.git "$CF"
cd "$CF"
uv sync
source .venv/bin/activate
```

Check the required tools:

```bash
compute-flow check
```

Install any missing dependencies reported, then run the check again.

## Download the reference source and models

Clone these repositories for the reference C++ brick implementation on GitHub and the prepared model, export scripts, and validation tools on Hugging Face.

{{% notice Note %}} The example GitHub brick and prepared Hugging Face package currently need access
permission. Configure GitHub Git credentials and run `hf auth login --add-to-git-credential` before
cloning them; these authentication steps won't be needed for downloads once the repositories are
public. The SHI-Labs checkpoint is already public. {{% /notice %}}

```bash
git clone https://github.com/kieranhejmadi01/OneFormer-Brick.git "$BRICK_REPO"
git clone https://huggingface.co/Kieranhejmadi-arm/oneformer-swin-tiny-ade20k-fp32 "$HF_REPO"
git -C "$HF_REPO" lfs pull
```

Download the original checkpoint into the Hugging Face cache used by the example tools:

```bash
export SOURCE_MODEL="shi-labs/oneformer_ade20k_swin_tiny"
export SOURCE_REVISION="05f2812b1eccf9909b3897777450f8d68148cafc"

hf download "$SOURCE_MODEL" --revision "$SOURCE_REVISION"
```

The SHI-Labs checkpoint provides the original model. 

## Prepare the export environment

Create a separate environment for source-model testing and export. Keep the Compute Flow environment
activated for SDK commands.

```bash
export EXPORT_PYTHON="$WORK/export-venv/bin/python"
uv venv "$WORK/export-venv" --python 3.12
uv pip install --python "$EXPORT_PYTHON" \
  -r "$HF_REPO/tools/requirements-export.txt"
```
