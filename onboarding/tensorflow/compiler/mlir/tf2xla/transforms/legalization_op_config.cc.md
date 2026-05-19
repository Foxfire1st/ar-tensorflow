# legalization_op_config.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/mlir/tf2xla/transforms/legalization_op_config.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/mlir/tf2xla/transforms/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/mlir/tf2xla/transforms/overview.md`](overview.md)

## Purpose

Configuration for which TensorFlow MLIR ops use the MLIR implementation in the
TF2XLA bridge. For the check-numerics path, it explicitly lists
`TF::CheckNumericsOp` among ops legalized through the old bridge using
`MlirXlaOpKernel`.

## Code Commentary

### Logic

- Builds a set of TensorFlow dialect op `TypeID`s.
- Comments distinguish ops that should use MLIR due to performance from ops that
  are legalized in the old bridge through `MlirXlaOpKernel`.
- `TF::CheckNumericsOp` is included in the `MlirXlaOpKernel` group.

### Conventions

- Use this file to decide whether a TensorFlow dialect op is expected to route
  through MLIR implementation support in TF2XLA.
- Use TableGen op definitions for op semantics and TF2XLA kernel registration
  for op-name-to-kernel wiring.

### Invariants And Boundaries

- This is route selection/configuration, not the rewrite pattern body.
- Inclusion here does not prove the final runtime check is preserved; XLA
  registry policy can still allow elision for Assert/CheckNumerics.

### Todos

No local TODOs in the `CheckNumericsOp` config block.

## Docs References

No external documentation proves this internal legalization configuration. Use
source citations.

| Finding | Citations | Source Path |
|---|---|---|
| TF2XLA legalization configuration is internal TensorFlow compiler behavior. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `TF::CheckNumericsOp` appears in the group described as old-bridge ops using `MlirXlaOpKernel`. | L43-L50 | `tensorflow/compiler/mlir/tf2xla/transforms/legalization_op_config.cc` |
| The TF2XLA kernel registration maps `CheckNumerics` to `MlirXlaOpKernel`. | L16-L22 | `tensorflow/compiler/tf2xla/kernels/check_numerics_op.cc` |
| The TensorFlow dialect TableGen definition gives `TF_CheckNumericsOp` must-execute and no-constant-fold traits. | L2109-L2138 | `tensorflow/compiler/mlir/tensorflow/ir/tf_generated_ops.td` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

