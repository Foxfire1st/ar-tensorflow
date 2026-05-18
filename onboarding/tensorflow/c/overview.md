# C API Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/c/` |
| onboardingRoute | `tensorflow/c/overview.md` |
| parentOverview | [`overview.md`](../../overview.md) |
| lastUpdated | 2026-05-18T11:36:03+02:00 |
| lastVerifiedCommitHash | 72ec73a31c7094d6a0353690fb583efd70d74e60 |
| lastVerifiedCommitDate | 2026-05-17T18:09:55-07:00 |

## What This Area Is

The stable, language-agnostic C API for TensorFlow. This is the **canonical ABI boundary** — all other language bindings (Python via `pywrap`, C++, Go, Java, Rust) ultimately go through this C interface. Provides opaque `TF_*` handles for graph construction, session management, tensor manipulation, and eager execution. Memory management is explicit — all handles must be freed with `TF_Delete*` functions. Internal C++ exceptions are caught and exposed as `TF_Status` error codes.

## What Belongs Here

| Path | Role |
|---|---|
| `c_api.h` | Primary public C API: graphs, operations, sessions, tensors, attributes |
| `c_api.cc` | Implementation (~1500 LOC); wraps C++ core objects |
| `tf_tensor.h` | TF_Tensor type definition and accessors |
| `c_api_experimental.h` | Experimental/unstable API extensions |
| `version.h` | API version constants |
| `eager/` | Eager execution C API: TFE_Context, TFE_Op, TFE_TensorHandle |
| `experimental/stream_executor/` | Experimental C plugin ABI for registering StreamExecutor platforms, devices, streams, memory, events, timers, and callbacks |

## What Does Not Belong Here

| What | Where |
|---|---|
| Core C++ implementation | `tensorflow/core/` |
| C++ Scope DSL | `tensorflow/cc/` |
| Python bindings | `tensorflow/python/` |
| Go/Java/JS bindings | `tensorflow/go/`, `java/`, `js/` |

## Structures Found Here

| Structure | Purpose |
|---|---|
| TF_Graph | Opaque computation graph container |
| TF_OperationDescription | Builder for graph operations (set name, type, inputs, attributes) |
| TF_Operation | Completed graph node |
| TF_Tensor | N-dimensional typed array; access via TF_TensorData()/TF_TensorByteSize() |
| TF_Status | Error handling struct; TF_GetCode()/TF_Message() for details |
| TF_Session | Graph execution context; TF_SessionRun() for feeds/fetches |
| TF_Buffer | Generic byte buffer (for serialization) |
| TF_ImportGraphDefOptions | Graph import configuration |
| TFE_Context | Eager execution context |
| TFE_Op | Eager operation handle |
| TFE_TensorHandle | Eager tensor reference |
| SP_Platform / SP_PlatformFns | Plugin-supplied platform metadata and callbacks for StreamExecutor registration |
| SP_StreamExecutor | Plugin-supplied stream, event, memory, callback, and synchronization function table |
| SP_StreamOptions | Optional stream creation options, including virtual-device stream priority |

## Operating Model

All operations go through opaque `TF_*` handles. The C API wraps C++ core objects with stable signatures that don't expose internal types. Memory management is explicit — every `TF_New*()` has a corresponding `TF_Delete*()`. Internal errors propagate as `TF_Status`.

### Graph Construction
```
TF_NewGraph() → empty graph
TF_NewOperation(graph, "Add", "my_add") → op description
TF_SetAttrType(op, "T", TF_FLOAT)
TF_AddInput(op, input_tensor)
TF_FinishOperation(op, status) → adds node to graph
```

### Graph Execution
```
TF_NewSession(graph, options, status)
TF_SessionRun(session, feeds, fetches, targets, outputs, status)
→ results returned as TF_Tensor* handles
```

### Eager Execution
```
TFE_NewContext(options, status)
TFE_NewOp(context, "Add", status)
TFE_OpAddInput(op, handle)
TFE_Execute(op, output_handle, status)
```

### Experimental StreamExecutor Plugin Flow

