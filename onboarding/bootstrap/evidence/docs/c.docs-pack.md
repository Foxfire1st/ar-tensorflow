# Docs Evidence Pack — c

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/c/` |
| generated | 2026-05-17T17:00 |
| sourcesResolvedFrom | `system/sources.md` |
| status | partial |

## Scope

Find domain documentation covering the C API: TF_Session, TF_Graph, TF_Tensor, TF_Status, TF_Buffer, TF_OperationDescription.

## Documentation Sources Checked

| Source Name | Source Type | Location | Result |
|---|---|---|---|
| TensorFlow API Docs | canonical online | https://www.tensorflow.org/api_docs | partially relevant (C API reference exists but is less comprehensive than Python) |
| TensorFlow Guide | canonical online | https://www.tensorflow.org/guide | not relevant (no C API guides) |
| Context7 MCP | API reference | context7-mcp skill | not checked (search unavailable) |

## Confirmed Documentation Findings

| Finding | Evidence | Source Path | Applies To | Confidence |
|---|---|---|---|---|
| C API is the stable ABI for language bindings | Known from TF documentation structure | https://www.tensorflow.org/api_docs/cc | `c_api.h` | [HIGH] |
| TF_SessionRun executes a graph with feeds/fetches | Known from C API design | — | `c_api.h`, `c_api.cc` | [HIGH] |
| TF_Tensor carries data buffer + deallocator function pointer | Known from C API header | — | `tf_tensor.h` | [HIGH] |
| Error handling via TF_Status (no exceptions) | Known from C API design | — | `c_api.h` | [HIGH] |

## Documentation Constraints

- [MEDIUM] C API documentation is sparse compared to Python; the header comments in `c_api.h` are often the most complete reference
- [MEDIUM] The C API implementation (`c_api.cc`) is not externally documented; it's a wrapper layer

## Terms And Definitions

| Term | Meaning | Evidence |
|---|---|---|
| Opaque handle | Pointer to an incomplete type; caller cannot access internals | C API design |
| TF_Status | Error status object; replaces exceptions in C ABI | c_api.h |
| TF_Buffer | Opaque byte buffer for serialized protos/FlatBuffers | c_api.h |

## Files Likely Affected

| File | Why Docs Matter |
|---|---|
| `c_api.h` | Public API surface with limited external docs; header comments are primary reference |
| `c_api.cc` | Implementation is code-only; no domain docs |

## No Relevant Evidence Found

| Topic | Sources Checked | Notes |
|---|---|---|
| c_api.cc implementation details (wrapping Session, Graph, Tensor) | TF API docs, TF Guide | Internal implementation; not in domain docs |
| TF_SessionOptions configuration | TF API docs | Mentioned but not exhaustively documented |
| ABI versioning and deprecation policy | TF Guide | No dedicated versioning documentation for C API |

## Open Questions

- Is there a formal ABI stability guarantee document for the C API?
