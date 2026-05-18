# device_base.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/device_base.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `DeviceBase` (L76-L210) — the abstract root of the device hierarchy, providing allocator access, thread pool management, and CPU/accelerator device info. `Device` (in `device.h`) extends this with kernel dispatch and resource management. Separating the base class keeps allocator and threading contracts independent of execution dispatch, allowing tools and utilities to interact with a device's memory system without depending on the full execution layer.

Also defines `DeviceContext` (L53-L131) — a RefCounted wrapper for device-specific execution state (CUDA streams, copy operations, event management) — and `PerOpGpuDevice` (L48-L51) — a thin Eigen GPU device wrapper used by OpKernelContext.

## Code Commentary

### Logic

**DeviceContext (L53-L131):** Ref-counted context passed alongside tensors to track device-side operations:

- `stream()` (L57) — Returns the `stream_executor::Stream` for GPU operations. CPU returns nullptr.
- `CopyCPUTensorToDevice / CopyDeviceTensorToCPU` (L66-L103) — Async tensor copies between host and device, signal completion via `StatusCallback`.
- `MaintainLifetimeOnStream(tensor, stream)` (L59) — Keeps tensor memory alive until stream work completes, preventing use-after-free from eager deallocation.
- `CopyTensorInSameDevice` (L83) — Intra-device tensor copy; default Unimplemented, GPU overrides for device-to-device memcpy.
- `ThenExecute(device, stream, func)` (L107) — Schedules a host function to run after all stream work completes.
- `host_memory_allocator()` (L128) — Returns pinned host memory allocator for efficient CPU↔GPU transfers.

**DeviceBase (L76-L210):** Abstract device with allocator and threading infrastructure:

- `env()` (L80) — Returns `tsl::Env` (filesystem, threading).
- `CpuWorkerThreads` (L82-L85) — Thread count and pool for CPU work. Set before graph execution starts.
- `AcceleratorDeviceInfo` (L97-L105) — Holds stream, default DeviceContext, EventMgr, and GPU ID. Set by GPU/TPU device implementations.
- `GetAllocator(AllocatorAttributes)` (L124) — Virtual; GPU returns BFCAllocator, CPU returns CPUAllocator. Base impl LOG(FATAL).
- `GetScopedAllocator(attr, step_id)` (L131) — Per-step allocator variant for graph optimization passes.
- `eigen_cpu_device()` (L140) — Returns `Eigen::ThreadPoolDevice` wrapping the CPU thread pool.
- `MakeTensorFromProto(tensor_proto, alloc_attrs, tensor*)` (L174) — Materializes a TensorProto into device memory.
- `SafeAllocFrontier(uint64_t)` (L183) — Returns a counter for GPU memory reuse safety; enables aggressive reclamation.
- `CopyTensorInSameDevice` (L187) — Intra-device deep copy.

**Symbolic Execution Devices (L215-L218):** `AddSymbolicExecutionDevice(name)` and `IsSymbolicExecutionDevice(name)` — used by the TF-XLA bridge. TF should not treat symbolic devices as normal execution targets.

### Conventions

- **Virtual inheritance** — DeviceBase is an abstract class; concrete implementations override GetAllocator, MakeTensorFromProto, etc.
- **Ref-counted DeviceContext** — Callers must Unref after TryGetDeviceContext; forgetting to do so leaks stream references.
- **Borrowed pointers** — DeviceBase does not own its thread pools or device info; callers ensure lifetime.

### Invariants And Boundaries

- AllocatorAttributes determine memory region (on-device, host-accessible, etc.).
- AcceleratorDeviceInfo must be fully populated (non-null) before GPU execution begins.
- Eigen CPU device vector (`eigen_cpu_devices_`) is populated at setup and read-only thereafter.
- DeviceBase does not own its thread pools or device info — callers guarantee the pointed-to objects outlive the device.- **GetAllocator FATALs by default** — Base impl calls LOG(FATAL); concrete devices MUST override.
- **DeviceContext ref counting** — TryGetDeviceContext returns a RefCounted pointer. Forgetting to Unref() leaks memory AND the underlying stream reference.
- **SafeAllocFrontier may return 0** — 0 means "no safety guarantee," not "everything is safe." Allocators must handle this case.
- **CPUWorkerThreads nullptr check** — `tensorflow_cpu_worker_threads()` CHECK-fails if `cpu_worker_threads_` is null. Must be set before any op that uses Eigen.
- **MakeTensorFromProto is device-specific** — GPU/TPU implementations handle host↔device copies; calling the base impl returns an error.
- **Symbolic devices must not be used for execution** — They exist only for XLA compilation tracing.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. DeviceBase is an internal C++ abstraction not covered by public TF documentation.

## Repo-Internal References

DeviceBase provides the memory and threading foundation that Device extends with kernel dispatch and resource management.

| Finding | Citations | Source Path |
|---|---|---|
| Device extends DeviceBase with kernel dispatch and resource mgmt | Device class | [`device.h`](device.h.md) |
| Allocator and AllocatorAttributes for memory attribution | allocator.h | [`tensorflow/core/framework/allocator.h`](allocator.h) |
| BFCAllocator is the GPU allocator implementation | BFCAllocator | [`tensorflow/core/common_runtime/bfc_allocator.h`](../../common_runtime/bfc_allocator.h) |
| EventMgr defers GPU buffer deallocation | EventMgr | [`tensorflow/core/common_runtime/event_mgr.h`](../../common_runtime/event_mgr.h) |
| OpKernelContext uses DeviceContext for stream-ordered execution | OpKernelContext | [`op_kernel.h`](op_kernel.h.md) |
| Eigen::ThreadPoolDevice wraps the CPU thread pool | Eigen interop | `third_party/eigen3` |

## Cross-Repo References

No meaningful cross-repo references found. Eigen and stream_executor are vendored dependencies, not cross-repo boundaries.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with DeviceBase, DeviceContext, AcceleratorDeviceInfo, allocator access, safe alloc frontier, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
