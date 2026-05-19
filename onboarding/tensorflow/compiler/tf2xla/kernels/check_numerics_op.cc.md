# check_numerics_op.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/tf2xla/kernels/check_numerics_op.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/overview.md`](../../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../../overview.md)

## Purpose

TF2XLA kernel registration for `CheckNumerics`. It maps the TensorFlow op name
to `MlirXlaOpKernel`, which means XLA handling goes through the MLIR-backed
legalization path instead of the normal core runtime kernel.

## Code Commentary

### Logic

- Includes `mlir_xla_op_kernel.h`.
- Registers the XLA op name `CheckNumerics` with `MlirXlaOpKernel`.
- Contains no local lowering code; all behavior is delegated to the MLIR XLA op
  kernel machinery and the legalization configuration.

### Conventions

- Treat this file as registration-only.
- Follow `MlirXlaOpKernel` and TF2XLA MLIR transforms for actual lowering or
  elision behavior.

### Invariants And Boundaries

- This file registers `CheckNumerics`, not `CheckNumericsV2`.
- Runtime CPU/GPU checking behavior is not reused by XLA compiled execution.

### Todos

No local TODOs in this registration file.

## Docs References

The XLA public docs explain XLA as TensorFlow's compiler path, but this
registration is proven by TensorFlow source.

| Finding | Citations | Source Path |
|---|---|---|
| XLA is TensorFlow's accelerated compiler path. | — | [TensorFlow XLA](https://www.tensorflow.org/xla) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `CheckNumerics` is registered as an XLA op backed by `MlirXlaOpKernel`. | L16-L22 | `tensorflow/compiler/tf2xla/kernels/check_numerics_op.cc` |
| TF2XLA legalization config lists `TF::CheckNumericsOp` among ops legalized in the old bridge using `MlirXlaOpKernel`. | L43-L50 | `tensorflow/compiler/mlir/tf2xla/transforms/legalization_op_config.cc` |
| The XLA op registry comments say Assert and CheckNumerics can be clustered by eliding them because XLA does not natively support them. | L168-L170 | `tensorflow/compiler/tf2xla/xla_op_registry.h` |

## Cross-Repo References

No sibling memory repo is configured. XLA dependencies are vendored under this
TensorFlow checkout.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

