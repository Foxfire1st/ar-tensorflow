# Cross-Repo Boundary Pack — lite

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/lite/` |
| generated | 2026-05-17T17:00 |
| topology | external |
| status | no-boundary-found |

## Allowed Adjacent Repos

| Adjacent Repo | Expected Branch | Actual Branch | Status | Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Boundary Summary

No cross-repo boundaries configured. TFLite uses FlatBuffers (vendored in `third_party/`) and has delegate backends (GPU, NNAPI, XNNPACK) that are Android/iOS platform APIs or vendored implementations.

## Vendored Dependencies

| Dependency | Location | Role | Interface |
|---|---|---|---|
| FlatBuffers | `third_party/flatbuffers/` | `.tflite` model serialization/deserialization | `flatbuffers::Verifier`, `flatbuffers::GetRoot<>()` |
| XNNPACK | `third_party/xnnpack/` | CPU-optimized delegate | `xnn_create_runtime`, `xnn_run_operator` |
| FP16 | `third_party/FP16/` | Half-precision float support | FP16 conversion functions |

## Platform Delegates (External, not vendored)

| Delegate | Platform | Interface |
|---|---|---|
| GPU | Android (OpenGL ES / OpenCL), iOS (Metal) | `TfLiteGpuDelegateCreate()` |
| NNAPI | Android NDK | `ANeuralNetworks*` functions |
| CoreML | iOS | `MLModel` API |
| Hexagon | Qualcomm DSP | Hexagon NN library |

## Same-Repo Facts That Must Not Be Classified As Cross-Repo

| Fact | Correct Bucket |
|---|---|
| FlatBuffers model parsing | Vendored (same repo checkout) |
| XNNPACK delegate integration | Vendored (bundled in third_party/) |
| GPU delegate OpenGL/Metal backends | Repo-Internal (delegate lives in lite/delegates/gpu/) |
| NNAPI delegate Android calls | Platform API (external, not a configured repo) |

## Boundary Risks

- [LOW] FlatBuffers schema version must match between converter and runtime
- [LOW] Platform delegate APIs (NNAPI, CoreML) change with OS versions; delegate code must adapt

## Needs Developer Confirmation

- Should FlatBuffers or XNNPACK be tracked as adjacent repos?
