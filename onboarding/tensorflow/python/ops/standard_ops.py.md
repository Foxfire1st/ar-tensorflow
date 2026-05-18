# standard_ops.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/ops/standard_ops.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

A bootstrap import hub that triggers gradient registration for all standard TensorFlow ops. It imports gradient modules (`array_grad`, `math_grad`, `nn_grad`, etc.) to ensure their `@RegisterGradient` decorators execute at import time, populating the global gradient registry.

This file is the reason `import tensorflow as tf` makes all standard ops differentiable — without it, gradient functions would not be registered and `GradientTape` / `tf.gradients` would fail for most ops.

## Code Commentary

### Logic

Pure import side-effect shim. The file imports gradient registration modules across all op categories:

- **Array** — `array_grad` (slice, concat, gather, etc.)
- **Math** — `math_grad` (add, mul, matmul, etc.)
- **NN** — `nn_grad` (conv2d, relu, softmax, etc.) and `cudnn_rnn_grad`
- **Data flow** — `data_flow_grad` (queue, stack, etc.)
- **Control flow** — `control_flow_grad` (cond, while, etc.)
- **State** — `state_grad` (assign, scatter, etc.)
- **Parsing** — `parsing_ops_grad`
- **IO** — `io_ops_grad`
- **Linalg** — `linalg_grad`
- **Manip** — `manip_grad` (reshape, tile, etc.)
- **Random** — `random_grad`
- **String** — `string_grad`
- **RPC** — `rpc_grad`
- **Collective** — `collective_grad`
- **Special math** — `special_math_grad`

Plus conditionally imports XLA, TensorRT, dataset, functional, lookup, tensor_array, map, and sdca gradient modules.

The `@RegisterGradient` decorators in each `*_grad.py` file register gradient functions in `ops._gradient_registry`. These are idempotent — importing twice has no effect.

### Conventions

- **Pure side-effect imports** — no functions or classes defined; only `import` statements.
- **Every op family has a corresponding `*_grad` module** — the naming convention is `<family>_grad.py` for `<family>_ops.py`.

### Invariants And Boundaries

- Re-export shim aggregating all public TF ops from gen_*_ops modules.
- No ops are defined here — only __all__ and import statements.

- This file must be imported before any gradient computation. `import tensorflow` triggers this transitively.
- The gradient registry is global and process-wide. Importing `standard_ops.py` populates it for all threads.
- Missing gradient modules cause import errors, not silent gradient failures.
- **All-or-nothing import** — rom tensorflow.python.ops import standard_ops imports every registered op's Python wrapper. This triggers eager context init and loads all gen_ modules. Startup impact: 200-500ms.
- **gen_* modules are auto-generated** — The underlying ops come from auto-generated code (gen_array_ops, gen_math_ops, etc.). API changes to these gen modules can cause standard_ops to fail on import with confusing ImportError.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Adding custom gradients via @tf.RegisterGradient | — | https://www.tensorflow.org/guide/create_op#gradients |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| ops.RegisterGradient decorator registers gradient functions in _gradient_registry | RegisterGradient class | [`tensorflow/python/framework/ops.py`](../tensorflow/python/framework/ops.py.md) |
| Individual gradient modules contain the actual gradient logic | *_grad.py files | `tensorflow/python/ops/array_grad.py`, `tensorflow/python/ops/math_grad.py`, etc. |
| GradientTape uses _gradient_registry to look up gradient functions | backprop.py | [`tensorflow/python/eager/backprop.py`](../tensorflow/python/eager/backprop.py.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
