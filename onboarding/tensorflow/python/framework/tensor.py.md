# tensor.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/framework/tensor.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Defines `tf.Tensor` — the primary Python object representing a multidimensional array of elements in TensorFlow. Every computation in TF produces or consumes `Tensor` objects. Also defines `TensorSpec` (a serializable description of a Tensor's shape, dtype, and name — used for function tracing and SavedModel) and `BoundedTensorSpec` (adds numeric bounds for quantization/range analysis).

This is the most fundamental Python-side type in TF. It bridges the Python API (operator overloading, iteration, numpy interop) with the C++ runtime (via `_op` references to `tf.Operation`, graph-mode/eager-mode dispatch).

## Code Commentary

### Logic

**`Tensor` (L123-L850+):** The core class. Inherits from `internal.NativeObject` and `core_tf_types.Symbol`.

- **Construction** — Not directly instantiated by users. Created by `tf.Operation.run()` or op functions. Stores:
  - `_op` — the `tf.Operation` that produced this tensor
  - `_value_index` — which output of the op this is
  - `_dtype` — `tf.DType`
  - `_name` — auto-generated from op name + output index
  - `_shape_val` / `_shape` — cached `TensorShape`

- **Operator overloading (L210-L240):** Defines `OVERLOADABLE_OPERATORS` — the set of Python dunder methods that may be overridden on `Tensor` (`__add__`, `__mul__`, `__matmul__`, `__getitem__`, `__neg__`, etc.). This enables `a + b`, `a @ b`, `a[0]` style syntax. The `_override_operator()` mechanism dispatches these to `tf.Operation` graph nodes.

