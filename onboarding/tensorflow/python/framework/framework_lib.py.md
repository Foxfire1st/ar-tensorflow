# framework_lib.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/framework/framework_lib.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

A public re-export hub for the `tensorflow.python.framework` package. Consolidates the most commonly used graph-building classes, functions, and utilities into a single import surface. This file exists so that other TF modules and external users can write concise imports like `from tensorflow.python.framework import framework_lib` instead of importing from multiple sub-modules.

## Code Commentary

### Logic

Pure re-export shim with no logic. Exports fall into categories:

- **Core types:** `Graph`, `Operation`, `Tensor`, `SparseTensor`, `DeviceSpec`, `IndexedSlices`
- **Graph utilities:** `device`, `name_scope`, `control_dependencies`, `colocate_with`, `get_default_graph`
- **Collections:** `GraphKeys`, `add_to_collection`, `get_collection`
- **Tensor utilities:** `convert_to_tensor`, `make_tensor_proto`, `make_ndarray`
- **Shape:** `Dimension`, `TensorShape`
- **Gradient registration:** `RegisterGradient`, `NotDifferentiable`, `NoGradient`
- **Wildcard exports:** All `dtypes` symbols, all `load_library` symbols
- **Randoms seeds:** `get_seed`, `set_random_seed`
- **Graph import:** `import_graph_def`

### Conventions

Follows conventions of the governing route-local overview.

### Invariants And Boundaries

- Re-export shim from tensorflow.python.framework.* — no new logic.
- Must include all public symbols from ops.py, tensor.py, dtypes.py, config.py, errors.py, random_seed.py, and device.py.

- This is a pure re-export module — no new logic or classes defined here.
- The `from dtypes import *` wildcard pulls in all `tf.dtypes` symbols. This is a notable namespace pollution vector.
- The `from load_library import *` wildcard exposes `load_op_library()` and related plugin loading functions.
- **Re-export list drifts** — When a new class is added to ops.py, framework_lib.py must be updated or the class is inaccessible via 	f.compat.v1. This is a common source of missing-symbol bugs.
- **Aliasing obscures origin** — rom tensorflow.python.framework.framework_lib import Tensor hides that the real Tensor lives in tensor.py. IDE "go to definition" may land in the shim.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| The individual re-exported modules contain the actual documentation | — | — |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| All exported symbols originate from tensorflow/python/framework/ sub-modules | ops, tensor, dtypes, etc. | `tensorflow/python/framework/ops.py`, `tensorflow/python/framework/tensor.py`, `tensorflow/python/framework/dtypes.py` |
| tf.load_op_library is re-exported via load_library wildcard import | load_library module | `tensorflow/python/framework/load_library.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
