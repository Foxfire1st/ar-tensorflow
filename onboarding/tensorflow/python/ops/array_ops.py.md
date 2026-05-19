# array_ops.py

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/python/ops/array_ops.py` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Large Python array-op helper module. For `check_numerics`, the important local
fact is narrow: this file registers the generated `gen_array_ops.check_numerics`
op as a unary elementwise API. The op definition and runtime behavior are
elsewhere.

## Code Commentary

### Logic

- Many array helpers and Python conveniences live in this file.
- Near the end of the module, TensorFlow registers generated elementwise ops
  without bespoke Python wrappers.
- `gen_array_ops.check_numerics` is one of those registered APIs.

### Conventions

- Do not search this whole file for `CheckNumerics` semantics. It is only a
  Python API registration point for this benchmark path.
- Follow the generated op name into `tensorflow/core/ops/array_ops.cc` for the op
  definition and `tensorflow/core/kernels/check_numerics_op.cc` for runtime
  kernels.

### Invariants And Boundaries

- Registration here makes the generated op participate in Python dispatch, but
  does not change dtype, statefulness, shape, or kernel behavior.

### Todos

No local TODOs in the `check_numerics` registration block.

## Docs References

TensorFlow public docs describe the user-facing `tf.debugging.check_numerics`
API. This source file only wires generated Python dispatch.

| Finding | Citations | Source Path |
|---|---|---|
| `tf.debugging.check_numerics` is the public numeric-check API. | — | [TensorFlow check_numerics API](https://www.tensorflow.org/api_docs/python/tf/debugging/check_numerics) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `array_ops.py` registers `gen_array_ops.check_numerics` as a unary elementwise API. | L6810-L6814 | `tensorflow/python/ops/array_ops.py` |
| `numerics.py` consumes `array_ops.check_numerics` from higher-level numeric-check helpers. | L73-L78; L128-L131 | `tensorflow/python/ops/numerics.py` |
| The C++ op definition names `CheckNumerics` and `CheckNumericsV2` as stateful unchanged-shape ops. | L1387-L1402 | `tensorflow/core/ops/array_ops.cc` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

