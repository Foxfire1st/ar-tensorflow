# direct_session.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/direct_session.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `DirectSession` (L46-L376) — the primary concrete implementation of the `Session` interface for single-process (local) execution. DirectSession orchestrates the entire execution pipeline: graph creation and placement, executor creation, memory management, threading, and Run() dispatch. It is the Session backend used by the Python API, the C API, and most single-machine TensorFlow usage. This is one of the most heavily exercised code paths in TensorFlow.

## Code Commentary

### Logic

**DirectSession (L46-L376):** Implements all Session pure-virtual methods:

- `Create(GraphDef)` (L65) — The heavy path. Constructs the graph from a GraphDef, runs the Placer to assign devices, partitions the graph into per-device subgraphs, creates per-partition Executors, and wires them through a Rendezvous.
- `Extend(GraphDef)` (L78) — Adds operations to an existing session. Lighter than Create but still runs placement and partition for the new subgraph.
- `Run(inputs, output_names, target_nodes, outputs)` (L95-L110) — Core execution dispatch. Creates an ExecutorBarrier for parallel device execution, calls RunAsync on each per-device Executor, waits for all to complete.
- `Run(RunOptions, inputs, output_names, target_nodes, outputs, RunMetadata)` (L120-L140) — Extended Run with per-step configuration: trace level, timeout, threadpool, inter-op/intra-op parallelism. Populates RunMetadata with partition graphs and step statistics.
- `PRunSetup / PRun` (L155-L180) — Partial run support. Creates a handle linking specific input/output names for incremental execution.
- `MakeCallable / RunCallable / ReleaseCallable` (L200-L250) — Subgraph callable API. Pre-creates executors and context to minimize per-invocation overhead.
- `Close()` (L260) — Shuts down all executors, releases device resources, clears ResourceMgrs.
- `ListDevices` (L275) — Delegates to DeviceMgr for the device list.
- `Finalize()` (L290) — Releases graph-level state after all callables are created. Reduces memory for inference-only sessions.

**Internal state:**
- **DeviceMgr** — Owns all devices (CPU, GPU, etc.). Created at session construction.
- **GraphExecutionState** — Manages mutable graph state across Create/Extend calls.
- **Per-device Executors** — One Executor per device partition.
- **RendezvousMgr** — Creates per-step IntraProcessRendezvous instances for cross-device tensor exchange.
- **FunctionLibraryRuntime** — Executes functions (tf.function, while_loop bodies, cond branches).
- **ThreadPool** — Inter-op thread pool for parallel RunAsync dispatch across devices.

### Conventions

- **One Session interface → one DirectSession instance** — DirectSession is the concrete backend; callers use it through the abstract Session API.
- **Lazy executor creation** — Executors are created during Create/Extend, not at construction.
- **CostModel optional** — DirectSession can optionally collect cost models for optimization; this adds measurable overhead to every Run().

### Invariants And Boundaries

- One DirectSession ≈ one Session in the Session API contract.
- Create() is idempotent — calling it twice on the same session returns an error.
- Run() dispatches to all per-device executors in parallel; the ExecutorBarrier synchronizes.
- Close() releases ALL resources; reusing a closed session is undefined behavior.
- Finalize() is irreversible — all graph state freed; only callables survive.- **Create() is expensive** — Graph construction, placement, and partitioning can take seconds for large graphs. Use MakeCallable/RunCallable for repeated execution.
- **Device partitioning is opaque** — The partitioner may split ops across devices in surprising ways. Use `log_device_placement=true` during debugging.
- **Thread pool contention** — Nesting session.Run() calls (e.g., inside a kernel) can deadlock if the outer run consumes all threads.
- **Run() after Finalize() fails** — Once graph state is finalized, only RunCallable works. This catches users by surprise.
- **PRun handle lifetime** — Partial run handles are invalidated by Close() and Extend(). Using a stale handle causes undefined behavior.
- **Close() while Run() in progress** — Not enforced at runtime. Results in use-after-free of executor state.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. DirectSession is an internal implementation detail not covered by public TF documentation.

## Repo-Internal References

DirectSession is the primary local Session implementation. It ties together placement, partitioning, executor creation, and execution dispatch.

| Finding | Citations | Source Path |
|---|---|---|
| Session is the abstract interface DirectSession implements | class Session | [`session.h`](../public/session.h.md) |
| Executor created per device partition; RunAsync dispatches | Executor | [`executor.h`](executor.h.md) |
| Placer assigns devices before partitioning | Placer | [`placer.h`](placer.h.md) |
| DeviceMgr owns all devices for this session | DeviceMgr | [`tensorflow/core/common_runtime/device_mgr.h`](device_mgr.h) |
| RendezvousMgr creates per-step Rendezvous instances | RendezvousMgr | [`tensorflow/core/common_runtime/rendezvous_mgr.h`](rendezvous_mgr.h) |
| FunctionLibraryRuntime for function execution | FunctionLibraryRuntime | [`function.h`](../framework/function.h.md) |
| GraphExecutionState for incremental graph building | GraphExecutionState | [`tensorflow/core/common_runtime/graph_execution_state.h`](graph_execution_state.h) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with full DirectSession lifecycle, internal state, Create/Run/Close paths, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
