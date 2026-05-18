# pluggable_device_factory.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `72ec73a31c7094d6a0353690fb583efd70d74e60` |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |
| governingOverview | `tensorflow/core/overview.md` |

## Governing Overview

This file is governed by the Core Runtime overview: [`tensorflow/core/overview.md`](../../overview.md).

## Purpose

This header declares `PluggableDeviceFactory`, the `DeviceFactory` implementation that lists physical pluggable devices, creates TensorFlow `PluggableDevice` instances from session options, returns device details, computes device locality, and constructs individual devices with memory and optional priority settings.

## Code Commentary

### Logic

The public API is the TensorFlow runtime `DeviceFactory` surface: `ListPhysicalDevices()`, `CreateDevices()`, and `GetDeviceDetails()`. The factory is parameterized by a TensorFlow device type and a StreamExecutor platform name.

The private API keeps the creation process split between locality discovery and device construction. `GetDeviceLocalities()` fills one `DeviceLocality` per TensorFlow device id. `CreatePluggableDevice()` allocates the memory-limited device, accepts an optional stream priority, and appends the result to the device vector.

### Conventions

- The factory stores `device_type_` and `platform_name_` as immutable strings after construction.
- Device creation signatures use `TfDeviceId` for TensorFlow-visible identity and `DeviceLocality` for runtime placement metadata.
- Optional priority is part of the private construction contract, not a public `DeviceFactory` method parameter.

### Invariants And Boundaries

- `CreatePluggableDevice()` must receive the same priority parsed for the corresponding virtual device in `pluggable_device_factory.cc`.
- Public `DeviceFactory` overrides should not expose plugin-specific C ABI details.
- The implementation file owns validation, platform lookup, memory-limit derivation, device-id mapping, and error construction.

### Todos

- No file-local TODOs in this header.

### Docs References

The public docs describe the pluggable device mechanism at a package and device-visibility level. The factory declaration is internal runtime machinery.

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow documents plugin packages that add new device support without device-specific TensorFlow source changes. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

The implementation defines the virtual-device parsing and priority propagation that this header declares.

| Finding | Citations | Source Path |
|---|---|---|
| `PluggableDeviceFactory` subclasses `DeviceFactory` and declares listing, creation, and detail methods. | L35-L47 | [pluggable_device_factory.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h) |
| The private creation helper takes a memory limit and optional priority before appending to the device vector. | L48-L64 | [pluggable_device_factory.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h) |
| The implementation parses virtual-device priority into `TfDeviceSpec` records. | L126-L217 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |
| `CreatePluggableDevice()` logs priority and forwards it into `PluggableDevice::Init()`. | L332-L388 | [pluggable_device_factory.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc) |

## Cross-Repo References

No adjacent repository is configured for this file. It declares TensorFlow runtime factory hooks for same-checkout pluggable device code.

| Finding | Citations | Source Path |
|---|---|---|
| The header depends only on TensorFlow runtime headers and public session options in this checkout. | L26-L33 | [pluggable_device_factory.h](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h) |

## Update History

- 2026-05-18: Created file-level onboarding for the PluggableDeviceFactory declaration after priority became part of private device construction
