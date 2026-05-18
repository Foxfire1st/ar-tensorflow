# pluggable_device_factory.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `575e43785c913c58644408a5ef5a7ff32c9026f5` |
| lastVerifiedCommitDate | 2026-05-17T20:03:29-07:00 |
| governingOverview | `tensorflow/core/overview.md` |

## Governing Overview

This file is governed by the Core Runtime overview: [`tensorflow/core/overview.md`](../../overview.md).

## Purpose

This file implements the factory that turns registered pluggable StreamExecutor platforms and `SessionOptions` device settings into TensorFlow `PluggableDevice` instances. It owns memory-limit calculation, virtual-device parsing, TensorFlow-to-platform device id mapping, device locality discovery, allocator-backed device construction, and priority propagation into `PluggableDevice::Init()`.

## Code Commentary

### Logic

`MinSystemMemory()` and `SingleVirtualDeviceMemoryLimit()` calculate the memory available to a virtual pluggable device. Unified memory remains unsupported here, and default allocation reserves a minimum system-memory amount unless a per-process memory fraction is configured.

`TfDeviceSpec` is the internal normalized device record: platform device id, memory limit bytes, optional stream priority, and TensorFlow device id. `ExtractVirtualDevices()` builds these records from `pluggable_device_options().experimental().virtual_devices()`, validates unsupported `device_ordinal`, validates priority count against `memory_limit_mb`, converts MB values to bytes, and inserts device-id mappings only after all specs are collected.

`CreateDevices()` validates the registered platform, handles no-device cases, extracts virtual-device specs when present, or builds default specs from visible devices and device count when absent. It then obtains device localities and calls `CreatePluggableDevice()` for each spec with memory and priority.

`CreatePluggableDevice()` maps TensorFlow id back to platform id, obtains the device description and allocator, checks allocator stats, constructs the `PluggableDevice`, logs memory and priority, initializes the device with priority, and appends it to the output vector.

`GetDeviceLocalities()` maps TensorFlow ids to platform ids, creates the executor before asking for the device description, derives NUMA locality, defaults missing NUMA to zero, and records bus id as `numa_node + 1`.

### Conventions

- Virtual-device priority is parallel to `memory_limit_mb`: if priority is present, the counts must match.
- `TfDeviceSpec` is intentionally local to the implementation file; public factory headers stay simpler.
- Device-id mappings are inserted after spec extraction, avoiding partial registration before validation completes.
- The code uses GPUOptions fields for pluggable device options because TensorFlow reuses those protobuf structures for pluggable-device configuration.

### Invariants And Boundaries

- `device_ordinal` is still unsupported for pluggable virtual devices.
- Priority entries are optional, but non-empty priority lists must match `memory_limit_mb` size.
- `CreateDevices()` must build one locality for each TensorFlow device spec before constructing devices.
- `GetDeviceLocalities()` must create the executor before `DescriptionForDevice()` so device description comes from an initialized executor path.
- Allocator stats must be available before constructing a `PluggableDevice`.

### Todos

- `MinSystemMemory()` notes that future memory-reservation heuristics could become more sophisticated.

### Docs References

Public TensorFlow docs establish the pluggable device model and user-visible device behavior. This file carries the internal runtime mapping from session options to device instances.

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow documents pluggable devices as separately installed plugin packages that communicate with the TensorFlow binary through C APIs. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |
| Public examples show pluggable devices becoming visible through TensorFlow physical device listing and usable with `tf.device` and `tf.function`. | L150-L168 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

The factory is the bridge between pluggable device options, TensorFlow device ids, StreamExecutor platforms, and `PluggableDevice` initialization.

| Finding | Citations | Source Path |
|---|---|---|
| Memory-limit calculation queries StreamExecutor device memory and rejects unsupported unified-memory configurations. | L53-L124 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `TfDeviceSpec` stores platform id, memory limit, optional priority, and TensorFlow device id. | L126-L138 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `ExtractVirtualDevices()` validates unsupported ordinals, priority/list size, visible devices, memory conversion, and device-id mapping. | L140-L217 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `CreateDevices()` chooses virtual-device or default-device spec creation, then forwards memory and optional priority into device creation. | L264-L323 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `CreatePluggableDevice()` logs priority and passes it into `PluggableDevice::Init()`. | L332-L388 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `GetDeviceLocalities()` creates the executor before calling `DescriptionForDevice()`. | L391-L435 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| Tests build virtual-device options with priority vectors and verify memory limits and TensorFlow-to-platform mappings. | L66-L104; L126-L149 | [pluggable_device_factory_test.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory_test.cc) |

## Cross-Repo References

No adjacent repository is configured for this file. It uses TensorFlow runtime code plus same-checkout StreamExecutor platform interfaces.

| Finding | Citations | Source Path |
|---|---|---|
| The factory includes StreamExecutor platform interfaces and TensorFlow pluggable-device runtime helpers from the TensorFlow checkout. | L32-L49 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |

## Update History

- 2026-05-18: Created file-level onboarding for virtual-device memory, priority, id mapping, and executor-before-description behavior