- **Graph vs eager dispatch (L270-L290):**
  - `__bool__` — raises `TypeError` in graph mode (can't use a symbolic tensor as a condition). AutoGraph converts `tf.constant(True)` patterns.
  - `__iter__` — raises `OperatorNotAllowedInGraphError` in graph mode. Works in eager mode by iterating over the first dimension.
  - `__len__` — always raises; use `.shape` instead.

- **Shape system (L248-L550):**
  - `shape` / `get_shape()` — returns `TensorShape`. In eager mode, always fully known. In `tf.function` tracing, may be partial (unknown dimensions = `None`).
  - `set_shape(shape)` — asserts/merges a shape. In eager mode, asserts compatibility. In graph mode, merges with existing partial shape. Prefer `tf.ensure_shape()` for compiler guarantees.
  - `ndim` — shortcut for `shape.rank`.

- **Numpy interop (L560-L600):**
  - `__array__` — raises `NotImplementedError` in graph mode (can't convert symbolic tensor to numpy).
  - `numpy()` — available on `EagerTensor` (the concrete eager subclass). Returns the tensor value as a `numpy.ndarray`.
  - `eval(session)` — graph-mode evaluation via a `tf.Session`. Calls `session.run()`.

- **Tensor identity (L158-L170):**
  - `name` — `"op_name:N"` format.
  - `device` — the device this tensor is placed on.
  - `graph` — the `tf.Graph` this tensor belongs to.
  - `op` — the `tf.Operation` that produced this tensor.

- **Tape recording (L330-L335):** `_record_tape()` — called during `tf.function` tracing or eager gradient tape tracking. Connects a captured graph tensor to its source for automatic differentiation.

- **Consumer tracking (L600+):** `consumers()` returns ops that use this tensor as input. `_as_node_def_input()` formats the tensor as a `"op_name:index"` string for protobuf serialization.

- **Hash/equality (L570-L585):**
  - `__hash__` — in eager mode with `_USE_EQUALITY` enabled (TF2 default), tensors are unhashable (use `tensor.ref()`). In graph mode, uses `id(self)`.
  - `__eq__` — operator overloading; dispatches to `gen_math_ops.equal`.

**`TensorSpec` (L900-L1200):** Serializable type specification:
- Stores `shape`, `dtype`, `name` — a blueprint for what a tensor "looks like."
- `is_compatible_with(tensor)` — checks if a concrete tensor matches this spec.
- `is_subtype_of(other_spec)` — subtype checking for polymorphic function dispatch.
- `from_spec()` / `BoundedTensorSpec.from_spec()` — factory constructors.
- Used by `tf.function` for input signature tracing and by SavedModel for serialization.

**`BoundedTensorSpec` (L1200-L1450):** Extends `TensorSpec` with `minimum`/`maximum` numpy arrays. Used for:
- Quantization range tracking
- Numerical range analysis for compiler optimizations
- `tf.range` and bounded ops that need to communicate value constraints

### Conventions

- **`_disallow*` pattern** — `_disallow(task)`, `_disallow_bool_casting()`, `_disallow_iteration()` throw descriptive errors suggesting `@tf.function` or eager mode.
- **EagerTensor subclass** — At runtime, eager tensors are actually `EagerTensor` instances (a C++-backed subclass). `Tensor` is the graph-mode base; `EagerTensor` adds `numpy()` and immediate evaluation.
- **`_USE_EQUALITY` flag** — TF2 defaults to `True` (tensors are unhashable, numpy equality semantics). TF1 compat mode uses `id()`-based hashing.

### Invariants And Boundaries

- Tensor operations never mutate; 	f.add(a, b) produces a new Tensor.
- Tensor::dtype is immutable for the lifetime of the tensor.
- Tensor::device is set at creation; cross-device tensors trigger implicit copies.
- EagerTensor (C++ pywrap_tfe) is the concrete backing for eager-mode tensors; Tensor wraps both.

- **Graph vs eager separation** — `Tensor.__bool__`, `__iter__`, `__len__` are disallowed in graph mode. This prevents silently incorrect behavior when a symbolic tensor is used as a Python control-flow condition.
- **Shape can be partial in `tf.function`** — `Tensor.shape` may contain `None` dimensions during tracing. Only at runtime are dimensions fully resolved.
- **Tensor is not numpy** — `__array__` raises in graph mode. Use `.numpy()` in eager mode or `session.run()` in graph mode.
- **Operator overloading creates graph nodes** — `a + b` on two `Tensor`s creates a new `tf.Operation` in the graph, not a computed value. This is how the computation graph is built implicitly.
- **Tensors are immutable** — once created, a Tensor's value cannot change (unlike `tf.Variable`).
- **Tensor equality is by identity, not value** — 	f.constant(1) == tf.constant(1) returns a Tensor of bools, not True. Use 	f.reduce_all(tf.equal(a, b)).numpy() for value equality.
- **NumPy tensor protocol** — Tensors are implicitly convertible to numpy arrays via __array__(). This triggers a host transfer if on GPU, blocking the stream until data arrives.
- **Shape propagation vs runtime** — 	ensor.shape may return partial shapes (None dimensions) if shapes are dynamic. 	ensor.shape.as_list() raises ValueError on None. Use 	f.shape(tensor) for runtime shapes.
- **Cached .numpy() is a copy** — Calling numpy() twice creates two separate copies. For large tensors, assign to a variable.
- **Name uniqueness** — Two tensors with the same name are different tensors if they come from different graph construction calls.


### Todos

- Convert `__array_priority__` to numpy's `__numpy_ufunc__` mechanism (L600, `TODO(mrry)`).

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Tensor guide: shape, dtype, eager vs graph mode | — | https://www.tensorflow.org/guide/tensor |
| Concrete function guide: TensorSpec usage in tf.function | — | https://www.tensorflow.org/guide/concrete_function |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| EagerTensor is the concrete runtime subclass of Tensor for eager execution; adds .numpy() | EagerTensor C++ class | `tensorflow/python/eager/pywrap_tensor.cc` |
| tf.Operation is the graph node that produces a Tensor | Operation class | `tensorflow/python/framework/ops.py` |
| TensorShape is the Python representation of shape | TensorShape class | `tensorflow/python/framework/tensor_shape.py` |
| dtypes module defines DType for Tensor.dtype | dtypes module | `tensorflow/python/framework/dtypes.py` |
| tensor_conversion_registry maps Python types to Tensor conversion functions | registry | `tensorflow/python/framework/tensor_conversion_registry.py` |
| record.record_operation is the gradient tape hook that _record_tape calls | gradient tape recording | `tensorflow/python/eager/record.py` |
| tf.Variable wraps a mutable Tensor | Variable class | `tensorflow/python/ops/resource_variable_ops.py` |
| autograph converts Python control flow using tf.Tensor bool casting patterns | AutoGraph transform | `tensorflow/python/autograph/` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
