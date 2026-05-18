# ops.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/framework/ops.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

The central module for graph construction in TensorFlow. Defines `tf.Graph` (the computation DAG container), `tf.Operation` (a node in the graph), `SymbolicTensor` / `_EagerTensorBase` (graph-mode and eager-mode tensor variants), gradient registration infrastructure, name scopes, device scopes, control dependencies, and the default graph stack.

This is the bridge between the Python API and the C++ graph runtime: every op call (like `tf.matmul(a, b)`) goes through `Graph.create_op()` to add a node, and every tensor created is tracked by its owning `Graph`.

## Code Commentary

### Logic

**`Graph` (L1974-L4660):** Wraps the C++ graph via `pywrap_tf_session.PyGraph`. Key state:
- **Collections** (`_collections`) — named buckets for grouping related objects. Standard keys in `GraphKeys` include `GLOBAL_VARIABLES`, `TRAINABLE_VARIABLES`, `REGULARIZATION_LOSSES`, `UPDATE_OPS`, etc.
- **Device function stack** — `with tf.device("/GPU:0"):` pushes a device function onto `_graph_device_function_stack`. All ops created within the block use this function to compute their device placement.
- **Control dependencies** — `with tf.control_dependencies([a, b]):` ensures ops in the block execute after `a` and `b`.
- **Colocation stack** — `with tf.colocate_with(op):` forces ops onto the same device as `op`.
- **Name scopes** — `with tf.name_scope("layer"):` prefixes op names, producing hierarchical names like `"layer/MatMul"`.
- **Gradient override maps** — `_gradient_override_map` and `_gradient_function_map` let the graph customize gradient computation per op type.
- **Finalization** — `Graph.finalize()` makes the graph immutable. No new ops can be added.
- **Thread safety** — Not thread-safe for construction (must use single thread or external sync). Uses `_group_lock` to coordinate between `Session.run()` and graph mutation.

Key methods:
- `create_op(op_type, inputs, dtypes, ...)` — the central op creation path. Validates inputs, resolves device, adds to graph.
- `as_default()` — context manager making this graph the default.
- `get_operation_by_name(name)` — lookup by fully-qualified name.
- `get_tensor_by_name(name)` — tensor lookup (e.g., `"MyOp:0"`).
- `get_collection(name)` / `add_to_collection(name, value)` — collection access.
- `control_dependencies(control_inputs)` — context manager returning a scope with added control edges.
- `gradient_override_map(op_type_map)` — context manager temporarily overriding gradient functions.

**`Operation` (L1094-L1700):** Wraps `pywrap_tf_session.PyOperation`. Represents one op node:
- `inputs` — list of input `Tensor`s
- `outputs` — list of output `Tensor`s
- `type` — op type string (e.g., `"MatMul"`, `"Add"`)
- `device` — assigned device
- `control_inputs` — ops that must execute before this one
- `get_attr(name)` / `set_attr(name, value)` — attribute access
- `run(feed_dict, session)` — executes this op in a `Session`

**`SymbolicTensor` (L260):** Graph-mode tensor (produced by `Operation.outputs[i]`). Cannot be evaluated directly — requires `Session.run()`.
**`_EagerTensorBase` (L302):** Eager-mode tensor. Backed by live memory. Supports `.numpy()`.

**`RegisterGradient` (L1700):** Decorator-based gradient registration:
```python
@tf.RegisterGradient("MyOp")
def _my_op_grad(op, grad):
    return grad * op.inputs[0]  # example
```
Stored in a global registry. `get_gradient_function(op)` looks up the gradient for a given op type.

**`name_scope` (L5660-L6105):** Two implementations — v1 (compat) and v2 (current default). Creates hierarchical naming for ops. v2 returns the scope name string; pushes it onto the graph's name stack so subsequent `create_op()` calls pick it up.

**`GraphKeys` (L5282):** Constants for standard collection names: `GLOBAL_VARIABLES`, `TRAINABLE_VARIABLES`, `REGULARIZATION_LOSSES`, `UPDATE_OPS`, `SUMMARIES`, `QUEUE_RUNNERS`, `LOCAL_VARIABLES`, etc.

### Conventions

