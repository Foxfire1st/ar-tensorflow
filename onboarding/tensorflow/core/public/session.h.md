# session.h

| Field | Value |
|---|---|
| repository | tensorflow |
| sourceFile | `tensorflow/core/public/session.h` |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | |
| lastVerifiedCommitDate | |

## Purpose

Defines the `Session` API — the primary user-facing C++ interface for executing TensorFlow computations. `Session` owns the lifecycle of a computation: creating graphs, extending them with additional operations, running with feeds and fetches, and closing resources. It is the bridge between graph construction (Python, C++, C APIs) and the execution engine (Executor, DirectSession).

The API is thread-safe: multiple threads may call `Run()` concurrently on the same Session. The primary implementation is `DirectSession` for single-machine execution; `MasterSession` handles distributed multi-machine execution.

## Code Commentary

### Session (L40-L65)

The abstract base class:
- **Create(GraphDef)** — initializes the session with a computation graph. Creates devices, compiles subgraphs, allocates resources.
- **Extend(GraphDef)** — adds new nodes to an existing session without recreating it. Only additive; existing nodes cannot be removed.
- **Run(feeds, fetches, targets, outputs)** — executes the graph:
  - `feeds`: map of output names→Tensors providing input values
  - `fetches`: list of tensor names to return
  - `targets`: list of node names to run to completion (no output needed)
  - `outputs`: returned tensors, one per fetch
- **Close()** — releases all session resources (devices, memory, variables).
- **PRun()** — partial run: begin execution, feed intermediates, fetch results incrementally.
- **ListDevices()** — returns available devices and their attributes.

### SessionOptions (L70+)

Configuration for session creation:
- **target** — gRPC target for distributed sessions (e.g., "grpc://worker:2222"). Empty for local.
- **config** — ConfigProto with device placement policy, graph options, GPU options, thread pool size.

### RunOptions / RunMetadata

- **RunOptions** — per-Run() configuration: trace level, timeout, output partition graphs.
- **RunMetadata** — per-Run() results: step statistics, partition graphs, cost estimates.

## Key Invariants

- Session is the owner of all graph state; closing the session frees everything.
- `Run()` is thread-safe for concurrent calls; each call is independent.
- `Extend()` is additive only — nodes cannot be removed once added.
- Feeds provide tensor data that replaces placeholder nodes in the graph.
- Fetches specify which tensor outputs to return; targets specify which ops must complete.

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| DirectSession is the primary local implementation | direct_session.h | `tensorflow/core/common_runtime/direct_session.h` |
| MasterSession for distributed execution | master_session.h | `tensorflow/core/distributed_runtime/master_session.h` |
| C API wraps Session: TF_NewSession, TF_SessionRun | c_api.h, c_api.cc | `tensorflow/c/c_api.h` |
| Python tf.Session wraps this via pywrap | session.py | `tensorflow/python/client/session.py` |
| ConfigProto defines session configuration | config.proto | `tensorflow/core/protobuf/config.proto` |

## Cross-Repo References

No relevant cross-repo evidence found.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TF Session and graph execution | — | https://www.tensorflow.org/api_docs/cc/class/tensorflow/Session |

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap
