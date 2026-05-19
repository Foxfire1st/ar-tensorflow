# attributes.py

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/python/eager/polymorphic_function/attributes.py` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/python/overview.md`](../../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../../overview.md)

## Purpose

Central string constants for function attributes used by polymorphic functions.
For the XLA hot path, this file gives the exact attribute names that
`polymorphic_function.py` writes onto traced functions.

## Code Commentary

### Logic

- Defines TensorRT, control-flow, and XLA-related attribute keys.
- `XLA_COMPILE` is the graph/function attribute `_XlaMustCompile`.
- `XLA_COMPILE_OPTIONAL` and `XLA_SCOPE` are adjacent optional/scope attribute
  names, but `tf.function(jit_compile=True)` uses `XLA_COMPILE`.
- `POLYMORPHIC_FUNCTION_ALLOWLIST` starts below these constants and controls
  which attributes are accepted through this path.

### Conventions

- Keep string values synchronized with C++ and graph compiler consumers.
- Treat these as protocol names, not user-facing option names.

### Invariants And Boundaries

- This file declares names only; it does not decide when attributes are emitted
  or how compiler/device routes interpret them.
- Search for the constant name and the string value when tracing behavior across
  Python and compiler layers.

### Todos

No local TODOs in the XLA attribute block.

## Docs References

No external documentation proves these internal attribute string constants. The
source file is the direct evidence.

| Finding | Citations | Source Path |
|---|---|---|
| Internal function attribute names are TensorFlow implementation details. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The XLA compile constants map to `_XlaMustCompile`, `_XlaCompile`, and `_XlaScope`. | L63-L67 | `tensorflow/python/eager/polymorphic_function/attributes.py` |
| `polymorphic_function.py` uses `attributes_lib.XLA_COMPILE` when `_jit_compile` is set. | L637-L640 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