- **Default graph pattern** — ops are added to the currently active `tf.Graph`, accessed via `get_default_graph()`. `with graph.as_default():` activates a specific graph.
- **`_as_graph_element()` protocol** — any object that can be converted to a `Tensor` or `Operation` implements this method. Used by the session API and feed/fetch resolution.
- **C++ backend for core classes** — `Graph`, `Operation`, and tensors wrap C++ objects via `pywrap_tf_session` / `pywrap_tfe`. The Python layer adds convenience methods, collections, and scope management.
- **v1 compat shims** — Many deprecated TF1 APIs (`get_default_session`, `reset_default_graph`, etc.) remain for backward compatibility but are discouraged in TF2.

### Invariants And Boundaries

- One default graph per thread — _DefaultGraphStack is thread-local.
- Finalized graphs are immutable — Graph.finalize() sets _finalized=True; any subsequent create_op() raises.
- Graph construction is not thread-safe — _group_lock serializes Session.run() vs graph mutation.
- Op names within a graph must be unique; auto-deduplication appends _N suffixes.
- Control dependencies are transitive — B→A and C→B means C indirectly depends on A.

- **One default graph per thread** — `_DefaultGraphStack` is thread-local. `tf.function` creates a new graph for tracing each time.
- **Finalized graphs are immutable** — `Graph.finalize()` sets `_finalized = True`. Any subsequent `create_op()` call raises.
- **Ops are not thread-safe** — Graph construction must happen from a single thread. `_group_lock` prevents `Session.run()` from racing with graph mutation.
- **Name uniqueness** — Op names within a graph must be unique. Auto-deduplication appends `_N` suffixes.
- **Control dependencies are transitive** — If op B depends on A, and op C depends on B, C indirectly depends on A.

- **Default graph is thread-local** — Code running in a thread has a different default graph than the main thread. `tf.function` tracing happens in a fresh graph. Ops created outside the intended graph silently go to the wrong place.
- **Name collisions at scale** — Auto-deduplication (`_N` suffixes) makes op names unpredictable. Code relying on exact op names (feeds/fetches, checkpoint paths) breaks when `_1`, `_2` appear unexpectedly.
- **_as_graph_element() is called implicitly** — Passing a numpy array or Python int to session.run() triggers conversion. If conversion fails, the error is opaque.
- **v1 compat layers mask v2 behavior** — `disable_eager_execution()`, `get_default_session()`, `reset_default_graph()` still exist and can accidentally put the runtime into v1 mode. Check for stale compat calls when debugging.
- **~6000 lines** — This file is massive because it bundles Graph, Operation, device scopes, name scopes, container scopes, collections, graph keys, gradient registry, and control flow context into one module. Changes here affect every TF API that creates ops.


### Todos

- Remove `output_types` from `Operation.from_node_def()` (L1185, `FIXME(b/225400189)`).
- Move op name validation to C++ `Graph::AddNode` (L1178, `TODO(mdan)`).

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Graph and session concepts in TF guide | — | https://www.tensorflow.org/guide/intro_to_graphs |
| tf.function creates and traces Graphs automatically | — | https://www.tensorflow.org/guide/function |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Tensor class defined in tensor.py; Operation.outputs returns Tensor objects | Tensor class | [`tensorflow/python/framework/tensor.py`](../tensorflow/python/framework/tensor.py.md) |
| EagerTensor is the concrete eager implementation in C++ | pywrap_tfe.TFE_Py_Tensor | `tensorflow/python/eager/pywrap_tensor.cc` |
| GraphDef serialization: Graph.as_graph_def() → protobuf | GraphDef proto | `tensorflow/core/framework/graph.proto` |
| C++ Graph class (core/graph/graph.h) is the backend for ops.py's Graph | Graph, Node, Edge | [`tensorflow/core/graph/graph.h`](C:\ew\ar-coordination\memory-repos\ar-tensorflow\onboarding\tensorflow\core\graph\graph.h.md) |
| device.py provides merge_device for device function composition | MergeDevice, merge_device | `tensorflow/python/framework/device.py` |
| control_flow_ops.py uses Graph._control_flow_context for while/cond | CondContext, WhileContext | `tensorflow/python/ops/control_flow_ops.py` |
| tf.Variable registers itself in GraphKeys.GLOBAL_VARIABLES | Variable._variable_call | `tensorflow/python/ops/resource_variable_ops.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
