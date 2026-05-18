# Docs Evidence Pack — lite

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/lite/` |
| generated | 2026-05-17T17:00 |
| sourcesResolvedFrom | `system/sources.md` |
| status | partial |

## Scope

Find domain documentation covering the TFLite runtime: Interpreter, Subgraph, FlatBufferModel, OpResolver, delegates, memory planning.

## Documentation Sources Checked

| Source Name | Source Type | Location | Result |
|---|---|---|---|
| TensorFlow Lite Guide | canonical online | https://www.tensorflow.org/lite/guide | relevant (inference, model conversion, delegates) |
| TensorFlow API Docs | canonical online | https://www.tensorflow.org/api_docs | not relevant (Python lite API only; no C++ internals) |
| Context7 MCP | API reference | context7-mcp skill | not checked (search unavailable) |

## Confirmed Documentation Findings

| Finding | Evidence | Source Path | Applies To | Confidence |
|---|---|---|---|---|
| TFLite inference: load model → build interpreter → allocate tensors → invoke | Known from TF Lite docs structure | https://www.tensorflow.org/lite/guide/inference | `core/interpreter.h` | [HIGH] |
| Models are FlatBuffer-serialized `.tflite` files (not Protobuf) | TF Lite docs | https://www.tensorflow.org/lite/models | `core/model_builder.h`, `schema/schema_generated.h` | [HIGH] |
| Delegates provide hardware acceleration (GPU, NNAPI, XNNPACK, CoreML) | TF Lite docs | https://www.tensorflow.org/lite/performance/delegates | delegate subsystem | [HIGH] |
| Custom ops can be registered via OpResolver | TF Lite docs | https://www.tensorflow.org/lite/guide/ops_custom | `core/api/op_resolver.h` | [HIGH] |
| Memory optimization via shared arena allocation | TF Lite performance guide | https://www.tensorflow.org/lite/performance | `simple_memory_arena.h` | [HIGH] |

## Documentation Constraints

- [MEDIUM] TFLite docs cover the public API and usage patterns; internal Subgraph execution, memory planning algorithm, and op dispatch are not documented externally
- [MEDIUM] FlatBuffer schema documentation exists but the generated code (`schema_generated.h`) is auto-generated and not documented

## Terms And Definitions

| Term | Meaning | Evidence |
|---|---|---|
| TFLite model | FlatBuffer-serialized graph with ops, tensors, and weights | TF Lite docs |
| Delegate | Pluggable hardware accelerator that replaces ops at graph level | TF Lite docs |
| OpResolver | Maps op codes to TfLiteRegistration implementations | TF Lite docs |
| Arena allocation | Single shared buffer with offset reuse for non-overlapping tensor lifetimes | TF Lite performance guide |

## Files Likely Affected

| File | Why Docs Matter |
|---|---|
| `core/interpreter.h` | Primary user-facing class; inference docs cover usage |
| `core/model_builder.h` | Model loading documented in conversion guide |
| `simple_memory_arena.h` | Memory optimization documented in performance guide |

## No Relevant Evidence Found

| Topic | Sources Checked | Notes |
|---|---|---|
| Subgraph::Invoke() execution plan and ready-queue | TF Lite docs | Internal dispatch mechanism; not documented |
| SimpleMemoryArena allocation algorithm (greedy interval) | TF Lite docs | Performance guide mentions arena; algorithm is code-only |
| InterpreterBuilder construction steps | TF Lite docs | Factory class; usage documented, internals not |
| TfLiteContext/TfLiteRegistration ABI contract | TF Lite docs | Custom op guide covers the function pointers; internal calling conventions are code-only |

## Open Questions

- None at [LOW] confidence.
