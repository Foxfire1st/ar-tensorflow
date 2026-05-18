# c_api.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/c/c_api.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/c/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/c/overview.md`](overview.md)

## Purpose

Implementation of the C API (~2900 lines). Translates between C opaque types (`TF_Graph`, `TF_Session`, `TF_Tensor`, `TF_Status`) and C++ internals (`Graph`, `DirectSession`, `Tensor`, `Status`). Every public C API function declared in `c_api.h` is implemented here as a thin bridge to the C++ core. This file is the boundary that makes TensorFlow usable from C, C++, Go, Java, Rust, and every other language with C FFI support.

## Code Commentary

### Logic

- **Bridge pattern** — Each C API function (e.g., `TF_NewSession`) constructs or calls the corresponding C++ object (e.g., `NewSession`).
- **Handle mapping** — Opaque `TF_*` pointers are cast from/to internal C++ object pointers. The cast happens in the implementation, invisible to API consumers.
- **Status translation** — C++ `Status` objects are converted to `TF_Status` with error code and message string.
- **Buffer management** — `TF_Buffer` wraps `tensorflow::StringPiece` with ref-counting for safe cross-language ownership.
- **Graph import/export** — `TF_GraphImportGraphDef` and `TF_GraphToGraphDef` serialize between protobuf and C-visible byte buffers.

### Conventions

- **Thin wrappers** — Each function is as thin as possible; heavy logic lives in the C++ core.
- **No exceptions** — All errors are returned through `TF_Status`; C++ exceptions are caught and converted.
- **Ref-counting** — Internal objects are ref-counted; the C API surface manages ref counts through Create/Delete pairs.

### Invariants And Boundaries

- Every `TF_New*` must be paired with `TF_Delete*`. No garbage collection.
- Opaque pointers are never dereferenced by API consumers.
- The C ABI is the stable contract; internal C++ implementations can change freely.
- Error status must be checked after every API call.- **2900+ lines of bridging code** — Every function is a thin wrapper, but errors in the translation layer (wrong cast, missed null check) cause crashes deep in TF internals.
- **Internal handle invalidation** — Many operations (TF_ImportGraphDef, TF_GraphSetTensorShape) invalidate previously returned pointers. The documentation states this; not all callers read the documentation.
- **Reentrancy** — C API functions call back into Python via registered callbacks. If Python code then calls C API functions on the same graph/session during a callback, deadlocks or use-after-free results.
- **Eager-to-graph mode conversion** — The C API works mostly in graph mode. Mixing C API graph construction with eager Python execution causes confusing errors.
- **Tensor data lifetime** — TF_TensorData() returns a raw pointer to an internal buffer. This pointer is valid only until the next TF_DeleteTensor or TF_TensorMaybeMove. Holding it across C API calls is undefined behavior.
- **Prefix scoping** — TF_ImportGraphDef with a `prefix` creates new op names. This must match the prefix used in TF_SessionRun feed/fetch names. Mismatch results in a silent empty run.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. The C API implementation is an internal bridge not covered by public TF documentation.

## Repo-Internal References

This file is the implementation behind the public C API. Every language binding routes through functions defined here.

| Finding | Citations | Source Path |
|---|---|---|
| Header declaring the C API surface | c_api.h | [`c_api.h`](c_api.h.md) |
| TF_Tensor implementation | tf_tensor.h | [`tf_tensor.h`](tf_tensor.h.md) |
| Graph, Session, Status are the C++ objects being bridged | core classes | [`tensorflow/core/public/session.h`](../core/public/session.h.md) |
| DirectSession is the local session implementation | DirectSession | [`tensorflow/core/common_runtime/direct_session.h`](../core/common_runtime/direct_session.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17: Initial file-level onboarding from full-bootstrap.
