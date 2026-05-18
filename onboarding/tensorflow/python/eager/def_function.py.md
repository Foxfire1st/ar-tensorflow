# def_function.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/eager/def_function.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

The public entry point for `tf.function` — the primary mechanism for converting Python functions into optimized TensorFlow graphs. This file is a thin re-export shim; the actual implementation lives in `tensorflow/python/eager/polymorphic_function/polymorphic_function.py`.

`tf.function` traces a Python function at call time, producing a `ConcreteFunction` with a traced computation graph. Subsequent calls with compatible argument types reuse the traced graph. The decorator supports JIT compilation via XLA (`jit_compile=True`), automatic control-flow conversion via AutoGraph, variable creation-on-first-call patterns, and input signature specification.

## Code Commentary

### Logic

This file re-exports:
- `Function` — the class representing a `tf.function`-decorated callable.
- `function` — the `tf.function` decorator/factory itself.
- `run_functions_eagerly()` / `functions_run_eagerly()` — global flag to disable tracing (for debugging).

**The real implementation** (`polymorphic_function.py`) centers on:
- `Function` class: wraps a Python callable. On first invocation, traces it via `FuncGraph`, producing a `ConcreteFunction`. Uses a `FunctionCache` to map input type signatures → traced graphs (polymorphic dispatch). Supports `input_signature` for explicit type contracts.
- `ConcreteFunction`: a traced graph + a callable. Can be serialized (SavedModel). Has `graph`, `inputs`, `outputs`, `structured_input_signature`, `structured_outputs`.
- **Tracing logic:** AutoGraph converts Python control flow (`if`, `while`, `for`) into `tf.cond` / `tf.while_loop`. Shape inference propagates through the traced graph. Variables created inside the function are "lifted" out (initialized once, captured on subsequent calls).
- **Retracing triggers:** New argument shapes or dtypes, new Python values for non-tensor arguments, changes in global state. Frequent retracing generates warnings.

### Conventions

- **`@tf.function` decorator** — the standard way to create graph functions: `@tf.function; def my_fn(x): return x + 1`
- **Polymorphic dispatch** — `tf.function` can handle multiple input shapes/dtypes by retracing. `get_concrete_function(x, y)` forces a specific trace.
- **AutoGraph integration** — the `autograph=True` flag (default) converts Python control flow to TF ops.

### Invariants And Boundaries

- Each unique input signature produces a distinct ConcreteFunction in the cache.
- Tracing happens once per signature; subsequent calls with same signature reuse the graph.
- Function tracing uses the active eager context — any device/execution mode changes are captured.
- Tracing side effects: Python control flow is evaluated during tracing, producing fixed graph structure.

- **Tensors are graph-mode inside `tf.function`** — `tf.Tensor.shape` may be partial; `__bool__` is disallowed (use AutoGraph or explicit `tf.cond`).
- **Side effects are traced once** — `print()`, `tf.print()`, and Python I/O inside `tf.function` execute only during tracing, not every call. Use `tf.print()` for runtime logging.
- **Variables persist across calls** — Variables created inside `tf.function` are initialized on first call and reused.
- **`input_signature` prevents retracing** — specifying `input_signature=[tf.TensorSpec(shape, dtype)]` locks the function to that signature.
- **`experimental_follow_type_hints=True`** — uses Python type annotations as input signatures.
- **Python semantics != graph semantics** — if x > 0: in a tf.function traces both branches; the condition is a tf.cond, not Python if. Side effects like print() run at trace time, not every call.
- **Signature mismatch is silent** — Calling a function with a new dtype/shape triggers retracing, which may silently use different ops. Use 	f.function(input_signature=...) to lock in exact types.
- **Retracing is expensive** — Each new signature triggers a full tracing + autograph + compilation cycle. Unpredictable inputs (varying batch sizes, different dtypes) cause excessive retracing.
- **Stateful ops inside function** — Variables modified inside tf.function must be tf.Variables, not Python lists/dicts. Python state is captured by value at trace time and never updated.
- **get_concrete_function() is for inspection** — Calling it with tensors (not TensorSpecs) traces immediately. Use tf.TensorSpec for cleanup-free signature definition.
- **Autograph limitations** — Not all Python constructs can be converted. Complicated loops, context managers, and generators fall back to running eagerly or crash.
- **XLA compilation at function level** — 	f.function(jit_compile=True) compiles with XLA. Shapes must be static; dynamic shapes cause retracing or failure.


### Todos

No known TODOs specific to this shim file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| tf.function guide: tracing, retracing, AutoGraph, signatures | — | https://www.tensorflow.org/guide/function |
| Concrete functions and SavedModel | — | https://www.tensorflow.org/guide/concrete_function |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Function and function decorator implementation | Function class, function() | `tensorflow/python/eager/polymorphic_function/polymorphic_function.py` |
| FuncGraph is the graph constructed during tracing | FuncGraph class | `tensorflow/python/framework/func_graph.py` |
| ConcreteFunction is the traced, callable result | ConcreteFunction | `tensorflow/python/eager/polymorphic_function/concrete_function.py` |
| AutoGraph converts Python control flow to tf.cond/tf.while_loop | autograph module | `tensorflow/python/autograph/` |
| FunctionCache maps input (type, value) → ConcreteFunction | function_cache.py | `tensorflow/core/function/polymorphism/function_cache.py` |
| TraceType system for polymorphic dispatch | trace_type.py | `tensorflow/core/function/trace_type/` |
| Context switching between eager and graph mode | context.py | [`tensorflow/python/eager/context.py`](../tensorflow/python/eager/context.py.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
