# File Card — tensorflow/core/framework/tensor.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/framework/tensor.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/framework/tensor.h.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | complex |
| Suggested wave | onboarding-wave-001 |
| suggested action | update onboarding |
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| localArea | core |

## Why This File Matters

Tensor is the fundamental data type: N-dimensional typed array with ref-counted TensorBuffer. Every computation in TF operates on Tensors.

## What The Worker Must Explain

- Purpose: multi-dimensional typed array with ref-counted backing buffer
- Main logic: shape, dtype, data access (flat, matrix, tensor, scalar, vec), TensorBuffer ref-counting
- Non-obvious behavior: Tensor is a cheap handle (copy = refcount increment); aliasing via Slice()
- Invariants: TensorBuffer ownership via RefCounted; memory can be CPU or GPU
- Interfaces: Device, Allocator, OpKernelContext (input/output tensors)

## Known Traps

- Tensor::matrix<>() etc. type-check at runtime; wrong type = crash
- Sliced tensors share the same TensorBuffer; mutation affects all slices
- Unaligned access on some hardware (ARM)

## Done When

- Onboarding updated. Commit hash backfilled.
