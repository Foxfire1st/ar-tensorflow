# Docs Evidence Pack — python

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/python/` |
| generated | 2026-05-17T17:00 |
| sourcesResolvedFrom | `system/sources.md` |
| status | partial |

## Scope

Find domain documentation covering the Python API: Tensor, Graph, Operation, EagerContext, tf.function, GradientTape, ResourceVariable, Keras Model training.

## Documentation Sources Checked

| Source Name | Source Type | Location | Result |
|---|---|---|---|
| TensorFlow API Docs | canonical online | https://www.tensorflow.org/api_docs | relevant (Python API reference for all symbols) |
| TensorFlow Guide | canonical online | https://www.tensorflow.org/guide | relevant (conceptual guides: eager execution, tf.function, autodiff, Keras) |
| Context7 MCP | API reference | context7-mcp skill | relevant (library lookups for Python symbols) |

## Confirmed Documentation Findings

| Finding | Evidence | Source Path | Applies To | Confidence |
|---|---|---|---|---|
| tf.Tensor is the core unit; has dtype, shape, device, numpy() | Python API docs | https://www.tensorflow.org/api_docs/python/tf/Tensor | `framework/tensor.py` | [HIGH] |
| tf.Graph contains tf.Operation nodes; graphs can be imported/exported as GraphDef | Python API docs | https://www.tensorflow.org/api_docs/python/tf/Graph | `framework/ops.py` | [HIGH] |
| tf.GradientTape records operations for automatic differentiation | TF Guide | https://www.tensorflow.org/guide/autodiff | `eager/backprop.py` | [HIGH] |
| tf.function compiles Python functions to TF graphs; tracing rules documented | TF Guide | https://www.tensorflow.org/guide/function | `eager/def_function.py` | [HIGH] |
| tf.Variable is a mutable tensor with read_value(), assign(), assign_add() | Python API docs | https://www.tensorflow.org/api_docs/python/tf/Variable | `ops/resource_variable_ops.py` | [HIGH] |
| Keras Model.fit() supports callbacks, validation, distributed training | TF Guide | https://www.tensorflow.org/guide/keras/training | `keras/engine/training.py` | [HIGH] |

## Documentation Constraints

- [MEDIUM] tf.function tracing behavior has many edge cases (Python side effects, control flow, shape polymorphism) — these are best documented in the source code and GitHub issues
- [MEDIUM] Keras training loop internals (steps_per_execution, distribution strategy) are complex and partially documented in guides; implementation details are code-level

## Terms And Definitions

| Term | Meaning | Evidence |
|---|---|---|
| Eager execution | Operations evaluate immediately; no graph is built | TF Guide: "Eager execution is an imperative programming environment" |
| Graph mode | Operations are added to a Graph; Session runs the graph | TF Guide |
| tf.function | Decorator that traces Python to a ConcreteFunction | TF Guide: function |
| GradientTape | Records forward-pass ops for backward-mode autodiff | TF Guide: autodiff |
| ResourceVariable | Variable backed by a ref-counted C++ resource handle | API docs |

## Files Affected

All 10 python/ onboarded files have corresponding Python API documentation.

## No Relevant Evidence Found

| Topic | Sources Checked | Notes |
|---|---|---|
| C++ pywrap bridge internals | TF API docs, TF Guide | Python→C++ bridge is an implementation detail; not in domain docs |
| Autograph conversion rules (Python→tf.while_loop etc.) | TF Guide | High-level overview exists; detailed conversion semantics are in source code |

## Open Questions

- None at [LOW] confidence; documentation is adequate for the Python API surface level.
