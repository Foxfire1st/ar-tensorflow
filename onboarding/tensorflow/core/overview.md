# Core Runtime Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/core/` |
| onboardingRoute | `tensorflow/core/overview.md` |
| parentOverview | [`overview.md`](../../overview.md) |
| lastUpdated | 2026-05-19T03:08:25+02:00 |
| lastVerifiedCommitHash | 575e43785c913c58644408a5ef5a7ff32c9026f5 |
| lastVerifiedCommitDate | 2026-05-17T20:03:29-07:00 |

## What This Area Is

The C++ runtime core is the engine that powers all graph computation, distributed execution, and hardware acceleration in TensorFlow. It contains the fundamental abstractions (Graph, Tensor, OpKernel, Device) and the execution machinery (Session, Executor, Rendezvous, Placer) that orchestrate computation across single and multi-machine deployments.

This area is entirely C++, with a thin public API layer (`public/`). It is the foundation upon which all TensorFlow APIs (Python, C, C++, Java, Go, JavaScript) are built. The core owns the contract for operation implementations (OpKernel), device placement logic, memory management, tensor flow synchronization, and the distributed Master/Worker runtime.

## Hot Path Summary

Use this route for graph execution, `Executor`, `DirectSession`, `OpKernel`, `Device`, `Rendezvous`, `Placer`, allocator, and pluggable-device runtime behavior. For XLA compilation semantics, prefer `tensorflow/compiler/`; use core only when the question reaches runtime dispatch, graph execution, or device integration.

## What Belongs Here

| Path | Role |
|---|---|
| `framework/` | Core abstractions: Tensor, OpKernel, Device, Op registry, shape inference, allocator, rendezvous |
| `graph/` | Graph DAG construction, validation, optimization, partitioning |
| `common_runtime/` | Execution engine: Executor, DirectSession, Placer, Grappler, device management |
| `common_runtime/pluggable_device/` | StreamExecutor-backed pluggable hardware device factory, device lifecycle, allocators, contexts, virtual devices, and stream setup |
| `kernels/` | 1000+ concrete operation implementations (math, NN, I/O, control flow) |
| `ops/` | Op definitions and gradient registration |
| `distributed_runtime/` | Multi-machine: Master, Worker, gRPC transport, graph distribution |
| `platform/` | Platform abstractions: threading, logging, file I/O |
| `lib/` | Utility libraries: ref-counting, hashtables, string utils |
| `data/` | tf.data Dataset pipeline infrastructure |
| `grappler/` | Graph optimization passes (constant folding, CSE, layout) |
| `profiler/` | Performance profiling and tracing |
| `protobuf/` | Protocol buffer definitions (GraphDef, NodeDef) |
| `public/` | Public C++ API headers (Session interface) |
| `transforms/` | Graph transformation passes |
| `tfrt/` | TensorFlow Runtime (experimental async runtime) |
| `tpu/` | TPU-specific execution and compilation |
| `debug/` | Graph debugging infrastructure |
| `nccl/` | NVIDIA Collective Communications integration |
| `summary/` | TensorBoard summary collection |
| `ir/` | Internal representations (MLIR integration) |

## What Does Not Belong Here

| Nearby Thing | Belongs Instead In |
|---|---|
| XLA compiler and backends | `tensorflow/compiler/` and `third_party/xla/xla/` |
| Python user API (Keras, eager) | `tensorflow/python/` |
| Mobile/embedded runtime | `tensorflow/lite/` |
| C language bindings | `tensorflow/c/` |
| C++ DSL (Scope) | `tensorflow/cc/` |
| Distributed tensor sharding | `tensorflow/dtensor/` |

## Structures Found Here

