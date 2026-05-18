# pluggable_device.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `72ec73a31c7094d6a0353690fb583efd70d74e60` |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |
| governingOverview | `tensorflow/core/overview.md` |

## Governing Overview

This file is governed by the Core Runtime overview: [`tensorflow/core/overview.md`](../../overview.md).

## Purpose

This file implements `PluggableDevice`: stream-group creation, initialization, allocator selection, kernel dispatch, synchronization, tensor copies, and TensorProto materialization for external hardware devices registered through TensorFlow's pluggable device runtime.

## Code Commentary

### Logic

`StreamGroupFactory` is a process-global cache keyed by device type, TensorFlow device id, and stream-group id. It creates the compute, host-to-device, device-to-host, and one to four device-to-device streams on first use, stores ownership in `allocated_streams_`, and returns raw stream pointers in the shared `StreamGroup`.

The stream-group creation path accepts optional priority and passes it to every stream created for the group. It also normalizes `num_dev_to_dev_copy_streams`: zero becomes one, invalid values outside one to four log an error and fall back to one.

`PluggableDevice::Init()` obtains the platform executor for the TensorFlow device id, creates the event manager, creates or reuses the stream group with optional stream priority, builds the `PluggableDeviceContext`, fills accelerator device info, maps the TensorFlow device id back to platform device id, and configures optional GPU-named thread-pool modes.

The compute paths log debug strings that include stream id and delegate the actual work to `OpKernel::Compute()` or `AsyncOpKernel::ComputeAsync()`. Copy and tensor-proto paths enforce DMA-copyability, allocate device tensors, wait for variant copies where needed, and use the device context for host-to-device transfer.

### Conventions

- Stream groups are shared per device and group id because the allocator is per device and must not be shared across unrelated streams.
- Environment variables still use GPU names even for pluggable devices; implementation comments mark this as future cleanup.
- `device_context_` is the default context, but op contexts can override it with a specific `PluggableDeviceContext`.

### Invariants And Boundaries

- `StreamGroupFactory::GetOrCreate()` is guarded by a mutex and intentionally leaks the global factory to preserve stream lifetime.
- All streams in a stream group receive the same optional priority.
- Device-to-device copy stream count is constrained to the range 1..4 after normalization.
- Blocking copies from non-DMA tensors are rejected before enqueue.
- `Sync()` waits across all streams through `PluggableDeviceUtil::SyncAll()`.

### Todos

- Existing TODOs note that stream-group factory may move to common runtime device code, GPU-named environment variables should become pluggable-device callbacks, and accelerator info naming may need cleanup.

### Docs References

The public docs establish why pluggable devices exist, but stream-group behavior and priority propagation are internal runtime details documented by source and tests.

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow documents pluggable devices as plugin packages that install alongside TensorFlow and communicate via C APIs. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |
| Public examples show pluggable devices appear as TensorFlow devices and work in eager and graph-style use. | L150-L168 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

This implementation is wired by the factory and the experimental StreamExecutor C adapter.

| Finding | Citations | Source Path |
|---|---|---|
| `StreamGroupFactory` shares stream groups for matching physical device and stream group id, because pluggable-device allocators are per device. | L72-L91 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| Stream creation uses optional priority for compute, host-to-device, device-to-host, and device-to-device streams. | L95-L152 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| `Init()` obtains the executor, event manager, stream group, device context, accelerator info, and platform device id mapping. | L203-L234 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| Optional thread-pool modes still come from GPU-named environment variables and can use global, private, or shared thread pools. | L236-L283 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| Compute, async compute, sync, allocator, tensor proto, and same-device copy behavior are implemented here. | L286-L479 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |
| The StreamExecutor C adapter converts optional priority into `SP_StreamOptions` before invoking plugin stream creation. | L372-L382 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |

## Cross-Repo References

No adjacent repository is configured for this file. Its external-device boundary is represented by TensorFlow's pluggable device plugin ABI and same-checkout StreamExecutor interfaces.

| Finding | Citations | Source Path |
|---|---|---|
| `PluggableDevice` includes StreamExecutor, pluggable-device context/init/state/util, and TensorFlow runtime headers from the TensorFlow checkout. | L30-L68 | [pluggable_device.cc](../../../../../../../../tensorflow/tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc) |

## Update History

- 2026-05-18: Created file-level onboarding for PluggableDevice implementation after stream priority propagation was added
