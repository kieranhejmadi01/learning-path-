---
title: Inference Code
description: Generate a C++ scaffold and use the OneFormer implementation as a reference for your model integration.
weight: 5
layout: learningpathall
---

With the model package prepared, connect it to Compute Flow through a C++ brick. The class loads
the runners, connects preprocessing and inference to postprocessing, and returns the Brick API's outputs.

## Generate the scaffold

Generate the C++ boilerplate in a separate directory. Replace the brick type and class name when
adapting this command to your model.

```bash
AUTHOR_ROOT="$WORK/authored-bricks"
python "$CF/scripts/create_brick_template.py" image_segmentation OneFormer \
  --output-root "$AUTHOR_ROOT"
```

The template creates a header, implementation, metadata, and README under
`image_segmentation/one_former`. Keep the API's input and output contract when connecting your
model.

## Use the completed implementation as a reference

The class in `$BRICK_REPO/image_segmentation/one-former/OneFormer.h` inherits the lifecycle from `BrickBaseV1`.
Its declarations identify the API, provide the lifecycle hooks, and store the three runners:

```cpp
class OneFormer : public BrickBaseV1 {
public:
    std::string_view api() const noexcept override { return "image_segmentation.v1"; }
    ValidationResult validate(const ComputeFlowManifest& manifest) const override;
    bool is_generation_complete() const override;

protected:
    void load_models_impl(const ComputeFlowManifest& manifest, ProfilerView profiler) override;
    void prepare_impl(const ComputeFlowManifest& manifest, ProfilerView profiler) override;
    void warmup_impl(ProfilerView profiler) override;
    void update_config_impl(const ComputeFlowManifest& manifest, ProfilerView profiler) override;
    TensorDict run_impl(const TensorDict& input, ProfilerView profiler) override;
    void reset_states_impl(ProfilerView profiler) override;

private:
    std::unique_ptr<IRunner> _preprocess;
    std::unique_ptr<IRunner> _segmenter;
    std::unique_ptr<IRunner> _postprocess;
    MemoryPlanner _memory;
    int64_t _maximum_dimension = 0;
    int64_t _num_classes       = 0;
    bool _complete             = false;
};
```

| Method                                               | Responsibility                            |
| ---------------------------------------------------- | ----------------------------------------- |
| `api()` and `validate()`                             | Identify the API and check the manifest   |
| `load_models_impl()`                                 | Create and set up runners                 |
| `prepare_impl()`                                     | Load assets and allocate reusable buffers |
| `run_impl()`                                         | Connect runners and return API outputs    |
| `warmup_impl()`                                      | Exercise the inference path               |
| `update_config_impl()`                               | Apply supported configuration changes     |
| `reset_states_impl()` and `is_generation_complete()` | Reset requests and report completion      |

In `$BRICK_REPO/image_segmentation/one-former/OneFormer.cpp`, `load_models_impl()` creates each runner from the manifest
and passes the owning brick's identity when setting it up:

```cpp
void OneFormer::load_models_impl(const ComputeFlowManifest& manifest, ProfilerView profiler) {
    _preprocess = make_runner(manifest["preprocess"].framework_type);
    _preprocess->setup(manifest["preprocess"], manifest.get_brick_info(), profiler);
    _segmenter = make_runner(manifest["segmenter"].framework_type);
    _segmenter->setup(manifest["segmenter"], manifest.get_brick_info(), profiler);
    _postprocess = make_runner(manifest["postprocess"].framework_type);
    _postprocess->setup(manifest["postprocess"], manifest.get_brick_info(), profiler);
}
```

Inside `run_impl()`, connect the three stages. This excerpt uses `image`, the input tensor
with a batch dimension added earlier in the method:

```cpp
_preprocess->set_input("image", image);
_preprocess->run();
_segmenter->set_input("image", _preprocess->get_output("image"));
_segmenter->run();
_postprocess->set_input("logits", _segmenter->get_output("logits"));
_postprocess->set_input("image", image);
_postprocess->run();
```

## Add checks with helper macros

Use the macros in `compute_flow/check.h` to report failures without writing the error handling
each time. In `validate()`, `COMPUTE_FLOW_VALIDATE` returns a failed `ValidationResult` if a condition
is false:

```cpp
COMPUTE_FLOW_VALIDATE(manifest.has_module("segmenter"));
COMPUTE_FLOW_VALIDATE(manifest["segmenter"].framework_type == RunnerType::ONNXRT);
```

In `run_impl()`, use `COMPUTE_FLOW_CHECK` for runtime conditions; it throws an exception on failure:

```cpp
const auto& source = input["image"];
COMPUTE_FLOW_CHECK(source.data != nullptr);
COMPUTE_FLOW_CHECK(source.format.dt == DataType::F32);
COMPUTE_FLOW_CHECK(source.format.shape.size() == 3);
```

Choose checks that enforce your model's requirements and the Brick API contract.

## Connect metadata and tests

The model manifest defines execution. Under `image_segmentation/one-former`, `info.json` declares
model sources and checksums, and `test/test_cases.yaml` defines CLI smoke tests. The repository's
root `registry.txt` lists this brick directory. Compute Flow generates build and binding glue.

Keep your scaffold for experimentation. Next, run the completed GitHub implementation without
needing to finish your own brick first.
