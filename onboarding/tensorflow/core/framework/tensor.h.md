# tensor.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/tensor.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `Tensor` (L60-L450) — the core data container for all TensorFlow computations. A Tensor is a ref-counted handle to an underlying `TensorBuffer` that stores a multi-dimensional array of a single DataType. Provides typed accessors (`flat<T>()`, `matrix<T>()`, `tensor<T, N>()`), slicing via `Slice()`, alignment guarantees, and serialization to/from `TensorProto`. This is the C++ counterpart to `tf.Tensor` in Python — almost every op's input and output is a Tensor.

## Code Commentary

### Logic

**Tensor (L60-L450):** Ref-counted handle with:

**Construction and shape:**
- `Tensor(dtype, TensorShape)` — Allocates an uninitialized tensor. Use `flat<T>()` or `vec<T>()` after.
- `Tensor(dtype, TensorShape, TensorBuffer*)` — Adopts an existing pre-allocated buffer. Takes ownership.
- `dtype()` — Returns `DataType` (DT_FLOAT, DT_INT32, DT_STRING, etc.). Immutable.
- `shape()` — Returns `TensorShape` (rank + dimensions). Immutable.
- `dims()` / `dim_size(d)` — Rank and per-dimension size.
- `NumElements()` — Total number of elements (product of all dims).
- `TotalBytes()` — Total memory in bytes (NumElements × DataTypeSize).
- `IsInitialized()` — True if a buffer is allocated.

**Typed access:**
- `flat<T>()` (L180) — Returns `typename TTypes<T>::Flat` (1D Eigen tensor map). CHECK-fails on dtype mismatch.
- `vec<T>()` — 1D view; CHECK-fails if rank != 1.
- `matrix<T>()` — 2D view; CHECK-fails if rank != 2.
- `scalar<T>()` — Scalar view; CHECK-fails if NumElements != 1.
- `tensor<T, N>()` — N-dimensional Eigen tensor map.
- `flat_inner_dims<T>(start_dim)` / `flat_outer_dims<T>(end_dim)` — Flattens inner/outer dimensions.
- `unaligned_flat<T>()` — Same as flat() but without alignment CHECK.

**Copy and slice:**
- `CopyFrom(tensor, shape)` — Deep copy with optional reshape.
- `Slice(start_dim, slice_spec)` — Returns a sub-tensor sharing the same buffer. No copy.
- `SubSlice(indices)` — Index-based slice into leading dimensions.

**Serialization:**
- `AsProtoField(&proto)` — Writes tensor data into TensorProto.
- `AsProtoTensorContent()` — Writes as raw bytes (more efficient than proto field).
- `FromProto(tensor_proto)` — Creates a Tensor from TensorProto.

**TensorBuffer (L470-L550):** The underlying data buffer. Ref-counted. Holds an `Allocator*` for deallocation. When the ref count hits zero, calls `Allocator::DeallocateRaw()`. `TensorBuffer::data()` returns `void*` — type-safety is at the Tensor level, not the buffer level.

### Conventions

- **Ref-counted handle** — Copying a `Tensor` is cheap (increments ref count). The underlying buffer is shared.
- **Eigen integration** — `flat<T>()` and friends return Eigen tensor maps backed by the TensorBuffer data pointer.
- **CHECK-fail on type mismatch** — `flat<float>()` on a DT_INT32 tensor kills the process. Always validate dtype first.
- **Copy-on-write** — `SubSlice` returns a shared view; mutate the result via the parent tensor.

### Invariants And Boundaries

- Tensor::dtype and Tensor::shape are immutable after construction.
- The underlying buffer is aligned to `EIGEN_MAX_ALIGN_BYTES`. Unaligned access is undefined behavior.
- `TensorSlice` carries shape/dtype metadata only, no data — attempting to read data from a TensorSlice produces garbage.
- Multiple Tensor objects can share the same TensorBuffer; `mutate()` calls that resize the buffer break sharing (copy-on-write).- **FATAL on type mismatch** — Calling `tensor.flat<float>()` when `tensor.dtype()` is DT_DOUBLE triggers LOG(FATAL) and kills the process. Always check dtype before accessing typed views.
- **TensorSlice vs Tensor** — TensorSlice only carries shape/dtype metadata, no data. Attempting to read data from a TensorSlice produces garbage.
- **Alignment is platform-dependent** — `EIGEN_MAX_ALIGN_BYTES` varies by platform. Custom allocators must match.
- **Buffer sharing** — Multiple Tensor objects can share the same TensorBuffer. Mutations via one Tensor affect all others.
- **String tensor caveat** — String tensors store offsets in a separate `uint32` array. Direct memcpy of raw bytes into a string tensor destroys the offset table.
- **TensorBuffer does not own device memory** — TensorBuffer holds an `Allocator*`. When the refcount hits zero, it calls `Allocator::DeallocateRaw`. The allocator must outlive the tensor.

### Todos

No known TODOs specific to this file.

## Docs References

TensorFlow's public documentation covers the Python-side tensor API extensively, which wraps this C++ implementation.

| Finding | Citations | Source Path |
|---|---|---|
| tf.Tensor Python API — shape, dtype, numpy conversion | — | [TF API: tf.Tensor](https://www.tensorflow.org/api_docs/python/tf/Tensor) |
| Tensor slicing and indexing | — | [TF Guide: Tensor Slicing](https://www.tensorflow.org/guide/tensor_slicing) |

## Repo-Internal References

Tensor is the fundamental data container consumed by every op, executor, and serialization path.

| Finding | Citations | Source Path |
|---|---|---|
| TensorShape defines the rank + dimensions | TensorShape | [`tensorflow/core/framework/tensor_shape.h`](tensor_shape.h) |
| OpKernelContext::input/allocate_output use Tensor | OpKernelContext | [`op_kernel.h`](op_kernel.h.md) |
| TensorProto for serialization/deserialization | TensorProto | [`tensorflow/core/framework/tensor.proto`](tensor.proto) |
| Eigen::TensorMap backs the typed accessors | Eigen integration | `third_party/eigen3` |
| Allocator for buffer allocation/deallocation | Allocator | [`tensorflow/core/framework/allocator.h`](allocator.h) |
| Rendezvous transports Tensors between devices | Rendezvous | [`rendezvous.h`](rendezvous.h.md) |

## Cross-Repo References

No meaningful cross-repo references found. Eigen is a vendored dependency.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added traps section for type mismatch FATAL, TensorSlice confusion, alignment, buffer sharing, string tensor encoding.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
