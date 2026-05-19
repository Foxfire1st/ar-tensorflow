# array_ops.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/ops/array_ops.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Core C++ operation definitions for TensorFlow array ops. For the numeric-check
hot path, this file defines the `CheckNumerics` and `CheckNumericsV2` op
contracts: inputs, outputs, dtype constraints, statefulness, message attribute,
and unchanged-shape inference.

## Code Commentary

### Logic

- `REGISTER_OP("CheckNumerics")` accepts a floating tensor and string message,
  returns the same dtype, is stateful, and preserves shape.
- `REGISTER_OP("CheckNumericsV2")` has the same basic signature and shape
  behavior.
- Kernel registration and actual value scanning live in
  `tensorflow/core/kernels/check_numerics_op.cc`.

### Conventions

- Op definitions describe graph-level contract; device behavior belongs to
  kernels.
- `SetIsStateful()` is load-bearing for optimizers and compiler decisions
  because this op represents a check with error side effects.

### Invariants And Boundaries

- Dtype set is limited to `bfloat16`, `half`, `float`, and `double`.
- Shape function is unchanged-shape; the op is semantically a checking
  pass-through.

### Todos

No local TODOs in the `CheckNumerics` op definition block.

## Docs References

TensorFlow public docs describe the user-facing `check_numerics` behavior. This
file is the source proof for the graph op contract.

| Finding | Citations | Source Path |
|---|---|---|
| `tf.debugging.check_numerics` reports invalid numeric values and otherwise returns the input tensor. | — | [TensorFlow check_numerics API](https://www.tensorflow.org/api_docs/python/tf/debugging/check_numerics) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `CheckNumerics` and `CheckNumericsV2` are registered as stateful unchanged-shape floating tensor ops. | L1387-L1402 | `tensorflow/core/ops/array_ops.cc` |
| Runtime kernels for these op names live in `check_numerics_op.cc`. | L365-L400 | `tensorflow/core/kernels/check_numerics_op.cc` |
| MLIR generated op definitions mirror the `CheckNumerics` user-facing InvalidArgument behavior. | L2109-L2138 | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

