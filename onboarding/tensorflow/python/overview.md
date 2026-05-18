# Python API Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/python/` |
| onboardingRoute | `tensorflow/python/overview.md` |
| parentOverview | [`overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## What This Area Is

The Python API layer is TensorFlow's primary user-facing surface. It exposes core computational abstractions — `Tensor`, `Variable`, `Graph`, `Operation`, `GradientTape` — alongside `tf.function` JIT compilation, the Keras high-level training API, distribution strategies, and data pipelines. In TF 2.x, **eager execution is the default**: operations execute immediately against C++ kernels, while `@tf.function` provides an opt-in pathway to graph mode for performance.

Key responsibilities: eager and graph-mode tensor operations, automatic differentiation via `GradientTape`, Python→graph tracing via Autograph and `tf.function`, Keras model training, distributed training orchestration, SavedModel serialization, and `tf.data` pipelines.

## What Belongs Here

| Path | Role |
|---|---|
| `framework/` | Core abstractions: Tensor, Graph, Operation, dtype, shape, device, op registration |
| `eager/` | Eager execution engine, GradientTape, tf.function tracing, context management |
| `ops/` | 500+ op implementations (math, array, NN, control flow, variables) |
| `keras/` | High-level API: Model, Layer, optimizers, losses, metrics, callbacks |
| `training/` | Legacy TF 1.x optimizers (kept for compatibility) |
| `autograph/` | Python→graph compiler: converts if/for/while to tf.cond/tf.while_loop |
| `distribute/` | Multi-GPU/TPU strategies: MirroredStrategy, TPUStrategy |
| `data/` | tf.data.Dataset pipeline (map, batch, prefetch, TFRecord) |
| `saved_model/` | SavedModel format I/O, signatures, asset management |
| `compiler/` | XLA/MLIR integration bridges |
| `client/` | Session API (TF 1.x, largely deprecated) |
| `types/` | Type annotations and TensorSpec ecosystem |
| `compat/` | v1/v2 compatibility gates |
| `trackable/` | Checkpoint tracking, variable discovery |

## What Does Not Belong Here

| What | Where |
|---|---|
| C++ runtime (Graph, Executor, kernels) | `tensorflow/core/` |
| XLA HLO IR and backends | `third_party/xla/xla/` |
| TFLite runtime | `tensorflow/lite/` |
| C API stable bindings | `tensorflow/c/` |
| DTensor mesh sharding | `tensorflow/dtensor/` |

## Structures Found Here

| Class | File | Purpose |
|---|---|---|
| Tensor | `framework/tensor.py` | Multi-dimensional array; eager or symbolic |
| Variable | `ops/resource_variable_ops.py` | Mutable state container (model weights) |
| Graph | `framework/ops.py` | DAG of Operations; computation IR for graph mode |
| Operation | `framework/ops.py` | Single node in a Graph; corresponds to one kernel call |
| GradientTape | `eager/backprop.py` | Automatic differentiation context; records ops for backprop |
| tf.function | `eager/def_function.py` | JIT decorator; traces Python→ConcreteFunction |
| ConcreteFunction | `eager/polymorphic_function/` | Traced graph + metadata; result of tf.function |
| Model | `keras/engine/training.py` | High-level learner: fit/predict/evaluate |
| Layer | `keras/engine/base_layer.py` | Composable stateful function; owns variables |
| DistributionStrategy | `distribute/distribute_lib.py` | Multi-device orchestrator |
| Dataset | `data/dataset_ops.py` | Lazy transformation pipeline |

## Operating Model

**Eager Mode (TF 2.x default):** Operations execute immediately, side effects visible, GradientTape records for autodiff, line-by-line debugging.

**Graph Mode (opt-in via @tf.function):** Python function traced to static graph, compiled and cached, XLA/Grappler optimizations applied, used for production performance.

## Main Flows

### Eager Execution
```
User: a = tf.constant([1,2,3])
  → convert_to_tensor() → Tensor.__new__()
  → execute_eager_op() [pywrap_tfe → C++]
  → kernel dispatched on device
  → EagerTensor returned immediately
```

### tf.function Tracing
```
@tf.function → first call traces
  → Autograph converts Python control flow
  → FuncGraph captures inputs, traces body
  → ConcreteFunction created with input signature
  → Compiled via Grappler/XLA
  → Subsequent calls reuse cached graph
```

### Automatic Differentiation
```
with GradientTape() as tape:
    y = model(x)
    loss = loss_fn(labels, y)
grads = tape.gradient(loss, vars)
  → walks backward through recorded ops
  → applies chain rule (reverse accumulation)
```

### Keras Training
```
model.compile(optimizer, loss, metrics)
model.fit(dataset, epochs=N)
  → per-batch: forward pass → loss → gradients → optimizer.apply
  → DistributionStrategy handles multi-device
```

## Load-Bearing Files

| File | Role | Onboarding |
|---|---|---|
| `framework/tensor.py` | Tensor class, numpy conversion | planned |
| `framework/ops.py` | Graph, Operation, name_scope | planned |
| `eager/context.py` | Eager context lifecycle | planned |
| `eager/def_function.py` | tf.function entry point | planned |
| `eager/backprop.py` | Gradient computation | planned |
| `ops/resource_variable_ops.py` | Variable class | planned |
| `keras/engine/training.py` | Model.fit() training loop | planned |
| `framework/framework_lib.py` | Public re-exports | planned |
| `ops/standard_ops.py` | Standard ops hub | planned |
| `keras/engine/base_layer.py` | Layer base class | planned |

## Local Invariants And Traps

- **Eager is default in TF 2.x** — graph mode is opt-in
- **Variables created inside tf.function** — only on first call; subsequent calls reuse
- **GradientTape only records differentiable ops** — non-differentiable ops are invisible
- **Python control flow in @tf.function** — auto-converted by Autograph; may have surprising behavior
- **tf.function retracing** — happens when input signature changes; can be expensive
- **DistributionStrategy scope** — variables created inside scope are replicated

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Python ops call C++ kernels via pywrap_tfe | execute_eager_op() | `eager/execute.py`, `tensorflow/c/` |
| tf.function produces graphs consumed by core Executor | ConcreteFunction → Graph | `eager/polymorphic_function/`, `core/common_runtime/` |
| Keras training uses GradientTape for autodiff | Model.fit() → tape.gradient() | `keras/engine/training.py`, `eager/backprop.py` |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TF Python API reference | — | https://www.tensorflow.org/api_docs/python |
| Eager execution guide | — | https://www.tensorflow.org/guide/eager |
| tf.function guide | — | https://www.tensorflow.org/guide/function |
| Keras guide | — | https://www.tensorflow.org/guide/keras |
| Distributed training guide | — | https://www.tensorflow.org/guide/distributed_training |

## File-Level Onboarding Map

(10 files — all planned per coverage plan wave 2)

## Child Overviews

Candidates: `keras/`, `distribute/`, `data/`

## How To Use This Area

1. Read `framework/framework_lib.py` — public API surface
2. Read `eager/context.py` — eager initialization
3. Read `framework/tensor.py` — Tensor semantics
4. Read `eager/def_function.py` + `eager/backprop.py` — tf.function + autodiff
5. Read `keras/engine/training.py` — high-level Model API

## Needs Verification

- Exact op count in ops/
- Keras standalone repo relationship

## Update History

- 2026-05-17: Initial route-local overview from full-bootstrap
