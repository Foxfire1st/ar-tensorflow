# c_api.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/c/c_api.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/c/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/c/overview.md`](overview.md)

## Purpose

Public C API header for TensorFlow — the stable, language-agnostic interface. Defines opaque types (`TF_Graph`, `TF_Session`, `TF_Tensor`, `TF_Status`, `TF_Buffer`) and function declarations (`TF_NewSession`, `TF_SessionRun`, `TF_NewTensor`, etc.) that form the canonical ABI boundary. All other language bindings (Python, C++, Go, Java) go through this interface.

## Code Commentary

### Logic

- **Opaque struct pattern** — All objects passed as pointers to opaque structs; implementation hidden from header consumers; consumers never access struct members directly.
- **TF_ prefix convention** — All public symbols prefixed with `TF_` to avoid namespace collisions.
- **Error model** — Functions accepting `TF_Status*` clear it on success or fill with error info on failure (TF_Code, TF_Message). No C++ exceptions cross the boundary.
- **Memory management** — All allocation via API: `TF_New*` creates, `TF_Delete*` frees. Custom deallocator callbacks allow external memory managers (e.g., numpy arrays).
- **Boolean convention** — Uses `unsigned char` (not C99 `bool`) for cross-compiler consistency.
- **Version API** — `TF_Version()` returns the runtime version string for compatibility checks.

### Conventions

- **Stable ABI contract** — All struct layouts and function signatures defined here are backward-compatible across TensorFlow releases.
- **Opaque handles** — No struct member access from outside the library; all state is managed through API functions.

### Invariants And Boundaries

- All `TF_*` handles must be freed with corresponding `TF_Delete*` — no garbage collection.
- `TF_Status` argument must be initialized by caller; API never takes ownership.
- `size_t` for byte sizes in caller's address space; `int` for array indices.
- The C API is ABI-stable across TensorFlow releases — backward compatible.- **TF_Status ownership** — Every TF_Status* returned by a C API function must be deleted with TF_DeleteStatus. Leaking status objects leaks error message strings. Use TF_NewStatus()/TF_DeleteStatus() pairs.
- **nullptr != safe** — Many functions accept nullptr TF_Status to ignore errors. But nullptr status causes undefined behavior if the function writes through it. Always pass a real TF_Status.
- **String encoding** — TF_StringEncodedSize and TF_StringEncode/Decode use on-disk TF format, not null-terminated C strings. Passing a C string to TF_StringEncode without proper length produces corrupted output.
- **Buffer ownership** — Many functions return TF_Buffer* which must be freed with TF_DeleteBuffer. The returned data pointer is invalid after deletion.
- **Thread safety varies** — TF_SessionRun is thread-safe. TF_Graph operation addition is NOT thread-safe. Graph construction must be single-threaded.
- **Version skew** — The C ABI is backward compatible but not forward compatible. Always match header and library versions.
- **Error recovery** — After TF_SessionRun returns an error, the session state may be inconsistent.

### Todos

No known TODOs specific to this file.

## Docs References

TensorFlow's C API is the primary language-binding boundary, documented in the public API reference.

| Finding | Citations | Source Path |
|---|---|---|
| C API overview and usage guide | — | [TF C API](https://www.tensorflow.org/api_docs/cc) |

## Repo-Internal References

This header is the canonical C ABI boundary. Every language binding routes through it.

| Finding | Citations | Source Path |
|---|---|---|
| Implementation wraps C++ core internals | c_api.cc | [`c_api.cc`](c_api.cc.md) |
| TF_Tensor type and memory lifecycle | tf_tensor.h | [`tf_tensor.h`](tf_tensor.h.md) |
| TF_Status error codes | tf_status.h | [`tensorflow/c/tf_status.h`](../../../tensorflow/c/tf_status.h) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template — added Governing Overview, Logic, Conventions, Invariants And Boundaries, Docs References, and Cross-Repo sections.
- 2026-05-17: Initial file-level onboarding from full-bootstrap.
