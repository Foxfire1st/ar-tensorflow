# tf_generated_ops.td

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/mlir/overview.md`](../../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/mlir/overview.md`](../../overview.md)

## Purpose

Generated TensorFlow MLIR dialect operation definitions. For the check-numerics
hot path, this file defines `TF_CheckNumericsOp`, its traits, summary,
description, arguments, and result contract.

## Code Commentary

### Logic

- `TF_CheckNumericsOp` is declared as TensorFlow op `"CheckNumerics"`.
- Traits include `TF_MustExecute`, `TF_NoConstantFold`, and same operand/result
  type resolution.
- The description states that the op reports `InvalidArgument` for NaN or Inf
  and otherwise returns the input tensor.
- The nearby `TF_IsFiniteOp` definition is a pure elementwise finite predicate
  and is a useful comparison point, but it is not the same checking op.

### Conventions

- Treat this file as dialect op metadata used by generated MLIR code.
- Do not confuse MLIR op traits with core runtime kernel registration or XLA
  registry elision policy.

### Invariants And Boundaries

- `TF_MustExecute` and `TF_NoConstantFold` express MLIR-level side-effect and
  optimization constraints for the TensorFlow dialect op.
- Final TF2XLA behavior still depends on legalization config and XLA registry
  policy.

### Todos

No local TODOs in the `TF_CheckNumericsOp` definition block.

## Docs References

TensorFlow public docs describe the user-facing `check_numerics` behavior. This
file is the source proof for the TensorFlow MLIR dialect op definition.

| Finding | Citations | Source Path |
|---|---|---|
| `tf.debugging.check_numerics` reports invalid numeric values and otherwise returns the input tensor. | — | [TensorFlow check_numerics API](https://www.tensorflow.org/api_docs/python/tf/debugging/check_numerics) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `TF_CheckNumericsOp` declares the `CheckNumerics` op with must-execute and no-constant-fold traits. | L2109-L2115 | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |
| The op description includes NaN/Inf examples and `InvalidArgument` behavior. | L2112-L2138 | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |
| `TF_IsFiniteOp` is a pure finite predicate and is separate from the must-execute check op. | L7322-L7335 | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |
| TF2XLA legalization config lists `TF::CheckNumericsOp` in the old-bridge `MlirXlaOpKernel` group. | L43-L50 | `tensorflow/compiler/mlir/tf2xla/transforms/legalization_op_config.cc` |

## Cross-Repo References

No sibling memory repo is configured. MLIR and StableHLO dependencies are used
through TensorFlow's build graph.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

