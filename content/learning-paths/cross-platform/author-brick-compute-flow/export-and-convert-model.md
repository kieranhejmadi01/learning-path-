---
title: Model package
description: Capture source-model outputs, explore an ONNX export, and inspect a prepared Compute Flow package.
weight: 4
layout: learningpathall
---

## Capture reference predictions in PyTorch

Before exporting, save the original model's predictions so you can check whether conversion changes
the class assigned to each pixel. `capture_baseline.py` is an example PyTorch script that runs
OneFormer on four sample images without using Compute Flow.

```bash
"$EXPORT_PYTHON" "$HF_REPO/tools/capture_baseline.py" --work-dir "$WORK"
```

After the weights load, the output is similar to:

```output
{"image": "astronaut", "public_shape": [3, 512, 512], "network_shape": [1, 3, 512, 512], "seconds": 1.0995803340010752}
{"image": "coffee_landscape", "public_shape": [3, 400, 600], "network_shape": [1, 3, 512, 768], "seconds": 0.8510422090002976}
{"image": "chelsea_odd", "public_shape": [3, 301, 451], "network_shape": [1, 3, 512, 767], "seconds": 0.698306500000399}
{"image": "astronaut_portrait", "public_shape": [3, 509, 311], "network_shape": [1, 3, 837, 512], "seconds": 0.7756780829986383}
```

`public_shape` describes the input image in CHW order; `network_shape` shows the batched, resized
input passed to the model. Timings vary by machine.

The script saves predictions in `$WORK/baseline/*.npz` and the summary in `baseline.json`. For your
own model, capture representative inputs and reference outputs using its original inference code.

This baseline checks the original model's behavior, not Brick API compatibility. Compare its inputs
and outputs with your chosen API and note any differences in tensor names, shapes, data types, or
meaning. You'll address these when implementing the brick.

## Choose a model runner

Compute Flow supports multiple runners, including ONNX Runtime, ExecuTorch, Llama.cpp and LiteRT; availability
depends on the target and build. Consider runners already used by other models in your application:
reusing one can reduce runtime dependencies, provided it supports your model's operators, shapes,
and target hardware. You can mix runners when needed.

Running the model in PyTorch establishes its reference behavior; it doesn't confirm that an exported
version works through a Compute Flow runner. Here, ONNX is the exported model format, and ONNX
Runtime is the runner that executes it.

This example uses ONNX Runtime, for which the adapted OneFormer export passed the shape and
prediction comparisons.

## Prepare OneFormer for ONNX Runtime

Compute Flow does not currently provide a PyTorch model runner. Export the model to a format supported by your chosen runner before implementing inference code.

For this example, use ONNX Runtime. The OneFormer-specific preparation scripts export the model to ONNX, compare predictions with the PyTorch baseline, and assemble the model package.

Optionally, reproduce the package:

```bash
"$EXPORT_PYTHON" "$HF_REPO/tools/prepare_model.py" \
  --work-dir "$WORK" --compute-flow "$CF"
```

The script writes the package to `$WORK/package`. It requires at least 99% pixel-label agreement with the original model for each test image.

The Hugging Face repository's `tools/` directory is reference code for this model. OneFormer needed shape and padding adaptations; your model may need different changes or none. Compare exported outputs with the original model, then test through Compute Flow's runner.

Continue with the downloaded package if you skip reproduction.

## Inspect the prepared package

Continue with the downloaded package whether or not you ran the optional export. Its `oneformer_manifest.json` is a [brick manifest](https://github.com/ARM-software/compute-flow/blob/main/docs/design/brick_manifest.md): it describes the resources the OneFormer implementation needs to load and run the model package.

It identifies the brick implementation and declares three ONNX graphs, their runners and tensor formats, the class-label file, and constants. The inference code reads these entries to configure its runners and connect preprocessing, segmentation, and postprocessing. Keeping these details in the manifest lets the implementation load a compatible package without hard-coding its model paths and settings.

The Brick API defines the public inputs and outputs. This manifest describes the implementation's model resources and internal tensors:

```json
{
    "brick": {
        "type": "image_segmentation",
        "name": "OneFormer",
        "version": "1.0"
    },
    "models": {
        "preprocess": {
            "runner": "onnxrt",
            "filename": "preprocess.onnx",
            "inputs": {
                "image": {
                    "index": 0,
                    "dtype": "float32",
                    "shape": [
                        1,
                        3,
                        -1,
                        -1
                    ]
                }
            },
            "outputs": {
                "image": {
                    "index": 0,
                    "dtype": "float32",
                    "shape": [
                        1,
                        3,
                        -1,
                        -1
                    ]
                }
            },
            "runner_params": {}
        },
        "segmenter": {
            "runner": "onnxrt",
            "filename": "oneformer.onnx",
            "inputs": {
                "image": {
                    "index": 0,
                    "dtype": "float32",
                    "shape": [
                        1,
                        3,
                        -1,
                        -1
                    ]
                }
            },
            "outputs": {
                "logits": {
                    "index": 0,
                    "dtype": "float32",
                    "shape": [
                        1,
                        150,
                        -1,
                        -1
                    ]
                }
            },
            "runner_params": {}
        },
        "postprocess": {
            "runner": "onnxrt",
            "filename": "postprocess.onnx",
            "inputs": {
                "logits": {
                    "index": 0,
                    "dtype": "float32",
                    "shape": [
                        1,
                        150,
                        -1,
                        -1
                    ]
                },
                "image": {
                    "index": 1,
                    "dtype": "float32",
                    "shape": [
                        1,
                        3,
                        -1,
                        -1
                    ]
                }
            },
            "outputs": {
                "mask": {
                    "index": 0,
                    "dtype": "int32",
                    "shape": [
                        -1,
                        -1
                    ]
                }
            },
            "runner_params": {}
        }
    },
    "assets": {
        "class_names": "assets/labels.txt"
    },
    "constants": {
        "maximum_dimension": {
            "type": "int64",
            "value": 1024
        },
        "num_classes": {
            "type": "int64",
            "value": 150
        }
    }
}
```

Use Compute Flow's `format_manifest.py --check` to check manifest formatting. This does not validate runtime compatibility or inference results.

```bash
python "$CF/scripts/format_manifest.py" --check "$HF_REPO/oneformer_manifest.json" \
  && echo "Manifest formatting check passed."
```

The script is silent on success; the added `echo` confirms that it passed:

```output
Manifest formatting check passed.
```

If formatting differs, it prints `<path>: manifest is not formatted` and exits with status `1`.
Check the downloaded files against their recorded checksums separately:

```bash
(cd "$HF_REPO" && shasum -a 256 -c SHA256SUMS)
```

The Hugging Face repository's [`tools/` directory](https://huggingface.co/Kieranhejmadi-arm/oneformer-swin-tiny-ade20k-fp32/tree/23f596de22c93e831da6f503891dafc37e04d10c/tools) is reference code specific to this OneFormer checkpoint and export workflow. Your model may need different scripts or adaptations.

## What you've learned

You can choose a supported runner, export a model, and compare its predictions with the original. You understand how the manifest describes the model package and how formatting and checksum checks help verify it. Export adaptations are specific to the model and runner.

Next, use the prepared OneFormer package to implement the brick. You'll then check its outputs and lifecycle through Compute Flow and exercise the existing C++ and Python CLIs.
