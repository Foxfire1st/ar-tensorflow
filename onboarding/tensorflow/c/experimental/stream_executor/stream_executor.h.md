# stream_executor.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/c/experimental/stream_executor/stream_executor.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | `72ec73a31c7094d6a0353690fb583efd70d74e60` |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |
| governingOverview | `tensorflow/c/overview.md` |

## Governing Overview

This file is governed by the C API overview: [`tensorflow/c/overview.md`](../../overview.md).

## Purpose

This header is the experimental C ABI for StreamExecutor plugins. It defines the opaque handles, versioned structs, callback tables, and `SE_InitPlugin` entrypoint that let external device plugins describe platforms, devices, streams, memory, events, timers, and host callbacks to TensorFlow without exposing TensorFlow's C++ runtime types directly.

## Code Commentary

### Logic

The header starts by defining the contract and naming conventions for `SE_*` structs filled by TensorFlow core and `SP_*` structs filled by plugins. `struct_size` is the compatibility guard on every extensible struct, and `ext` fields are reserved for plugin or future extension state.

`SP_StreamOptions` is the stream-creation options payload. It currently carries `has_priority` and `priority`, so option-aware stream creation must check the boolean before interpreting the integer.

`SP_StreamExecutor` is the central plugin callback table. It groups allocation callbacks, stream callbacks, event callbacks, timer callbacks, copy operations, synchronization, memset, host callback, and the option-aware `create_stream_with_options` hook.

Platform registration is represented by `SP_Platform`, `SP_PlatformFns`, and `SE_PlatformRegistrationParams`. A plugin implements `SE_InitPlugin`, fills platform metadata and callback tables, and hands those tables to TensorFlow for validation and registration.

### Conventions

- `SE_` means TensorFlow core generally fills the struct; `SP_` means the plugin generally fills it.
- `struct_size` is always explicit and sits outside the normal ownership prefix rule.
- Plugin-owned fields should be cleaned up through the matching destroy callback.
- Platform and device type names are C strings that must remain valid beyond initialization.

### Invariants And Boundaries

- This API is explicitly under active development and does not promise full versioning stability yet.
- `SP_StreamOptions.has_priority` gates `SP_StreamOptions.priority`; consumers must not treat the priority integer as meaningful when the boolean is false.
- `create_stream` remains the basic callback, while `create_stream_with_options` is the option-aware extension point used by current StreamExecutor priority plumbing.
- This header is ABI declaration only; the C++ adapter and validation logic live in `stream_executor.cc`.

### Todos

- The header still references the TensorFlow community versioning strategy and says the API lacks full versioning guarantees.
- Future fields should extend structs through `struct_size` rather than changing existing field meanings.

### Docs References

TensorFlow's public plugin documentation explains the high-level pluggable device model: device support is distributed as plugin packages and communicates with the TensorFlow binary through C APIs. It does not document this internal `SP_*` StreamExecutor struct surface in detail.

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow describes pluggable devices as external plugin packages that use C APIs to communicate with TensorFlow. | L140-L141 | [TensorFlow GPU device plugins](https://www.tensorflow.org/install/gpu_plugins) |

## Repo-Internal References

This file is paired with `stream_executor.cc`, which validates the structs and adapts plugin callbacks into C++ StreamExecutor objects. The priority-related stream option is exercised by the StreamExecutor tests.

| Finding | Citations | Source Path |
|---|---|---|
| The header states the active-development status, `SE_`/`SP_` ownership convention, `struct_size` rule, and extension field convention. | L23-L43 | [stream_executor.h](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.h) |
| `SP_StreamOptions` carries optional stream priority through `has_priority` and `priority`. | L97-L105 | [stream_executor.h](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.h) |
| `SP_StreamExecutor` defines plugin callbacks for memory, streams, events, timers, copies, synchronization, host callbacks, and option-aware stream creation. | L246-L435 | [stream_executor.h](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.h) |
| Platform registration is declared through `SP_Platform`, `SP_PlatformFns`, and `SE_PlatformRegistrationParams`, ending in the plugin-provided `SE_InitPlugin` entrypoint. | L446-L546 | [stream_executor.h](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.h) |
| The C++ adapter converts optional StreamExecutor priority into `SP_StreamOptions` before creating a stream. | L372-L382 | [stream_executor.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor.cc) |
| The test confirms option-aware stream creation receives a priority and stores it on the test stream. | L303-L326 | [stream_executor_test.cc](../../../../../../../../tensorflow/tensorflow/c/experimental/stream_executor/stream_executor_test.cc) |

## Cross-Repo References

No meaningful adjacent-repository reference is configured for this file. The C ABI is implemented inside the TensorFlow checkout and adapts to the vendored XLA StreamExecutor API through same-workspace includes in `stream_executor.cc`.

| Finding | Citations | Source Path |
|---|---|---|
| No cross-repo dependency is configured for this memory repo; StreamExecutor integration is handled inside the TensorFlow checkout. | n/a | n/a |

## Update History

- 2026-05-18: Created file-level onboarding for the experimental StreamExecutor C ABI header after the pluggable-device stream priority refresh
