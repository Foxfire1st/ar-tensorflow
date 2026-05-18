# function.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/function.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines the function system (L50-L1103) — TensorFlow's mechanism for representing, registering, optimizing, and executing reusable subgraphs as first-class callable units. Functions power `tf.function`, while_loop/cond control flow, gradient computation, and SavedModel. The core types are `FunctionDef` (protobuf serialization), `FunctionDefLibrary` (named collection), `FunctionBody` (in-memory graph representation), `FunctionLibraryRuntime` (execution engine), and `FunctionLibraryDefinition` (lookup + versioning). This is the largest and most complex header in the core framework.

## Code Commentary

### Logic

**FunctionDef and Related Protos (L67-L120):**
- `FunctionDef` — Proto describing a function: signature (OpDef), body (list of NodeDefs), ret mapping, control flow metadata. Created via `FunctionDefHelper::Create()`.
- `FunctionDefLibrary` — Collection of FunctionDefs, typically packed into GraphDef's `library` field.
- `GradientDef` — Maps a function name to its gradient function name.
- `FunctionDefHelper::Create(name, inputs, outputs, node_defs, ret_map)` — Programmatic constructor for building functions in C++.

**FunctionBody (L125-L160):** In-memory representation of a function ready for execution:
- `graph` — Owned `Graph*` containing the function's nodes.
- `arg_nodes / ret_nodes` — The _Arg and _Retval placeholder nodes.
- `fdef` — The original serialized FunctionDef.

**FunctionLibraryDefinition (L170-L450):** The type registry for functions. Maintains a hierarchical namespace:
- `AddFunctionDef / AddGradientDef` — Register new functions and their gradients.
- `Find(name)` — Lookup by name. Returns nullptr if not found.
- `Contains(name)` — True if the function exists in this library or any parent.
- `ToProto()` — Serializes to FunctionDefLibrary for embedding in GraphDef/SavedModel.
- Composite functions (functions composed of other functions) are fully resolved through the definition hierarchy.

**FunctionLibraryRuntime (L470-L750):** The execution engine for functions:
- `Instantiate(name, attrs, &handle)` — Creates a concrete instantiation with specific attribute values. This is where attribute substitution and graph specialization happen.
- `Run(Options, handle, args, rets, done)` — Executes an instantiated function. Options carry step_id, rendezvous, runner, cancellation manager.
- `RunSync(Options, handle, args, rets)` — Synchronous wrapper around Run().
- `ReleaseHandle(handle)` — Releases an instantiated handle.
- `IsStateful(name)` — Returns true if the function has side effects.
- `GetFunctionBody(handle)` — Returns the FunctionBody for inspection.

**FunctionLibraryRuntime::Options (L500-L540):** Execution context: step_id, rendezvous, cancellation_manager, runner, stats_collector, collective_executor, step_container. Mirrors Executor::Args.

**Optimization Pipeline (L800-L950):**
- `OptimizeGraph(flib, &graph)` — Applies Grappler optimizations (inlining, constant folding, CSE, layout optimization).
- `InlinedFunctionBodyPlacer` — Assigns devices to inlined nodes when a function is inlined at its call site.
- `FunctionalizeControlFlow` — Converts native TF control flow (Switch/Merge/Enter/Exit) into functional while_loop/cond.

**Registration Macros (L1000-L1103):**
- `REGISTER_FUNCTION_GRADIENT(name, grad_fn)` — Registers gradient function for built-in or user-defined functions.
- `REGISTER_OP_GRADIENT(op_name, grad_fn)` — Associates a Python gradient function with a C++ op.
- Both use static initialization and deferred registration.

### Conventions

- **Hierarchical library** — FunctionLibraryDefinition supports parent-child chains; child libraries inherit from parents.
- **Instantiate-before-Run** — Functions are instantiated once, then Run() can be called many times.
- **Thread safety** — Run() is thread-safe; Instantiate() is not.

### Invariants And Boundaries

- FunctionDef names are unique within a FunctionDefLibrary.
- FunctionBody::graph is owned; callers must not delete it.
- FunctionLibraryDefinition is hierarchical — child libraries inherit from parents.
- Function instantiation is idempotent per (name, attrs) pair — same inputs produce the same Handle.
- Run() on a handle from a different FLR instance is undefined behavior.
- Functions can reference other functions — cyclic dependencies are detected at Instantiate time.- **1103-line header** — This file is a dependency magnet. Changes can trigger rebuilds of half the codebase.
- **Function vs Op distinction** — Functions have their own definition (FunctionDef, not OpDef) and their own instantiation path. Don't mix FunctionLibraryDefinition with OpRegistry lookups.
- **Handle lifetime** — Instantiate() returns a Handle that must be released via ReleaseHandle(). Leaking handles leaks the instantiated graph and its memory.
- **Thread safety** — Run() is thread-safe (multiple calls on same handle allowed). Instantiate() is not — must be called from a single thread or externally synchronized.
- **Inlining limitations** — Not all functions can be inlined. Functions using resources, control flow, or non-trivial attributes may fail inlining silently.
- **Gradient registration ordering** — REGISTER_FUNCTION_GRADIENT must happen after both the function and its gradient function are registered. Static init order is undefined; use deferred registration.
- **Cross-function references** — If function A calls function B, B must be in the FunctionLibraryDefinition (or a parent) when A is instantiated. Missing functions fail at instantiation time, not registration time.

### Todos

No known TODOs specific to this file.

## Docs References

TensorFlow's public documentation covers tf.function, gradient computation, and graph execution — all of which route through the function system defined here.

| Finding | Citations | Source Path |
|---|---|---|
| tf.function guide: how Python functions become TF graphs | — | [TF Guide: tf.function](https://www.tensorflow.org/guide/function) |
| Creating custom ops with gradients | — | [TF Guide: Create an op — Gradients](https://www.tensorflow.org/guide/create_op#gradients) |
| Intro to graphs and functions | — | [TF Guide: Graphs and Functions](https://www.tensorflow.org/guide/intro_to_graphs) |

## Repo-Internal References

The function system is the bridge between op registration, graph construction, execution, and gradients. Almost every subsystem depends on it.

| Finding | Citations | Source Path |
|---|---|---|
| OpDef used for function signatures (input/output types) | OpDef | [`tensorflow/core/framework/op_def.proto`](op_def.proto) |
| Graph construction for FunctionBody | Graph | [`graph.h`](../graph/graph.h.md) |
| OpKernel used by function call ops (CallOp, SymbolicGradientOp) | OpKernel | [`op_kernel.h`](op_kernel.h.md) |
| OpRegistry for op lookups during function instantiation | OpRegistry | [`op.h`](op.h.md) |
| Executor runs functions via FunctionLibraryRuntime | Executor | [`executor.h`](../common_runtime/executor.h.md) |
| DirectSession owns the top-level FunctionLibraryRuntime | DirectSession | [`direct_session.h`](../common_runtime/direct_session.h.md) |
| Placer assigns devices for inlined function bodies | Placer | [`placer.h`](../common_runtime/placer.h.md) |
| Gradient computation uses the function gradient registry | GradientTape | `tensorflow/python/eager/backprop.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with FunctionDef, FunctionBody, FunctionLibraryDefinition, FunctionLibraryRuntime, optimization pipeline, REGISTER_FUNCTION_GRADIENT, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