| Structure | Defining File | Purpose |
|---|---|---|
| Graph | `graph/graph.h` | DAG of Nodes/Edges; the computation IR |
| Node | `graph/graph.h` | Vertex in Graph; stores op type, name, inputs, device |
| Edge | `graph/graph.h` | Directed edge between nodes (data or control dependency) |
| Tensor | `framework/tensor.h` | N-dimensional typed array with ref-counted buffer |
| OpKernel | `framework/op_kernel.h` | Base class for op implementations; Compute() is the contract |
| Device | `framework/device.h` | CPU/GPU/TPU abstraction; owns allocator, thread pool |
| Session | `public/session.h` | User-facing API: Create/Extend/Run graphs |
| DirectSession | `common_runtime/direct_session.h` | Local single-machine session |
| Executor | `common_runtime/executor.h` | Runs graph; manages pending counts, rendezvous |
| Rendezvous | `framework/rendezvous.h` | Producer-consumer tensor passing synchronization |
| Placer | `common_runtime/placer.h` | Device placement algorithm |
| Allocator | `framework/allocator.h` | Memory allocation interface; BFCAllocator for GPU |
| FunctionLibraryRuntime | `framework/function.h` | tf.function execution, inlining, caching |
| PluggableDevice | `common_runtime/pluggable_device/pluggable_device.h` | LocalDevice implementation for StreamExecutor-backed external hardware platforms |
| PluggableDeviceFactory | `common_runtime/pluggable_device/pluggable_device_factory.h` | DeviceFactory that maps platform devices and virtual devices into TensorFlow devices |
| TfDeviceSpec | `common_runtime/pluggable_device/pluggable_device_factory.cc` | Internal factory record carrying platform id, TF id, memory limit, and optional stream priority |

## Operating Model

### Graph Execution Flow

1. **Graph Construction** — User builds a Graph via API, adding Nodes with operators
2. **Session Creation** — DirectSession created with SessionOptions (devices, threads)
3. **Graph Registration** — Session::Create/Extend registers graph, validates, builds devices
4. **Graph Optimization** — Grappler applies passes (constant folding, CSE, layout)
5. **Graph Partitioning** — Placer assigns nodes to devices; cross-device edges become Send/Recv
6. **Executor Creation** — Per-device partition gets its own Executor
7. **Session::Run()** — User calls with feeds/fetches; feeds copied to Rendezvous
8. **Executor Dispatch** — Each Executor::RunAsync() runs independently
9. **Kernel Dispatch** — Ready nodes dispatched via Device::Compute(OpKernel, context)
10. **Tensor Passing** — Output tensors written to Rendezvous; downstream waits via Recv
11. **Completion** — All executors complete; outputs collected from Rendezvous

### Op Registration Flow

1. REGISTER_OP macro defines OpDef (inputs, outputs, attributes) → OpRegistry
2. REGISTER_KERNEL_BUILDER maps (op_name, device_type) → OpKernel factory
3. At execution, Executor looks up kernel factory, instantiates, caches in OpSegment

### Pluggable Device Creation Flow

1. `PluggableDeviceFactory::CreateDevices()` reads `SessionOptions.config.pluggable_device_options()`
2. Virtual-device config is normalized into `TfDeviceSpec` records with platform device id, TF device id, memory limit, and optional priority
3. Device-id mappings are registered before device objects are created
4. Device locality lookup creates the platform executor before asking for the device description
5. `CreatePluggableDevice()` builds the allocator-backed `PluggableDevice` and passes priority into `Init()`
6. `PluggableDevice::Init()` creates compute, host-to-device, device-to-host, and device-to-device streams with the same optional priority

## Load-Bearing Files

