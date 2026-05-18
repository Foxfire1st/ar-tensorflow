# Area Report — lite

| Field | Value |
|---|---|
| repo | tensorflow |
| area | lite |
| sourceRoute | `tensorflow/lite/` |
| generated | 2026-05-17T17:00 |
| priority | HIGH |
| confidence | [HIGH] |

## Structure

```
core/       → Interpreter, Subgraph, InterpreterBuilder, OpResolver, TfLiteRuntime
c/          → Stable C ABI: TfLiteContext, TfLiteTensor, TfLiteRegistration
kernels/    → 100+ builtin op implementations (conv, pooling, activation, etc.)
delegates/  → GPU, NNAPI, XNNPACK, CoreML, Hexagon delegates
schema/     → FlatBuffer model schema (schema_generated.h + schema.fbs)
profiling/  → Performance instrumentation
```

## Interfaces

| Interface | Role | Key Files |
|---|---|---|
| Interpreter | Top-level execution: load → allocate → invoke | `core/interpreter.h` |
| Subgraph | Per-subgraph execution unit with tensors/nodes | `core/subgraph.h` |
| FlatBufferModel | `.tflite` deserialization | `core/model_builder.h` |
| InterpreterBuilder | Model → Interpreter factory | `core/interpreter_builder.h` |
| TfLiteContext/Tensor/Registration | Stable C ABI for op/delegate implementations | `c/common.h` |
| OpResolver | Op code → TfLiteRegistration mapping | `core/api/op_resolver.h` |
| SimpleMemoryArena | Dynamic tensor memory allocator | `simple_memory_arena.h` |
| TfLiteDelegate | Hardware accelerator plugin interface | `c/common.h` |

## Patterns

- **FlatBuffer models** — zero-copy deserialization (unlike TF's protobuf)
- **Static memory planning** — all tensor buffers pre-allocated in shared arena; no malloc during inference
- **Op interface via function pointers** — TfLiteRegistration with init/free/prepare/invoke
- **Delegate substitution** — delegates replace node(s) with a single delegate node; op-resolved post-substitution
- **Signature-based execution** — SignatureRunner for multi-signature models

## Concerns

- **Memory constraints** — arena size is fixed after AllocateTensors(); no dynamic growth
- **Op availability** — op must be in resolver; missing ops fail at build time
- **Delegate compatibility** — not all ops support all delegates; fallback can silently degrade performance
- **Schema versioning** — FlatBuffer schema evolves; forward/backward compatibility handled at load time
- **Micro separate** — `micro/` is a separate codebase with no shared runtime code

## Key Files

| File | Role | Onboarding |
|---|---|---|
| `core/interpreter.h` | Execution engine | done |
| `core/subgraph.h` | Subgraph execution unit | done |
| `core/model_builder.h` | Model loader | done |
| `core/interpreter_builder.h` | Model→Interpreter factory | done |
| `core/c/common.h` | Stable C ABI | done |
| `core/api/op_resolver.h` | Op resolution interface | done |
| `simple_memory_arena.h` | Memory allocator | done |
| `schema/schema_generated.h` | FlatBuffer schema types | done |

## Confidence

[HIGH] — All 8 load-bearing files source-read and onboarded. Architecture verified from headers.

## Update History

- 2026-05-17: Initial area report from full-bootstrap Phase 2
