# tf_tensor.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/c/tf_tensor.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/c/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/c/overview.md`](overview.md)

## Purpose

Defines `TF_Tensor` — the C API's opaque tensor type and its management functions. `TF_Tensor` wraps the C++ `Tensor` class behind an opaque pointer, providing allocation (`TF_AllocateTensor`), creation from raw data (`TF_NewTensor`), data access (`TF_TensorData`), and lifecycle management (`TF_DeleteTensor`). Also defines string tensor encoding/decoding, byte size, element count, and dims queries. This is how every language binding (Python, Go, Java, Rust) moves tensor data across the C ABI boundary.

## Code Commentary

### Logic

- `TF_AllocateTensor(dtype, dims, num_dims, len)` — Allocates uninitialized device/host memory. Returns nullptr on failure.
- `TF_NewTensor(dtype, dims, num_dims, data, len, deallocator, deallocator_arg)` — Creates a tensor wrapping caller-owned data. The deallocator callback is invoked when the tensor is deleted.
- `TF_TensorData(tensor)` — Returns `void*` to the raw data buffer. Not type-safe; caller must check `TF_TensorType()` first.
- `TF_TensorType(tensor)` — Returns `TF_DataType` enum (TF_FLOAT, TF_INT32, etc.).
- `TF_NumDims(tensor)` / `TF_Dim(tensor, dim_index)` — Returns rank and per-dimension size.
- `TF_TensorByteSize(tensor)` — Total bytes of the data buffer.
- `TF_TensorElementCount(tensor)` — Number of logical elements (may differ from byte size for variable-length types like strings).
- `TF_StringEncode(src, dst_len, dst, dst_len, status)` / `TF_StringDecode` — Encode/decode individual string tensor elements using TF's on-disk format.

### Conventions

- **Opaque pointer** — `TF_Tensor*` is never dereferenced by API consumers.
- **Deallocator pattern** — Custom deallocators allow external memory managers (numpy, Java ByteBuffer) to own tensor memory.
- **No type safety at C level** — `TF_TensorData` returns `void*`; type checking is the caller's responsibility.

### Invariants And Boundaries

- TF_TensorData() pointer is valid until TF_DeleteTensor() or TF_TensorMaybeMove().
- String tensors use a TF-specific offset-table encoding; raw memcpy corrupts the table.
- Allocated tensors are aligned to EIGEN_MAX_ALIGN_BYTES.
- Deallocator callbacks must not call back into the TF C API (deadlock risk).- **TF_TensorData is not type-safe** — Returns void* regardless of tensor dtype. Casting to the wrong type reads garbage. Always check TF_TensorType() first.
- **TF_AllocateTensor allocates uninitialized memory** — Unless you memset or memcpy into the buffer, the tensor contains random bytes. TF_NewTensor() with raw data is safer.
- **ByteSize vs ElementCount** — TF_TensorByteSize returns total bytes; TF_TensorElementCount returns element count. Dividing ByteSize by sizeof(T) when the type is unknown produces wrong results for variable-length types (strings).
- **Alignment** — TF_AllocateTensor aligns to EIGEN_MAX_ALIGN_BYTES. Callers writing custom allocators must respect this alignment. Unaligned tensors cause SIGBUS on some architectures.
- **String tensor encoding** — String tensors use a TF-specific multi-byte encoding with offset tables. Direct memcpy of C strings into string tensor data corrupts the offset table. Use TF_StringEncode for each element.
- **Deallocator callback reentrancy** — The deallocator callback runs inside TF_DeleteTensor. Calling back into the TF C API from inside the deallocator may deadlock on internal mutexes.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. TF_Tensor is a C ABI type not covered by public TF documentation.

## Repo-Internal References

TF_Tensor is the C ABI tensor type used by c_api.h, language bindings, and the eager C API.

| Finding | Citations | Source Path |
|---|---|---|
| C API functions that create and consume TF_Tensor | c_api.h | [`c_api.h`](c_api.h.md) |
| C++ Tensor class that TF_Tensor wraps | Tensor class | [`tensor.h`](../core/framework/tensor.h.md) |
| Eager C API also uses TF_Tensor for TFE_TensorHandle | eager/c_api.h | `tensorflow/c/eager/c_api.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17: Initial file-level onboarding from full-bootstrap.
