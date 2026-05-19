# numerics.py

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/python/ops/numerics.py` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Python helpers that connect floating-point tensors to `CheckNumerics`. This file
is relevant when a question is about v1 graph-wide numeric checks or
`assert_all_finite`, not when it is about the generated
`tf.debugging.check_numerics` op wrapper itself.

## Code Commentary

### Logic

- `verify_tensor_all_finite_v2` converts the input to a tensor, colocates with
  it, calls `array_ops.check_numerics`, and returns the original tensor through
  a control dependency.
- `add_check_numerics_ops` is graph-mode only and scans the default graph for
  half, float, and double outputs.
- For each eligible output, the helper creates a `check_numerics` op and chains
  dependencies so producer checks run before consumer checks.
- The helper refuses eager execution and control-flow graphs.

### Conventions

- Use this file for helper APIs. The direct generated `check_numerics` API is
  exposed through generated op wrappers and registered in `array_ops.py`.
- Error behavior ultimately comes from the core kernel and op definition.

### Invariants And Boundaries

- `add_check_numerics_ops` depends on graph operation order.
- The file does not define the `CheckNumerics` op or kernel; it inserts uses of
  that op into Python graph code.

### Todos

No local TODOs in the checked numeric helper path.

## Docs References

TensorFlow public docs describe `tf.debugging.check_numerics` as checking for
NaN and Inf values and returning the input tensor when values are finite.

| Finding | Citations | Source Path |
|---|---|---|
| `tf.debugging.check_numerics` is the public numeric-check API. | — | [TensorFlow check_numerics API](https://www.tensorflow.org/api_docs/python/tf/debugging/check_numerics) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `verify_tensor_all_finite_v2` calls `array_ops.check_numerics` and returns the checked input with dependencies. | L50-L78 | `tensorflow/python/ops/numerics.py` |
| `add_check_numerics_ops` is graph-mode only and rejects eager execution. | L81-L113 | `tensorflow/python/ops/numerics.py` |
| The graph-wide helper scans graph operations, checks float outputs, and creates `array_ops.check_numerics` dependencies. | L115-L131 | `tensorflow/python/ops/numerics.py` |
| `array_ops.py` registers `gen_array_ops.check_numerics` as a unary elementwise API. | L6810-L6814 | `tensorflow/python/ops/array_ops.py` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

