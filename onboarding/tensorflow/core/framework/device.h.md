# device.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/device.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `Device` (L45-L188) — the concrete device class extending `DeviceBase` with kernel execution dispatch, resource management, and op segment tracking. Each `Device` instance represents one physical or virtual compute target (CPU core, GPU, TPU slice). The `Device` class manages `OpKernel` registration per (op_type, device) pair via `OpSegment`, provides `Compute()` and `ComputeAsync()` for kernel execution, and owns a `ResourceMgr` for per-device resource variables and tensors.

## Code Commentary

### Logic

**Device (L45-L188):** Extends `DeviceBase` (L45). Key members and methods:

**Kernel dispatch:**
- `Compute(op_kernel, &context)` (L120) — Synchronous kernel execution. Blocks until the kernel's Compute() returns.
- `ComputeAsync(op_kernel, &context, done)` (L130) — Asynchronous kernel execution. The `done` callback is invoked when the kernel's `ComputeAsync()` signals completion. GPU kernels typically use this path to avoid blocking host threads.
- `TryGetDeviceContext(&device_context)` (L80) — Returns the per-device `DeviceContext` for stream-ordered execution. Ref-counted; callers must Unref.

**Op registry per device:**
- `op_segment_` (L160) — An `OpSegment` tracking which `OpKernel` handles each (op_type, node_name) pair on this device. One device can have multiple kernels for the same op type (different instances, different attributes).
- `GetOpSegment()` (L85) — Returns a reference to the op segment for inspection/registration.

**Resource management:**
- `resource_manager()` (L90) — Returns a `ResourceMgr*` that lives for the device's lifetime. Used by resource variables, lookup tables, and queues.
- `Attributes()` (L100) — Returns device attributes (DeviceAttributes proto) for placement and device matching.

**Construction:**
- Constructor (L70-L75) takes `tsl::Env*`, `DeviceAttributes`, and `std::unique_ptr<DeviceBase::AcceleratorDeviceInfo>`. GPU devices pass populated AcceleratorDeviceInfo; CPU devices pass nullptr.

### Conventions

- **One Device per physical/logical target** — No sharing of Device instances across hardware contexts.
- **Owned by DeviceMgr** — Devices are managed by `DeviceMgr` (device_set.h), not created standalone.
- **ResourceMgr per device** — Resource lifetimes are tied to the device, not the session. Resources survive across session Create/Close boundaries.

### Invariants And Boundaries

- Device::Compute() must not be called on an uninitialized device — AcceleratorDeviceInfo must be non-null for GPU devices.
- OpSegment entries are unique per (op_type, node_name) — duplicate registration is an error.
- ResourceMgr outlives all kernels — resources are freed only when the Device is destroyed.
- Device is not copyable or movable.- **ComputeAsync callback fires on an unspecified thread** — The `done` callback for GPU kernels may run on a stream executor thread. Callers must not assume the calling thread.
- **DeviceContext ref counting** — `TryGetDeviceContext()` returns a RefCounted pointer. Forgetting to Unref leaks the stream reference and may delay GPU memory reclamation.
- **OpSegment is per-device** — A kernel registered on GPU 0 is not available on GPU 1. Placer must assign the right device before kernel lookup.
- **ResourceMgr does not persist across sessions** — Resources stored in the device's ResourceMgr are per-process, not per-session. Multiple sessions on the same device share resources.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. Device is an internal C++ abstraction not covered by public TF documentation.

## Repo-Internal References

Device is the central execution target — ops are placed on devices, kernels are registered per-device, and resources are per-device.

| Finding | Citations | Source Path |
|---|---|---|
| DeviceBase provides the allocator/threading contract Device extends | DeviceBase class | [`device_base.h`](device_base.h.md) |
| OpKernel executed by Device::Compute/ComputeAsync | OpKernel class | [`op_kernel.h`](op_kernel.h.md) |
| DeviceMgr owns the set of all devices in a session | DeviceMgr | [`tensorflow/core/common_runtime/device_mgr.h`](../../common_runtime/device_mgr.h) |
| ResourceMgr for per-device resource variables and tensors | ResourceMgr | [`tensorflow/core/framework/resource_mgr.h`](resource_mgr.h) |
| Placer assigns nodes to Device instances | Placer | [`tensorflow/core/common_runtime/placer.h`](../../common_runtime/placer.h.md) |
| DeviceContext for stream-safe GPU execution | DeviceContext | [`device_base.h`](device_base.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template — merged Key Invariants and Traps into `### Invariants And Boundaries`, moved Docs References under `## Code Commentary`, added `### Conventions` and `### Todos`, added `doc_type` metadata.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with Device class API, DeviceContext, stream management, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