| File | Role | Why It Matters | Onboarding |
|---|---|---|---|
| `graph/graph.h` | Graph DAG | All computation is a graph; this is the IR | planned |
| `framework/tensor.h` | Tensor data | All ops consume/produce Tensors | planned |
| `framework/op_kernel.h` | Op contract | Compute() is the fundamental execution contract | planned |
| `framework/device.h` | Device abstraction | All kernels run on Devices | planned |
| `public/session.h` | Session API | User-facing graph lifecycle | planned |
| `common_runtime/direct_session.h` | Local session | Primary single-machine execution | planned |
| `common_runtime/executor.h` | Execution engine | Core runtime loop | planned |
| `framework/rendezvous.h` | Tensor sync | Inter-device tensor passing | planned |
| `framework/op.h` | Op registry | All ops registered here | planned |
| `framework/function.h` | Function runtime | tf.function execution | planned |
| `common_runtime/placer.h` | Device placement | Node→device assignment | planned |
| `framework/device_base.h` | Device base | Abstract device interface | planned |
| `common_runtime/pluggable_device/pluggable_device.h` | Pluggable device object | External hardware device lifecycle and stream context | candidate |
| `common_runtime/pluggable_device/pluggable_device.cc` | Pluggable device implementation | Stream group creation, allocators, sync, copy behavior | candidate |
| `common_runtime/pluggable_device/pluggable_device_factory.cc` | Pluggable device factory | Virtual-device parsing, locality, executor ordering, priority propagation | candidate |

## Local Invariants And Traps

- **Graph is a DAG** — cycles are forbidden; guaranteed by construction
- **Tensors are immutable** — once produced, value and shape do not change
- **Ref-counted buffers** — TensorBuffer uses ref-counting; zero-copy slicing possible
- **OpKernel::Compute() must be thread-safe** — called concurrently; must not block (use AsyncOpKernel for blocking ops)
- **Device assignment is fixed** — once placed, node runs on same device for graph lifetime
- **Rendezvous deadlock risk** — two devices waiting on each other via Rendezvous hangs indefinitely
- **AsyncOpKernel required for blocking ops** — synchronous OpKernel blocking will deadlock bounded thread pool
- **Cross-device tensor copies** — automatic via Send/Recv ops, incurring latency and memory overhead
- **Pluggable device virtual priorities are per virtual device** — priority entries must either be absent or match `memory_limit_mb` entry count
- **Pluggable device ordinal remains unsupported in virtual-device config** — device ordinal settings still return Unimplemented for pluggable virtual devices
- **Executor before description** — pluggable device locality now forces `ExecutorForDevice()` before `DescriptionForDevice()` so device metadata comes from an initialized executor path

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| OpKernel is the op implementation contract; all kernels inherit | OpKernel class, KernelDefBuilder for registration | `framework/op_kernel.h`, `framework/kernel_def_builder.h` |
| Session orchestrates via Executor; DirectSession creates per-partition Executors | Session interface, DirectSession implementation | `public/session.h`, `common_runtime/direct_session.h` |
| Placer assigns nodes to devices; used by DirectSession after graph registration | Placer::Run(), called from DirectSession | `common_runtime/placer.h`, `common_runtime/direct_session.h` |
| Rendezvous coordinates tensor passing between devices in same session | Rendezvous interface, instantiated in DirectSession | `framework/rendezvous.h`, `common_runtime/rendezvous_mgr.h` |
| Device owns memory allocator, kernel registry, thread pool | Device interface, DeviceMgr manages set | `framework/device.h`, `common_runtime/device_mgr.h` |
| PluggableDevice stream groups are created through StreamExecutor and share compute/copy stream setup per TF device id | `StreamGroupFactory::GetOrCreate()` calls `CreateStream(priority)` for compute, H2D, D2H, and D2D streams | `common_runtime/pluggable_device/pluggable_device.cc`, `tensorflow/c/experimental/stream_executor/stream_executor.cc` |
| PluggableDeviceFactory carries virtual-device memory and optional priority through creation | `ExtractVirtualDevices()` returns `TfDeviceSpec`; `CreatePluggableDevice()` logs and forwards priority to `PluggableDevice::Init()` | `common_runtime/pluggable_device/pluggable_device_factory.cc`, `common_runtime/pluggable_device/pluggable_device.h` |
| Device locality lookup depends on executor initialization before description lookup | `GetDeviceLocalities()` calls `ExecutorForDevice(...).status()` before `DescriptionForDevice()` | `common_runtime/pluggable_device/pluggable_device_factory.cc` |

