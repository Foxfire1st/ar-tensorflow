# Area Report — python

| Field | Value |
|---|---|
| repo | tensorflow |
| area | python |
| sourceRoute | `tensorflow/python/` |
| generated | 2026-05-17T17:00 |
| priority | CRITICAL |
| confidence | [HIGH] |

## Structure

```
framework/     → Tensor, Graph, Operation, dtype, device (core abstractions)
eager/         → Context, GradientTape, tf.function, polymorphic_function
ops/           → 500+ op implementations + resource_variable_ops
keras/         → Model, Layer, optimizers, callbacks, fit/predict/evaluate
training/      → V1 optimizers, training loops
distribute/    → Strategy (Mirrored, MultiWorker, TPU)
autograph/     → Python control flow → graph ops compiler
saved_model/   → Model serialization/deserialization
compiler/      → XLA/MLIR integration helpers
data/          → tf.data Dataset pipeline
```

## Interfaces

| Interface | Role | Key Files |
|---|---|---|
| Pywrap bridge | C++↔Python tensor/op marshalling | various `pywrap_*.cc` |
| EagerContext | Global eager execution singleton | `eager/context.py` |
| GradientTape | Records forward pass for backward AD | `eager/backprop.py` |
| tf.function | Python→graph tracing + autograph | `eager/def_function.py`, `polymorphic_function/` |
| Keras Model | fit/predict/evaluate high-level API | `keras/engine/training.py` |
| Variable | Mutable state via ResourceVariable | `ops/resource_variable_ops.py` |

## Patterns

- **Eager-first** — TF 2.x executes eagerly by default; graph mode is opt-in via tf.function
- **Tracing** — tf.function traces Python code once per input signature, builds a ConcreteFunction
- **Autograph** — converts Python control flow (if/while) to tf.cond/tf.while_loop
- **Resource handles** — Variables are ref-counted handles to C++ resources, not in-graph nodes
- **Keras as high-level** — virtually all high-level modeling goes through Keras

## Concerns

- **Tracing overhead** — first call to @tf.function is slow; retriggered by new shapes/dtypes
- **Eager/graph semantic gaps** — some Python constructs don't trace (e.g., side effects)
- **Memory** — eager tensors are not graph-optimized; memory grows with tensor count
- **Shape polymorphism** — retracing on new shapes can cause unexpected slowdowns
- **Keras→C++ round trips** — fit() calls into C++ training loops; debugging can be opaque

## Key Files

| File | Role | Onboarding |
|---|---|---|
| `framework/tensor.py` | Eager/graph Tensor type | done |
| `framework/ops.py` | Graph, Operation, gradient registry | done |
| `eager/context.py` | EagerContext singleton | done |
| `eager/def_function.py` | tf.function re-export | done |
| `eager/backprop.py` | GradientTape | done |
| `ops/resource_variable_ops.py` | ResourceVariable | done |
| `keras/engine/training.py` | Keras Model training API | done |
| `framework/framework_lib.py` | Re-export hub | done |
| `ops/standard_ops.py` | Gradient registration trigger | done |
| `__init__.py` | Package dunders | done |

## Confidence

[HIGH] — All 10 load-bearing files source-read and onboarded. Architecture verified from module structure.

## Update History

- 2026-05-17: Initial area report from full-bootstrap Phase 2
