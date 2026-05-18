# resource_variable_ops.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/ops/resource_variable_ops.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Defines `tf.Variable` — the mutable state container in TensorFlow. `ResourceVariable` (the default `tf.Variable` implementation in TF2) wraps a resource handle to mutable tensor storage. Variables persist across `Session.run()` calls in graph mode, and across function calls in eager/`tf.function` mode.

Variables are the backbone of model training: they hold trainable weights, are automatically watched by `GradientTape`, and are updated by optimizers. They also integrate with checkpointing (`tf.train.Checkpoint`) and SavedModel serialization.

## Code Commentary

### Logic

**`BaseResourceVariable` (L377):** Abstract base inheriting from `variables.Variable` and `core.Tensor`. Provides:
- `read_value()` — returns a `Tensor` snapshot of the current value
- `assign(value)` / `assign_add(delta)` / `assign_sub(delta)` — mutation ops
- `scatter_*` methods — sparse updates
- `sparse_read(indices)` / `gather_nd(indices)` — partial reads
- `_lazy_read()` — optimized read for control-dependency tracking
- Operator overloading: `+`, `-`, `*`, `/`, `@`, etc. all dispatch to `read_value()` then the op.

**`ResourceVariable` (L1707):** The concrete implementation, also inherits `CompositeTensor`. Key constructor args:
- `initial_value` — tensor or callable. Sets shape and dtype.
- `trainable` — adds to `TRAINABLE_VARIABLES` collection (default `True`). Non-trainable variables are excluded from optimizer updates.
- `dtype` — type override.
- `constraint` — optional projection function applied after optimizer updates (e.g., non-negativity, unit norm).
- `synchronization` / `aggregation` — distributed training settings (`ON_READ`, `ON_WRITE`; `SUM`, `MEAN`, `ONLY_FIRST_REPLICA`).
- `shape` — explicit shape (when `validate_shape=False` allows unknown-shape initial values).
- `handle` — existing resource handle (for variable import from MetaGraph).

Key properties:
- `device` — the device the variable's resource lives on.
- `handle` — the resource handle tensor (`DT_RESOURCE`).
- `trainable` — whether this variable is trainable.
- `constraint` — the post-update projection function.
- `initial_value` / `initializer` — the initial value tensor or op.

**`ResourceVariableGradient` (L1669):** Gradient-aware wrapper used by `GradientTape.gradient()`. When a variable is watched, the tape records which ops read from it. The gradient wrapper tracks how the variable's value flows through the computation.

**`UninitializedVariable` (L2274):** A placeholder for variables that haven't been initialized yet. Used during `tf.function` tracing for deferred initialization patterns.

**`VariableSpec` (L2622):** Type spec for serializing/deserializing variables in SavedModel and `tf.function` input signatures.

### Conventions

- **`tf.Variable(initial_value, trainable=True)`** — standard constructor. Creates on the default device.
- **`v.assign(new_value)`** — returns an op that updates the variable. In eager mode, executes immediately.
- **Variables are tensors** — they can be used directly in ops (implicit `read_value()`).
- **ResourceVariable is the TF2 default** — the old ref-based `Variable` (`use_resource=False`) is deprecated.

### Invariants And Boundaries

- Variables outlive graph construction — created once, read/modified across many graph runs.
- assign() returns a Tensor representing the assignment op; it must be executed (eagerly or via Session.run) for the value to change.
- Variable shape is fixed at creation. reshape() creates a new variable (or raises error).
- ResourceVariable (TF2) replaces RefVariable (TF1). RefVariable semantics differ: RefVariables could be passed by reference through ops.

- **Shape and dtype are fixed at creation** — `assign()` must use a value with compatible shape and dtype.
- **Variable lives on a device** — the resource handle is placed on a specific device at creation time. Cannot be moved.
- **`validate_shape=True`** (default) — the shape must be fully known at creation time.
- **GradientTape auto-watches trainable variables** — no need to call `tape.watch(v)` for variables.
- **Variable is not a numpy array** — `.numpy()` reads the current value from device to host.
- **`assign_add`/`assign_sub` are atomic on a single device**, but not across devices without synchronization.
- **assign() vs assign_add()** — Both return an op that must be executed. In graph mode, forgetting to run the returned op means the variable never updates.
- **scatter_update on CPU vs GPU** — scatter_update is not deterministic on GPU for overlapping indices. Use scatter_nd_update for deterministic results.
- **named vs unnamed handle** — Variables with 
ame='foo' can be retrieved via get_variable('foo'); unnamed variables can't. Naming collisions cause auto-deduplication.
- **AggregationMethod for distributed training** — When using tf.distribute.Strategy, variable reads don't guarantee latest value. Use 	f.distribute.DistributedValues and aggregation.
- **Lazy initializer** — Variables with initial_value=None are initialized lazily on first use. Accessing before initialization crashes.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Variable guide: creation, assignment, sharing, saving | — | https://www.tensorflow.org/guide/variable |
| Distributed variable synchronization and aggregation | — | https://www.tensorflow.org/guide/distributed_training |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Base class variables.Variable defines the abstract Variable interface | VariableV1 class | `tensorflow/python/ops/variables.py` |
| GradientTape auto-watches trainable ResourceVariables | tape.py, backprop.py | [`tensorflow/python/eager/backprop.py`](../tensorflow/python/eager/backprop.py.md) |
| Optimizers (SGD, Adam, etc.) read trainable_variables and call assign* methods | optimizer_v2.py | `tensorflow/python/keras/optimizer_v2/optimizer_v2.py` |
| Checkpoint saves variable values via Trackable | trackable base | `tensorflow/python/trackable/base.py` |
| SavedModel serialization uses VariableSpec | nested_structure_coder | `tensorflow/python/saved_model/nested_structure_coder.py` |
| DistributedStrategy replicates variables across devices | MirroredVariable, SyncOnReadVariable | `tensorflow/python/distribute/values.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
