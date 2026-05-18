# backprop.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/eager/backprop.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Defines `tf.GradientTape` — TensorFlow's automatic differentiation API for eager (and `tf.function`) execution. GradientTape records operations as they execute, then computes gradients via reverse-mode automatic differentiation (`tape.gradient(target, sources)`).

Also provides legacy helpers: `gradients_function()` (returns a callable that computes gradients), `implicit_grad()`, and `implicit_val_and_grad()` — all built on top of the tape infrastructure.

## Code Commentary

### Logic

**`GradientTape` (L705-L1250):** Context manager for recording differentiable operations.

- **Recording:**
  - `__enter__()` / `__exit__()` — pushes/pops the tape onto a thread-local tape stack.
  - `watch(tensor)` — explicitly marks a tensor for gradient tracking. Trainable `tf.Variable`s are automatically watched unless `watch_accessed_variables=False`.
  - `watched_variables()` — returns the variables watched by this tape.
  - `stop_recording()` — context manager to temporarily pause recording (useful for inference-only ops during training).
- **Gradient computation:**
  - `gradient(target, sources, output_gradients, unconnected_gradients)` — computes `d(target)/d(sources)` for each source. Returns `None` for sources that don't affect the target.
  - Targets can be scalar or non-scalar. For non-scalar targets, the sum of `output_gradients` is used to weight the gradient.
  - `jacobian(target, sources, ...)` — computes the full Jacobian matrix (all partial derivatives). More memory-intensive than `gradient`.
  - `batch_jacobian(target, sources, ...)` — assumes first dimension is batch and computes per-example Jacobians efficiently.
- **Persistence:**
  - `persistent=True` — the tape persists after `gradient()`, allowing multiple gradient calls on the same recorded computation. Resources are released on garbage collection.
  - `persistent=False` (default) — tape is consumed after the first `gradient()` call.
- **Higher-order gradients:** Nested `GradientTape` contexts record gradients as differentiable ops, enabling second-order differentiation.

**`gradients_function(f, params)` (L338):** Takes a Python function `f` and returns a new function that computes `df/dparams`. Uses `GradientTape` internally. Unlike `GradientTape`, this API is purely functional — no context manager needed.

**`implicit_grad(f)` / `implicit_val_and_grad(f)`:** Decorators that wrap a function so it returns gradients alongside its normal output.

### Conventions

- **`with tf.GradientTape() as tape:`** is the standard usage pattern.
- **`tape.gradient(loss, model.trainable_variables)`** in training loops.
- **Non-scalar targets** are summed element-wise before backprop; use `output_gradients` for weighted gradients.

### Invariants And Boundaries

- GradientTape::gradient() can be called at most once per tape instance.
- Watched tensors must be EagerTensors; constant tensors (tf.constant) are not watched by default.
- Persistent tapes allow multiple gradient() calls; non-persistent tapes auto-delete after first call.
- Recording stops when tape context exits unless watch_accessed_variables=True.

- **Tape records ops, not values** — the tape records the computation graph of operations. It does not store tensor values (memory isn't consumed per-element).
- **Tensors must be watched before ops are executed** — `tape.watch(x)` must happen before `y = f(x)`.
- **Only real/complex dtypes are differentiable** — integer, bool, and string tensors cannot carry gradients.
- **Persistent tapes hold memory** — recording many ops in a persistent tape can exhaust memory. Call `del tape` to release.
- **Tapes are thread-local** — each thread has its own tape stack. Gradients cannot cross thread boundaries.
- **`tf.function` gradients** — inside `tf.function`, `GradientTape` works via the `_gradient_function` dispatch path, which wraps graph ops as `_MockOp`s and looks up registered gradient functions from `ops._gradient_registry`.
- **Tape is single-use by default** — Calling gradient() twice on a non-persistent tape raises RuntimeError. Use persistent=True for multiple gradients; remember to del tape after.
- **tf.constant is not watched** — Only tf.Variable and explicitly watched tensors are tracked. Computing gradients w.r.t. a constant silently returns None.
- **Stop recording side** — Within a 	ape.stop_recording() block, ops are not tracked. But if the block creates variables, subsequent gradient() misses the variable's contribution.
- **Nesting GradientTapes** — Two nested GradientTape() contexts watching the same variable compute second-order gradients via forward-over-back. Incorrect nesting order silently produces wrong results.
- **Unconnected gradients** — If the target is not a function of the source (no differentiable path), gradient returns None. This is not an error — it means "no contribution." Check for None in custom gradient code.
- **Memory: tape holds intermediate activations** — The tape keeps all intermediate tensors alive until gradient() is called. For large models, call gradient() within the tape context or use a non-persistent tape.
- **Eager vs graph mode** — GradientTape only works in eager mode. For tf.function, use tf.GradientTape inside the function.


### Todos

- Properly support `CompositeTensor` in all functions (L20, `TODO(b/159343581)`).

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Automatic differentiation with GradientTape | — | https://www.tensorflow.org/guide/autodiff |
| Advanced autodiff: Jacobians, Hessians, higher-order gradients | — | https://www.tensorflow.org/guide/advanced_autodiff |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| tape.py provides push_new_tape/pop_tape (the C++-backed tape stack) | C++ tape interface | `tensorflow/python/eager/tape.py` |
| imperative_grad.py provides the imperative gradient computation for eager mode | imperative grad | `tensorflow/python/eager/imperative_grad.py` |
| ops._gradient_registry maps op names to gradient functions (registered via RegisterGradient) | gradient registry | [`tensorflow/python/framework/ops.py`](../tensorflow/python/framework/ops.py.md) |
| tf.Variable is automatically watched; resource_variable_ops.py | Variable class | [`tensorflow/python/ops/resource_variable_ops.py`](../tensorflow/python/ops/resource_variable_ops.py.md) |
| UnconnectedGradients enum controls behavior when a source doesn't affect the target | enum definition | `tensorflow/python/ops/unconnected_gradients.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
