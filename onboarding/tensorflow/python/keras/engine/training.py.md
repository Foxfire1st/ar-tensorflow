# training.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/keras/engine/training.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Defines `tf.keras.Model` — the high-level user-facing class that groups layers into a trainable/inferable model. `Model` inherits from `Layer` and provides the full training/inference lifecycle: `compile()` (configure optimizer, loss, metrics), `fit()` (train for N epochs), `evaluate()` (compute loss/metrics on test data), and `predict()` (run inference).

Supports two creation modes: the **Functional API** (connect `Input` → layer calls → `Model(inputs, outputs)`) and **subclassing** (override `__init__` and `call()`). Also integrates with `tf.distribute.Strategy` for multi-GPU/TPU training, callbacks, checkpointing, and SavedModel export.

## Code Commentary

### Logic

**`Model` class (L134-L2700+):** ~2700 lines. Key method groups:

- **Configuration:**
  - `compile(optimizer, loss, metrics, loss_weights, ...)` — configures the model for training. Stores optimizer, loss functions, and metrics. Supports `run_eagerly=True` for debugging.
  - `build(input_shape)` — creates layer weights based on input shape (lazy weight creation).

- **Training:**
  - `fit(x, y, epochs, batch_size, validation_data, callbacks, ...)` — the main training loop. Iterates over data, runs forward pass → loss computation → backward pass → optimizer step. Supports `tf.data.Dataset`, numpy arrays, and generator inputs. Reports metrics per batch/epoch via callbacks (e.g., `ModelCheckpoint`, `EarlyStopping`, `TensorBoard`).
  - `train_on_batch(x, y)` — single-batch training step. No callbacks.
  - `train_step(data)` — the per-batch logic (overridable for custom training loops via subclassing).

- **Evaluation:**
  - `evaluate(x, y, ...)` — computes loss and metrics on test data. No gradient computation.
  - `test_step(data)` — per-batch eval logic.
  - `predict(x, ...)` — inference only (no loss/metrics).

- **Serialization:**
  - `save(filepath)` — saves model weights (HDF5/TF format) or full model (SavedModel).
  - `load_weights(filepath)` — restores weights.
  - `get_config()` / `from_config()` — config dict serialization (Functional API models only).

- **Distribution:**
  - `_distribution_strategy` — when compiled under `tf.distribute.MirroredStrategy` (etc.), training is automatically replicated across devices.
  - `make_train_function()`, `make_test_function()`, `make_predict_function()` — create `tf.function`-wrapped step functions that distribute ops correctly.

- **Callbacks integration:** `fit()` invokes callbacks at `on_epoch_begin`, `on_batch_end`, `on_train_end`, etc. `History` callback collects loss/metrics per epoch.

### Conventions

- **`model.compile()` before `model.fit()`** — establishes optimizer and losses.
- **Functional API** — `inputs = Input(shape); x = Dense(64)(inputs); model = Model(inputs, x)` — preferred for standard architectures.
- **Subclassing** — override `call(self, inputs, training=False)` — for dynamic architectures.
- **`training` arg in `call()`** — controls dropout, batch-norm behavior. Set automatically during `fit()` vs `predict()`.

### Invariants And Boundaries

- compile() must precede fit()/evaluate()/predict() — raises an error otherwise.
- Loss and metrics are stateful; evaluate() resets metrics before each eval loop.
- fit() supports only training-aware layers; Dropout/BatchNormalization behave differently based on training argument.
- Weights are model-owned; model.trainable_weights is the optimizer's target.
- Distribution strategy must be set before model creation — with strategy.scope(): model = ... ensures correct device placement.

- **`compile()` must precede `fit()`/`evaluate()`/`predict()`** — raises an error otherwise.
- **Loss and metrics are stateful** — `model.evaluate()` resets metrics before the eval loop. `model.fit()` resets per epoch.
- **`fit()` supports only training-aware layers** — layers like `Dropout` and `BatchNormalization` behave differently based on the `training` argument.
- **Weights are model-owned** — `model.weights` collects all trainable + non-trainable weights. `model.trainable_weights` is the optimizer's target.
- **Distribution strategy must be set before model creation** — `with strategy.scope(): model = ...` ensures variables are created on the correct devices.

- **fit() converges slowly or not at all — check compile()** — Wrong loss function, optimizer, or metrics configuration in compile() causes silent training failure. No runtime error for mismatched loss/output shapes until the first batch.
- **steps_per_execution > 1 hides errors** — Multiple steps run in C++ before returning to Python. If step 2 fails, steps 3-N still partially execute. Callbacks fire only at Python boundaries.
- **validation_data vs validation_split** — validation_split uses the last fraction of training data. If data is not shuffled, validation set may be non-representative.
- **class_weight with custom training loops** — class_weight is only applied when using the built-in fit(). Custom train_step() must implement weighting manually.
- **evaluate() resets metrics before the loop** — Calling evaluate() twice on the same model gives identical results (metrics reset). To accumulate across multiple evaluate calls, use custom loops.
- **Callbacks fire between batches, not inside C++ loop** — If steps_per_execution=100, on_batch_end fires only every 100 batches. Profiling/timing callbacks become coarse.
- **Model.predict() returns numpy arrays** — Large datasets cause out-of-memory if predict() collects all outputs in RAM. Use predict(..., callbacks=[...]) or a tf.data pipeline with unbatching.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Keras Model guide: Functional API vs subclassing, compile, fit, evaluate | — | https://www.tensorflow.org/guide/keras/custom_layers_and_models |
| Training and evaluation with built-in methods | — | https://www.tensorflow.org/guide/keras/train_and_evaluate |
| Distributed training with Keras | — | https://www.tensorflow.org/guide/distributed_training |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Layer is the base class; Model inherits build(), call(), weights, losses from it | Layer class | `tensorflow/python/keras/engine/base_layer.py` |
| Optimizer classes (Adam, SGD, RMSprop) consumed by compile() | optimizer classes | `tensorflow/python/keras/optimizer_v2/` |
| Loss classes (BinaryCrossentropy, MSE, etc.) | loss classes | `tensorflow/python/keras/losses.py` |
| Metrics (Accuracy, AUC, etc.) | metric classes | `tensorflow/python/keras/metrics.py` |
| Callbacks (ModelCheckpoint, EarlyStopping, TensorBoard) | callback classes | `tensorflow/python/keras/callbacks.py` |
| tf.distribute.Strategy integration for multi-device training | distribution strategies | `tensorflow/python/distribute/` |
| SavedModel export via save() | SavedModel | `tensorflow/python/saved_model/` |
| tf.function wrapping of train/test/predict steps | def_function.py | [`tensorflow/python/eager/def_function.py`](../tensorflow/python/eager/def_function.py.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
