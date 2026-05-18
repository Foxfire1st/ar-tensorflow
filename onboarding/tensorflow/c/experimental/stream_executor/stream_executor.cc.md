# stream_executor.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/c/experimental/stream_executor/stream_executor.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `72ec73a31c7094d6a0353690fb583efd70d74e60` |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |
| governingOverview | `tensorflow/c/overview.md` |

## Governing Overview

This file is governed by the C API overview: [`tensorflow/c/overview.md`](../../overview.md).

## Purpose

This file adapts the experimental StreamExecutor C API declared in `stream_executor.h` into TensorFlow's C++ StreamExecutor interfaces. It validates plugin-supplied structs, wraps `SP_*` handles in C++ `C*` classes, implements platform registration, and forwards core StreamExecutor calls into plugin callback tables.

## Code Commentary

### Logic

The adapter validates plugin structs before registration. `ValidateSPStreamExecutor()` requires all core memory, stream, event, copy, synchronization, callback, and memset hooks that TensorFlow expects to call; unified memory callbacks become mandatory only when the platform advertises unified-memory support.

`CStreamExecutor` is the main bridge from C++ StreamExecutor methods to `SP_StreamExecutor` callbacks. It wraps device memory allocation, host allocation, allocator stats, synchronization, copy operations, device description construction, events, streams, and memory allocator creation.

`CreateStream()` is the current priority-sensitive path. It translates `std::optional<std::variant<StreamPriority, int>>` into `SP_StreamOptions`, sets `has_priority`, casts the variant payload to `int`, and calls `CStream::Create(&options)`.

`CPlatform` owns the plugin-provided platform, callback tables, device functions, stream executor table, and timer functions. It uses an executor cache to create a `CStreamExecutor` per ordinal, and `InitStreamExecutorPlugin()` loads or calls `SE_InitPlugin`, validates the filled structs, and registers the new platform with `PlatformManager`.

### Conventions

- Adapter classes use the `C` prefix to signal a C-backed implementation of a C++ StreamExecutor concept.
- `OwnedTFStatus` and `StatusFromTF_Status` bridge C status callbacks into `absl::Status`.
- Plugin callback tables are validated before registration so runtime adapter methods can assume required function pointers exist.
- The adapter owns cleanup through plugin destroy callbacks rather than assuming C++ ownership of plugin internals.

### Invariants And Boundaries

- `CreateStream()` must preserve optional priority semantics: absent priority becomes `has_priority = false`; present priority is passed as an integer.
- `DescriptionForDevice()` reads through an existing cached executor; callers that need a description before the executor exists must create the executor first.
- `EnablePeerAccessTo()` remains unsupported for pluggable devices in this adapter.
- Plugin platform registration must validate platform metadata, platform callbacks, device callbacks, and stream executor callbacks before registration.

### Todos

- Existing TODOs note that `OwnedTFStatus` should be removed and that plugin `use_bfc_allocator` should eventually become available to `PluggableDeviceProcessState`.

### Docs References

TensorFlow's public plugin documentation covers the user/developer model at a high level. The detailed C-to-C++ StreamExecutor adapter behavior in this file is established by repository source and tests.

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow documents that pluggable device support is shipped as plugins and communicates with TensorFlow through C APIs. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

The header defines the ABI surface this adapter consumes, while the tests exercise option-aware stream creation and registration behavior.

| Finding | Citations | Source Path |
|---|---|---|
| The file declares itself as the implementation of core StreamExecutor classes in terms of the C API and contains platform registration for pluggable devices. | L15-L22 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| `ValidateSPStreamExecutor()` enforces the required callback table before TensorFlow relies on plugin callbacks. | L125-L159 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| `CStreamExecutor` wraps allocation, stats, synchronization, copy, device description, event, and stream creation behavior over plugin callbacks. | L209-L383 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| `CreateStream()` converts optional priority into `SP_StreamOptions` and creates the stream through the C adapter. | L372-L382 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| `CPlatform` owns plugin callback tables and registers a platform through `PlatformManager`. | L422-L552 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| The stream creation test sets `create_stream_with_options`, checks `has_priority`, and verifies the priority value survives destruction. | L303-L326 | [stream_executor_test.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor_test.cc) |

## Cross-Repo References

This file imports and implements vendored XLA StreamExecutor interfaces from the TensorFlow checkout. No separate adjacent repository is configured for memory updates.

| Finding | Citations | Source Path |
|---|---|---|
| The adapter includes XLA StreamExecutor headers as the C++ interface it implements, but those headers are consumed from the TensorFlow checkout. | L44-L58 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |

## Update History

- 2026-05-18: Created file-level onboarding for the C-to-StreamExecutor adapter after option-aware stream creation was added