`experimental/stream_executor/stream_executor.h` defines `SP_*` structs that plugins populate and TensorFlow validates with `struct_size` checks. `stream_executor.cc` adapts those C callbacks into StreamExecutor objects. Current stream creation builds `SP_StreamOptions` from the optional StreamExecutor priority and calls the option-aware stream creation path so pluggable device virtual-device priority can reach plugin streams.

## Load-Bearing Files

| File | Role | Onboarding |
|---|---|---|
| `c_api.h` | Public C API surface (graphs, sessions, tensors) | planned |
| `c_api.cc` | C API implementation (wraps C++ core) | planned |
| `tf_tensor.h` | TF_Tensor type and accessors | planned |
| `experimental/stream_executor/stream_executor.h` | Experimental plugin ABI surface, including `SP_StreamOptions` and `SP_StreamExecutor` | planned |
| `experimental/stream_executor/stream_executor.cc` | C-to-StreamExecutor adapter used by pluggable device platforms | planned |

## Local Invariants And Traps

- **All TF_* handles must be freed** — TF_Delete* required; no garbage collection
- **C API is versioned and ABI-stable** — backward compatible across releases
- **Internal C++ exceptions caught** — exposed as TF_Status with TF_Code
- **TF_Tensor data access** — must use TF_TensorData() and TF_TensorByteSize(); data may be on device
- **Thread safety** — TF_Session is thread-safe for concurrent Run() calls
- **Experimental StreamExecutor ABI is still active-development** — `struct_size` gates extension compatibility, but the header explicitly says it does not yet carry full versioning guarantees
- **Stream priority is optional** — `SP_StreamOptions.has_priority` must be checked before interpreting `priority`

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Wraps tensorflow/core/ C++ internals | c_api.cc delegates to Session, Graph, etc. | `tensorflow/core/public/session.h`, `tensorflow/core/graph/graph.h` |
| Python bindings call through this layer | `pywrap_tf_session.py` imports the pybind session module; `tf_session_wrapper.cc` includes `tensorflow/c/c_api.h` and binds many `TF_*` calls | `tensorflow/python/client/pywrap_tf_session.py`, `tensorflow/python/client/tf_session_wrapper.cc` |
| C++ API (cc/) builds on top of this | ClientSession wraps TF_Session | `tensorflow/cc/client/client_session.h` |
| Experimental StreamExecutor C API bridges plugins into TensorFlow's StreamExecutor and pluggable device runtime | C header defines `SP_StreamOptions`/`SP_StreamExecutor`; adapter turns optional priorities into option-aware stream creation | `tensorflow/c/experimental/stream_executor/stream_executor.h`, `tensorflow/c/experimental/stream_executor/stream_executor.cc`, `tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc` |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TF C API reference | — | https://www.tensorflow.org/api_docs/cc (C API section) |

## File-Level Onboarding Map

| Source File | Onboarding File | Status | Reason |
|---|---|---|---|
| `c_api.h` | `c_api.h.md` | exists | Public C API surface |
| `c_api.cc` | `c_api.cc.md` | exists | C API implementation |
| `tf_tensor.h` | `tf_tensor.h.md` | exists | Tensor ABI and lifecycle |
| `experimental/stream_executor/stream_executor.h` | `experimental/stream_executor/stream_executor.h.md` | exists | Experimental plugin ABI now carries stream options |
| `experimental/stream_executor/stream_executor.cc` | `experimental/stream_executor/stream_executor.cc.md` | exists | Adapter forwards stream priority into C stream creation |

## Child Overviews

None.

## How To Use This Area

1. Read `c_api.h` — all public functions and their contracts
2. Read `c_api.cc` — implementation patterns
3. Read `tf_tensor.h` — tensor lifecycle management
4. For pluggable device plugin behavior, read `experimental/stream_executor/stream_executor.h` with `experimental/stream_executor/stream_executor.cc`

## Needs Verification

- None

## Update History

- 2026-05-18: Added file-level onboarding map entries for experimental StreamExecutor header and adapter files
- 2026-05-18: Refreshed after StreamExecutor C API drift; added experimental stream option and pluggable device bridge context
- 2026-05-17: Initial route-local overview from full-bootstrap
