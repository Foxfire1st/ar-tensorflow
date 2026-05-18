# pluggable_device.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/pluggable_device/pluggable_device.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `72ec73a31c7094d6a0353690fb583efd70d74e60` |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |
| governingOverview | `tensorflow/core/overview.md` |

## Governing Overview

This file is governed by the Core Runtime overview: [`tensorflow/core/overview.md`](../../overview.md).

## Purpose

This header declares `tensorflow::PluggableDevice`, the `LocalDevice` implementation used for StreamExecutor-backed external hardware devices. It defines the device lifecycle contract, compute/copy entrypoints, allocator hooks, executor access, and private stream/context state that the implementation fills during initialization.

## Code Commentary

### Logic

`PluggableDevice` is constructed with session options, TensorFlow device identity, platform identity, memory limit, locality, physical device description, device allocator, CPU allocator, and a sync-every-op flag.

`Init()` is the post-construction setup step. It now accepts an optional stream priority, defaulting to no priority, and returns initialization status after the implementation obtains a StreamExecutor, event manager, stream group, device context, and accelerator device info.

The public overrides cover synchronous and async kernel execution, device synchronization, allocator selection, tensor proto materialization, and same-device tensor copies. The `executor()` accessor exposes the StreamExecutor that controls the pluggable device after initialization.

Private state keeps the device and CPU allocators, the StreamExecutor pointer, stream group pointers for compute and copy paths, the `PluggableDeviceContext`, accelerator info, TensorFlow device id, platform name, event manager, optional thread pool, and host-allocation compatibility flag.

### Conventions

- Construction stores identity and allocator inputs; `Init()` wires runtime resources that depend on the registered platform and TensorFlow device mapping.
- Stream groups separate compute, host-to-device, device-to-host, and device-to-device paths.
- Copy helpers return an immediate status for pre-enqueue failures while still reporting completion through a callback.

### Invariants And Boundaries

- `Init()` is the boundary where the optional virtual-device stream priority enters the concrete device.
- `executor_` starts as `nullptr` and is valid only after successful initialization.
- `device_context_` is owned by the device after initialization and must be unreferenced by the destructor.
- The header declares lifecycle and runtime hooks only; stream-group allocation and copy behavior live in `pluggable_device.cc`.

### Todos

- Existing implementation comments still track possible accelerator-info naming cleanup and future movement of GPU-named environment variables into pluggable-device callbacks.

### Docs References

TensorFlow's public docs describe pluggable devices as external packages that integrate with TensorFlow through C APIs. This header is an internal runtime declaration for that mechanism.

| Finding | Citations | Source Path |
|---|---|---|
| Pluggable device support is distributed as plugin packages and communicates with TensorFlow through C APIs. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

The implementation and factory define how this class is initialized and constructed.

| Finding | Citations | Source Path |
|---|---|---|
| `PluggableDevice` subclasses `LocalDevice`, declares `Init()` with optional stream priority, and exposes compute, sync, allocator, tensor-copy, and executor methods. | L49-L83 | [pluggable_device.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.h) |
| Private state stores allocators, StreamExecutor, stream group, device context, accelerator info, event manager, and optional thread pool. | L85-L120 | [pluggable_device.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.h) |
| The implementation passes stream priority into stream group creation during initialization. | L203-L234 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| The factory forwards optional priority into `PluggableDevice::Init()`. | L332-L388 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |

## Cross-Repo References

No adjacent repository is configured for this file. It depends on StreamExecutor types from the TensorFlow checkout and is linked to the experimental C plugin ABI under `tensorflow/c/`.

| Finding | Citations | Source Path |
|---|---|---|
| The header depends on StreamExecutor and the pluggable-device context/runtime headers inside the TensorFlow checkout. | L25-L45 | [pluggable_device.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.h) |

## Update History

- 2026-05-18: Created file-level onboarding for the PluggableDevice declaration after optional stream priority was added to `Init()`
