# executor.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/executor.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `Executor` (L40-L117) — the graph execution engine that runs a resolved, placed Graph. An Executor takes a graph, a Rendezvous for cross-device tensor transport, and a runner for thread dispatch, then invokes each node's OpKernel when all its inputs are ready. Uses a pending-count scheduler: each node tracks how many inputs remain unproduced; when the count hits zero, the node is dispatched. Multiple threads can call `RunAsync()` concurrently on the same Executor.

Also defines `ExecutorBarrier` (L130-L190) for synchronizing parallel executor runs, and `NewLocalExecutor()` factory (L122).

## Code Commentary

### Logic

**Executor (L40-L117):** Abstract class with a single key entry point:

- `RunAsync(Args, DoneCallback)` (L100-L110) — Executes the entire graph. Calls `done` with OK or error when complete.
- `Run(Args)` (L113-L117) — Synchronous wrapper; blocks via `absl::Notification` until RunAsyncInternal completes.

**Executor::Args (L62-L95):** The context bundle for a single RunAsync() call:
- `step_id` (L63) — Process-wide unique identifier for this execution step.
- `rendezvous` (L65) — Rendezvous for cross-device tensor exchange; can be nullptr for single-device graphs.
- `stats_collector` (L66) — Collects per-op timing/memory traces for profiling.
- `call_frame` (L67) — Passes arguments and receives return values for function execution; nullptr for top-level graph runs.
- `cancellation_manager` (L68) — Allows aborting execution mid-flight via cancellation callbacks.
- `runner` (L82) — Function to dispatch closures (typically to a bounded threadpool). Controls intra-op parallelism.
- `run_all_kernels_inline` (L85) — If true, all kernels execute on the scheduling thread. For debugging and single-threaded environments.
- `sync_on_finish` (L75) — If true, calls `Device::Sync()` after all kernels complete, flushing GPU work.
- `deadline` (L79) — Per-step timeout for kernel completion. Empty means no deadline.
- `session_state / tensor_store / step_container / collective_executor` — Shared state for resource variables, tensor caches, and collective operations.

**Pending-Count Scheduling:** The executor maintains a pending-count per node — the number of unproduced inputs. When a node's output is produced, all consumers' pending-counts decrement. When a node's pending-count reaches zero, its kernel is dispatched via the runner.

**NewLocalExecutor (L122):** Factory: `NewLocalExecutor(params, graph, &executor)`. Creates an executor for single-machine execution.

**ExecutorBarrier (L130-L190):** Synchronizes N parallel executor runs. Provides `Get()` returning a `StatusCallback`; when all N callbacks fire, the barrier's completion callback is invoked with the aggregated status. Self-deleting after completion. If any executor fails, the Rendezvous is aborted.

### Conventions

- **Dataflow execution** — Graph edges ARE scheduling dependencies.
- **Runner-based dispatch** — Kernel invocation is delegated through the `runner` closure, allowing thread pool control.
- **Callback-driven completion** — RunAsync() never blocks; completion is signaled via the DoneCallback.

### Invariants And Boundaries

- Every node executes at most once per RunAsync() call.
- A node's kernel runs only after ALL input-producing nodes have completed (data dependency).
- RunAsync() can be called concurrently from multiple threads on the same Executor instance.
- `done` callback fires exactly once — either success or error.
- If cancellation_manager fires, all pending kernels receive cancellation callbacks; RunAsync completes with Cancelled error.- **DoneCallback on error still has side effects** — Even if `done(Error)` is called, some kernels may have produced outputs. Tensors in the Rendezvous may be partially populated. Cleanup must handle partial execution.
- **call_frame lifecycle** — If provided, call_frame must outlive the executor run. Function arguments are borrowed, not copied.
- **deadline enforcement is kernel-dependent** — The executor passes the deadline to kernels, but not all kernels check it. An op that never checks deadline never times out.
- **sync_on_finish blocks** — With sync_on_finish=true, the done callback is delayed until Device::Sync() completes (draining the entire GPU compute stream).
- **run_all_kernels_inline overrides parallelism** — All ops run on the scheduling thread, serializing execution. Only use for debugging.
- **ExecutorBarrier self-deletes** — After the completion callback fires, the barrier is deleted. Don't reference it after the callback.
- **Rendezvous must outlive RunAsync** — The executor doesn't own the Rendezvous; callers must ensure its lifetime.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. The Executor is an internal runtime component not covered by public TF documentation.

## Repo-Internal References

The Executor is the runtime engine beneath DirectSession. Every graph Run() dispatches through executor instances.

| Finding | Citations | Source Path |
|---|---|---|
| LocalExecutor is the single-machine implementation | LocalExecutor | [`tensorflow/core/common_runtime/local_executor.cc`](local_executor.cc) |
| Graph provides the node/edge structure to execute | Graph | [`graph.h`](../graph/graph.h.md) |
| OpKernel::Compute() called for each node | OpKernel | [`op_kernel.h`](../framework/op_kernel.h.md) |
| Rendezvous transports tensors between devices during execution | Rendezvous | [`rendezvous.h`](../framework/rendezvous.h.md) |
| Device::Compute dispatches kernels with device-specific hooks | Device | [`device.h`](../framework/device.h.md) |
| CancellationManager for aborting in-flight execution | CancellationManager | [`tensorflow/core/framework/cancellation.h`](../framework/cancellation.h) |
| DirectSession creates and runs local executors | DirectSession | [`direct_session.h`](direct_session.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with Executor::Args, pending-count scheduling, ExecutorBarrier, NewLocalExecutor, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
