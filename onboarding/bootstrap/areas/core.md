# Area Report — core

| Field | Value |
|---|---|
| repo | tensorflow |
| area | core |
| sourceRoute | `tensorflow/core/` |
| generated | 2026-05-17T17:00 |
| priority | CRITICAL |
| confidence | [HIGH] |

## Structure

The core C++ runtime is organized in layered internal boundaries:

```
public/         → Session API (user-facing entry point)
framework/      → Tensor, OpKernel, Device, Op, Function, ResourceMgr
graph/          → Graph DAG, Node, Edge, attributes
common_runtime/ → Executor, DirectSession, Placer, Rendezvous, BFCAllocator
kernels/        → 1000+ OpKernel implementations (cpu/, gpu/)
ops/            → Op registration macros (REGISTER_OP)
distributed_runtime/ → Master/Worker gRPC service, remote rendezvous
grappler/       → Graph optimization passes (constant folding, layout, etc.)
platform/       → OS abstraction, threading, logging
protobuf/       → Generated protobuf types (GraphDef, NodeDef, etc.)
lib/            → Generic C++ utilities (hash, random, etc.)
profiler/       → Instrumentation and profiling
data/           → tf.data input pipeline (Dataset, Iterator)
```

## Interfaces

| Interface | Role | Key Files |
|---|---|---|
| Session API | User-facing create/extend/run/close | `public/session.h` |
| OpKernel contract | Op implementors: `Compute(OpKernelContext*)` | `framework/op_kernel.h` |
| Device abstraction | CPU/GPU/TPU allocator, stream, threadpool | `framework/device.h`, `framework/device_base.h` |
| Function Library | Function definition, instantiation, optimization | `framework/function.h` |
| Rendezvous | Producer-consumer tensor passing (local/remote) | `framework/rendezvous.h` |
| Placer | Assigns ops to devices (cost model + constraints) | `common_runtime/placer.h` |
| C API bridge | `c/c_api.cc` wraps Session and Graph | `../c/c_api.h` |

## Patterns

- **Ref-counted tensors** — TensorBuffer is ref-counted; Tensor is a thin handle
- **Op registration via macros** — `REGISTER_OP("name").Input(...).Output(...)` in `ops/`
- **Kernel registration via macros** — `REGISTER_KERNEL_BUILDER(Name("name").Device(DEVICE_CPU), KernelClass)`
- **Graph as DAG** — no cycles in operation dependencies; edges are data or control
- **Eager fallback** — in eager mode, each op executes immediately; no graph is built
- **Device-local allocators** — each device has its own BFCAllocator, not a global heap

## Concerns

- **Thread safety** — Executor runs ready ops concurrently; OpKernel::Compute must be reentrant
- **Memory** — BFCAllocator fragmentation is a known issue for large models
- **Shape inference** — ops must support partial shapes for dynamic graph building
- **Backward compatibility** — GraphDef proto must remain compatible across versions
- **Device placement** — Placer can fail silently if no device satisfies constraints

## Key Files

| File | Role | Onboarding |
|---|---|---|
| `graph/graph.h` | DAG data structure | done |
| `framework/tensor.h` | Tensor + TensorBuffer | done |
| `framework/op_kernel.h` | Op execution contract | done |
| `framework/op.h` | Op registration + definition | done |
| `framework/device.h` | Device abstraction + allocators | done |
| `framework/device_base.h` | Base device + field types | done |
| `framework/function.h` | Function definition + library | done |
| `framework/rendezvous.h` | Tensor send/recv | done |
| `common_runtime/executor.h` | Graph execution engine | done |
| `common_runtime/direct_session.h` | Local session implementation | done |
| `common_runtime/placer.h` | Op-to-device assignment | done |
| `public/session.h` | User-facing Session API | done |

## Confidence

[HIGH] — All 12 load-bearing files have been source-read and onboarded. Architecture verified from headers and BUILD files.

## Update History

- 2026-05-17: Initial area report from full-bootstrap Phase 2
