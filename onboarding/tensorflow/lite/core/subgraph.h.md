# subgraph.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/subgraph.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `Subgraph` — the per-subgraph execution unit in TFLite. Each subgraph is a flat list of nodes (ops) and tensors with pre-planned memory allocation. `Subgraph::Invoke()` walks the execution plan, calling op `prepare`/`invoke` in order. Multi-subgraph models (e.g., with While/If control flow) have one `Subgraph` per branch.

The `Interpreter` owns a vector of `Subgraph` instances; the primary subgraph (index 0) is the main entry point.

## Code Commentary

### Logic

**`Subgraph` class:**
- **Execution plan** — a topological ordering of node indices. Built during `AllocateTensors()`.
- **`AllocateTensors()`** — memory-planning pass: computes tensor lifetimes, assigns offsets in a shared arena via `MemoryPlanner`.
- **`Invoke()`** — iterates over the execution plan, calling each op's `Prepare()` (on first run or reshape) then `Invoke()`.
- **`ResizeInputTensor(index, shape)`** — marks the subgraph as needing re-allocation.
- **Tensor management** — `tensors_` vector holds `TfLiteTensor` structs. Resource variables (`kTfLiteResource`) persist across `Invoke()` calls.

### Conventions

- **Static memory planning** — tensor buffers are pre-allocated in a shared arena. No dynamic allocation during inference.
- **Op interface via `TfLiteRegistration`** — each node holds a pointer to its registration struct with `init`, `free`, `prepare`, `invoke` function pointers.

### Invariants And Boundaries

- Subgraph wraps a sub-set of the model's ops for execution (e.g., main graph, each branch).
- Each Subgraph has its own allocator and execution plan.
- Subgraphs share tensor data through the parent Interpreter's tensor arena.

- **Fixed execution order** — the execution plan is computed once and reused unless reshaped.
- **Shared memory arena** — `SimpleMemoryArena` allocates offsets for all tensors, minimizing total memory footprint.
- **Delegates modify the execution plan** — applying a delegate replaces node(s) with a single delegate node.
- **Execution plan is static** — The plan is computed during AllocateTensors() and not updated. Adding ops or modifying control flow requires a new Interpreter.
- **Subgraph boundaries are opaque** — Which ops execute in which subgraph is determined by the converter. Edge tensors between subgraphs cost extra memory (not shared).
- **Cancellation in sub-execution** — Cancelling a subgraph Invoke() leaves the arena in an undefined state. Input/output tensors may be partially written.


### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation covering Subgraph internals was found.

| Finding | Citations | Source Path |
|---|---|---|
| No relevant documentation found | — | — |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Interpreter owns and orchestrates Subgraph instances | Interpreter class | [`tensorflow/lite/core/interpreter.h`](../tensorflow/lite/core/interpreter.h.md) |
| MemoryPlanner assigns tensor offsets in the shared arena | planner interface | `tensorflow/lite/memory_planner.h` |
| TfLiteRegistration defines the op interface (init, prepare, invoke) | registration struct | [`tensorflow/lite/core/c/common.h`](../tensorflow/lite/core/c/common.h.md) |
| GraphInfo provides node/edge connectivity for the execution plan | graph info | `tensorflow/lite/graph_info.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
