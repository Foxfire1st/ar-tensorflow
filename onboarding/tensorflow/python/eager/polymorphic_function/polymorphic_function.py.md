# polymorphic_function.py

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/python/overview.md`](../../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../../overview.md)

## Purpose

Implementation body for `tf.function`. For XLA-facing questions, this is the
frontend file that stores the `jit_compile` option, converts it into graph
function attributes, and enters the optional XLA tracing context before the
function body is traced or called.

## Code Commentary

### Logic

- `Function.__init__` accepts `jit_compile` and stores it as `_jit_compile`.
- Tracing option generation copies current function attributes and adds
  `_XlaMustCompile` when `jit_compile` is not `None`; true values also add
  `NO_INLINE`.
- Variable-creation tracing and normal calls both wrap user code in
  `OptionalXlaContext`, so XLA intent is visible while the graph is built.
- The public `function()` decorator maps deprecated `experimental_compile`
  usage into `jit_compile` before constructing the `Function` wrapper.
- The docstring records the important user-facing contract: `jit_compile=True`
  requires the whole function to be compilable by XLA or TensorFlow raises
  `InvalidArgumentError`.

### Conventions

- Use `_jit_compile` for the Python-level option and `_XlaMustCompile` for the
  graph/function attribute consumed downstream.
- Treat this file as frontend intent and tracing behavior. Runtime kernel,
  compiler registry, and MLIR legalization behavior live in compiler/core routes.

### Invariants And Boundaries

- This file does not decide whether an individual op has a valid XLA lowering.
- `jit_compile=None` can be promoted by the `TF_FUNCTION_JIT_COMPILE_DEFAULT`
  environment behavior before decoration.
- Nested functions and current graph XLA context affect metrics and tracing, but
  not the existence of the `_XlaMustCompile` attribute when `jit_compile` is set.

### Todos

- TODO comments around XLA control-flow annotation and `experimental_compile`
  removal are local cleanup notes, not behavior guarantees.

## Docs References

TensorFlow's public docs describe `tf.function(jit_compile=True)` as the user
switch for XLA compilation and state that non-compilable functions can fail.

| Finding | Citations | Source Path |
|---|---|---|
| Public `tf.function` docs describe `jit_compile=True` as XLA compilation. | — | [TensorFlow tf.function API](https://www.tensorflow.org/api_docs/python/tf/function) |
| TensorFlow guide material also presents `tf.function(jit_compile=True)` as an XLA performance path. | — | [TensorFlow function guide](https://www.tensorflow.org/guide/function) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| `Function.__init__` accepts `jit_compile` and stores it as `_jit_compile`. | L467-L479; L518-L523 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| Tracing options add `_XlaMustCompile` and `NO_INLINE` based on `_jit_compile`. | L629-L640 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| Function tracing and calls enter `OptionalXlaContext(self._jit_compile)`. | L574-L600; L815-L833 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| The public decorator docstring records the XLA compilability and `InvalidArgumentError` contract. | L1588-L1601 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| The decorator constructs `Function(...)` after resolving `jit_compile` and deprecated `experimental_compile`. | L1647-L1675 | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| The `_XlaMustCompile` attribute name comes from the sibling attributes module. | L64 | `tensorflow/python/eager/polymorphic_function/attributes.py` |

## Cross-Repo References

No sibling memory repo is configured. XLA compiler behavior is in this TensorFlow
checkout under `tensorflow/compiler/` and vendored `third_party/xla/`.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