## Cross-Repo References

| Finding | Citations | Source Path |
|---|---|---|
| XLA compilation backend accessed via compiler stack; HLO IR in third_party/xla/ | — | `tensorflow/compiler/`, `third_party/xla/xla/` |
| Python API calls into core via pywrap/C bindings | — | `tensorflow/python/` |
| TFLite is independent codebase; does NOT use core Executor | — | `tensorflow/lite/` |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow architecture and graph execution | — | https://www.tensorflow.org/guide/core |
| C++ API reference | — | https://www.tensorflow.org/api_docs/cc |
| Op development guide (REGISTER_OP, REGISTER_KERNEL) | — | https://www.tensorflow.org/guide/create_op |
| Graph optimization (Grappler) | — | https://www.tensorflow.org/guide/graph_optimization |

## File-Level Onboarding Map

| Source File | Onboarding File | Status | Reason |
|---|---|---|---|
| `graph/graph.h` | `graph/graph.h.md` | planned | Core DAG abstraction |
| `framework/tensor.h` | `framework/tensor.h.md` | planned | Tensor data container |
| `framework/op_kernel.h` | `framework/op_kernel.h.md` | planned | Op execution contract |
| `framework/device.h` | `framework/device.h.md` | planned | Device abstraction |
| `public/session.h` | `public/session.h.md` | planned | Session API |
| `common_runtime/direct_session.h` | `common_runtime/direct_session.h.md` | planned | Local session |
| `common_runtime/executor.h` | `common_runtime/executor.h.md` | planned | Execution engine |
| `framework/rendezvous.h` | `framework/rendezvous.h.md` | planned | Tensor sync |
| `framework/op.h` | `framework/op.h.md` | planned | Op registry |
| `framework/function.h` | `framework/function.h.md` | planned | Function runtime |
| `common_runtime/pluggable_device/pluggable_device.h` | `common_runtime/pluggable_device/pluggable_device.h.md` | exists | Pluggable device lifecycle and Init contract |
| `common_runtime/pluggable_device/pluggable_device.cc` | `common_runtime/pluggable_device/pluggable_device.cc.md` | exists | Stream group creation and priority propagation |
| `common_runtime/pluggable_device/pluggable_device_factory.h` | `common_runtime/pluggable_device/pluggable_device_factory.h.md` | exists | Factory declaration and priority-aware construction signature |
| `common_runtime/pluggable_device/pluggable_device_factory.cc` | `common_runtime/pluggable_device/pluggable_device_factory.cc.md` | exists | Virtual-device parsing and executor/description ordering |

## Child Overviews

None yet. Candidates: `common_runtime/pluggable_device/`, `kernels/`, `distributed_runtime/`, `grappler/`

## How To Use This Area

1. Read `graph/graph.h` — understand Graph as DAG
2. Read `public/session.h` — trace Session::Run() contract
3. Read `common_runtime/direct_session.h` — local implementation
4. Read `common_runtime/executor.h` — node dispatch
5. Read `framework/rendezvous.h` — tensor synchronization
6. For external hardware/plugin behavior, read `common_runtime/pluggable_device/pluggable_device_factory.cc` with `common_runtime/pluggable_device/pluggable_device.cc`

## Needs Verification

- Exact op and kernel count
- Relationship between tfrt/ and main runtime

## Update History

- 2026-05-19T03:08:25+02:00: Added hot path summary for generated route-index discovery hints.
- 2026-05-18: Added file-level onboarding map entries for PluggableDevice and PluggableDeviceFactory files
- 2026-05-18: Refreshed after pluggable device drift; added virtual-device priority and executor-before-description context
- 2026-05-17: Initial route-local overview from full-bootstrap
