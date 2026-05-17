# TensorFlow Lite Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/lite/` |
| onboardingRoute | `tensorflow/lite/overview.md` |
| parentOverview | [`overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | |
| lastVerifiedCommitDate | |

## What This Area Is

TensorFlow Lite is the mobile and embedded ML runtime. It loads `.tflite` FlatBuffer models, resolves operations via a pluggable `OpResolver`, schedules subgraph execution, and supports hardware acceleration through a delegate interface (GPU, NNAPI, XNNPACK, CoreML, Hexagon). A separate `micro/` implementation targets embedded systems with no heap allocation.

Key design: minimal binary size, low latency, no runtime Protobuf dependency (FlatBuffer is zero-copy), memory arena pre-allocation, and delegate-based hardware offload.

## What Belongs Here

| Path | Role |
|---|---|
| `core/` | Main runtime: Interpreter, Subgraph, model loading, signature runners |
| `c/` | C API & stable type definitions (TfLiteContext, TfLiteTensor, TfLiteRegistration) |
| `kernels/` | 100+ builtin operation kernels (conv, add, reshape, LSTM, etc.) |
| `delegates/` | Hardware acceleration: GPU, NNAPI, XNNPACK, FlexDelegate, CoreML, Hexagon |
| `schema/` | FlatBuffer schema (schema_generated.h) — the .tflite model format |
| `profiling/` | Performance monitoring: op timing, memory tracking |
| `tools/` | Model tools: converter, optimizer, verifier, benchmark |
| `async/` | Asynchronous execution (experimental) |
| `micro/` | TFLite Micro — embedded/microcontroller runtime (separate codebase) |
| `python/` | Python bindings (Interpreter wrapper) |
| `java/` | Android/Java bindings |
| `swift/`, `objc/`, `ios/` | iOS support |
| `experimental/` | Experimental features |
| `internal/` | Internal helpers (signature_def, compatibility) |

## What Does Not Belong Here

| What | Where |
|---|---|
| Core TF runtime (Graph, Executor) | `tensorflow/core/` |
| XLA compiler | `tensorflow/compiler/` |
| Python API | `tensorflow/python/` |
| Model training | `tensorflow/python/keras/` |

## Structures Found Here

| Structure | Defining File | Purpose |
|---|---|---|
| Interpreter | `core/interpreter.h` | Top-level graph execution engine |
| Subgraph | `core/subgraph.h` | Single execution unit with nodes/tensors |
| FlatBufferModel | `core/model_builder.h` | Deserializes .tflite FlatBuffer files (zero-copy mmap) |
| InterpreterBuilder | `core/interpreter_builder.h` | Model→interpreter construction; op resolution, delegate application |
| OpResolver | `core/api/op_resolver.h` | Maps builtin/custom op codes→TfLiteRegistration implementations |
| TfLiteContext | `c/common.h` | Stable C ABI: context passed to all op kernels |
| TfLiteTensor | `c/common.h` | Runtime tensor: data pointer, shape, type, quantization params |
| TfLiteRegistration | `c/common.h` | Op registration: init/free/prepare/invoke function pointers |
| Delegate | `delegates/` | Hardware accelerator abstraction; intercepts ops for GPU/NNAPI/etc. |
| SimpleMemoryArena | `simple_memory_arena.h` | Dynamic tensor allocation with interval-based reuse |

## Operating Model

### Model Loading & Execution
```
.tflite file (FlatBuffer)
  ↓ FlatBufferModel::BuildFromFile() (zero-copy mmap)
  ↓ InterpreterBuilder constructs Interpreter
  ↓ OpResolver maps op codes→kernels
  ↓ Delegates applied (replace supported ops)
  ↓ Interpreter::AllocateTensors() → memory arena planning
  ↓ Interpreter::Invoke() → subgraph execution
```

### Delegate Integration
```
InterpreterBuilder applies delegates during construction
Delegate identifies supported ops → replaces with delegate kernels
During Invoke(), delegate kernels execute on accelerator
Fallback to CPU for unsupported ops
```

## Load-Bearing Files

| File | Role | Onboarding |
|---|---|---|
| `core/interpreter.h` | Interpreter — main execution engine | planned |
| `core/subgraph.h` | Subgraph execution unit | planned |
| `core/model_builder.h` | FlatBuffer model deserialization | planned |
| `core/interpreter_builder.h` | Model→interpreter construction | planned |
| `c/common.h` | Stable C ABI types | planned |
| `core/api/op_resolver.h` | Op resolution abstraction | planned |
| `simple_memory_arena.h` | Dynamic tensor allocation | planned |
| `schema/schema_generated.h` | FlatBuffer schema | planned |

## Local Invariants And Traps

- **.tflite is FlatBuffer** — not Protobuf; zero-copy deserialization
- **Memory arena pre-allocates** — no dynamic allocation during inference
- **Delegate op replacement before AllocateTensors** — ordering matters
- **Micro is separate codebase** — no shared code with standard TFLite; no heap allocation
- **All ops implement against C ABI** (c/common.h) — stable interface

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite Converter converts TF SavedModel→.tflite | TOCO/converter tools | `tensorflow/lite/tools/` |
| Some delegates use XLA for compilation | XNNPACK, GPU delegate | `tensorflow/lite/delegates/` |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite guide | — | https://www.tensorflow.org/lite/guide |
| TFLite inference | — | https://www.tensorflow.org/lite/guide/inference |
| TFLite delegates | — | https://www.tensorflow.org/lite/performance/delegates |
| TFLite Micro | — | https://www.tensorflow.org/lite/microcontrollers |

## File-Level Onboarding Map

(8 files — all planned per coverage plan wave 4)

## Child Overviews

Candidates: `delegates/`, `micro/`

## How To Use This Area

1. Read `core/interpreter.h` — main entry point
2. Read `core/subgraph.h` — execution unit
3. Read `c/common.h` — stable ABI types
4. Read `core/model_builder.h` — model loading
5. Read `core/api/op_resolver.h` — op resolution

## Needs Verification

- Exact delegate count and platform coverage
- Micro/code sharing boundary details

## Update History

- 2026-05-17: Initial route-local overview from full-bootstrap
