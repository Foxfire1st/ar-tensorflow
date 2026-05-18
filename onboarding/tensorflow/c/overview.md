# C API Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/c/` |
| onboardingRoute | `tensorflow/c/overview.md` |
| parentOverview | [`overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

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

## Load-Bearing Files

| File | Role | Onboarding |
|---|---|---|
| `c_api.h` | Public C API surface (graphs, sessions, tensors) | planned |
| `c_api.cc` | C API implementation (wraps C++ core) | planned |
| `tf_tensor.h` | TF_Tensor type and accessors | planned |

## Local Invariants And Traps

- **All TF_* handles must be freed** — TF_Delete* required; no garbage collection
- **C API is versioned and ABI-stable** — backward compatible across releases
- **Internal C++ exceptions caught** — exposed as TF_Status with TF_Code
- **TF_Tensor data access** — must use TF_TensorData() and TF_TensorByteSize(); data may be on device
- **Thread safety** — TF_Session is thread-safe for concurrent Run() calls

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Wraps tensorflow/core/ C++ internals | c_api.cc delegates to Session, Graph, etc. | `tensorflow/core/public/session.h`, `tensorflow/core/graph/graph.h` |
| Python bindings call through this layer | pywrap_tensorflow.py → c_api | `tensorflow/python/client/pywrap_tensorflow.py` |
| C++ API (cc/) builds on top of this | ClientSession wraps TF_Session | `tensorflow/cc/client/client_session.h` |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TF C API reference | — | https://www.tensorflow.org/api_docs/cc (C API section) |

## File-Level Onboarding Map

(3 files — all planned per coverage plan wave 5)

## Child Overviews

None.

## How To Use This Area

1. Read `c_api.h` — all public functions and their contracts
2. Read `c_api.cc` — implementation patterns
3. Read `tf_tensor.h` — tensor lifecycle management

## Needs Verification

- None

## Update History

- 2026-05-17: Initial route-local overview from full-bootstrap
