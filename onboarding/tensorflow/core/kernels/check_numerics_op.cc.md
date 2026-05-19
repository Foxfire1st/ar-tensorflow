# check_numerics_op.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/kernels/check_numerics_op.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

CPU and GPU runtime kernels for `CheckNumerics` and `CheckNumericsV2`. This is
the file that actually scans tensor values in non-XLA runtime execution and
raises `InvalidArgumentError` when NaN or Inf values are detected.

## Code Commentary

### Logic

- CPU kernels pass the input tensor through to the output, scan flat tensor data,
  and set an `InvalidArgumentError` when finite checks fail.
- GPU kernels are async: they allocate an anomaly indicator buffer, launch GPU
  check kernels, copy indicators back to host, then set an error from the event
  callback.
- V2 variants specialize anomaly detection and messages but use the same kernel
  registration pattern.
- The bottom of the file registers CPU kernels for half, bfloat16, float, and
  double, plus GPU kernels when CUDA or ROCm is enabled.

### Conventions

- The output is the original input tensor unless an error status is set.
- Message text is used as an error prefix.
- GPU path must keep tensor references alive until the async callback has read
  anomaly state.

### Invariants And Boundaries

- This file is runtime kernel behavior. XLA compiled behavior is routed through
  compiler registrations and may elide checks.
- The op definition controls statefulness and dtype constraints; this file
  supplies device-specific kernels.

### Todos

No local TODOs in the kernel registration hot path.

## Docs References

TensorFlow public docs describe the user-facing effect of `check_numerics`; this
file is the source proof for CPU/GPU kernel mechanics.

| Finding | Citations | Source Path |
|---|---|---|
| `tf.debugging.check_numerics` reports invalid numeric values and otherwise returns the input tensor. | — | [TensorFlow check_numerics API](https://www.tensorflow.org/api_docs/python/tf/debugging/check_numerics) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The CPU kernel forwards input to output, scans for NaN/Inf, and sets `InvalidArgumentError` on failure. | L80-L112 | `tensorflow/core/kernels/check_numerics_op.cc` |
| The GPU kernel launches device checks, copies anomaly indicators to host, and sets `InvalidArgumentError` in the callback. | L193-L304 | `tensorflow/core/kernels/check_numerics_op.cc` |
| CPU and GPU registrations bind `CheckNumerics` and `CheckNumericsV2` to concrete device/type kernels. | L365-L400 | `tensorflow/core/kernels/check_numerics_op.cc` |
| The op definitions mark both ops stateful and preserve input shape. | L1387-L1402 | `tensorflow/core/ops/array_ops.cc` |

## Cross-Repo References

No sibling memory repo is configured. GPU support depends on TensorFlow's CUDA or
ROCm build configuration inside the same checkout.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

