# session.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/public/session.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines the abstract `Session` interface (L50-L155) — the primary public API for running TensorFlow graphs. `Session` creates graph execution environments, runs graphs with `Run()`, extends graphs with `Extend()`, and manages resources with `Close()`. It is the interface implemented by `DirectSession` (local) and `GrpcSession` (distributed), and is the entry point consumed by the Python `tf.Session` API and the C API.

## Code Commentary

### Logic

**`Session::Options` (L50-L60):** Configuration struct carrying `target` (gRPC server address), `config` (ConfigProto), and `env` (tsl::Env for filesystem access).

**Factory functions (L135-L155):**
- `NewSession(Options)` (L135) — Creates a new local `DirectSession`. Returns `Status` + `Session*` out-param.
- `NewSession(options, &session)` (L145) — Variant with full `SessionOptions` proto.

**Core API:**
- `Create(GraphDef)` (L75) — Constructs a graph in the session. Heavy — runs placement, partitioning, and executor creation.
- `Extend(GraphDef)` (L80) — Adds operations to an existing session without recreating executors. Lighter than Create.
- `Run(inputs, output_names, target_ops, outputs)` (L95) — Executes the graph. `inputs` is a map of `<name, Tensor>`, `outputs` is the received tensors. Blocks until complete.
- `Close()` (L110) — Releases all graph state, device resources, and thread pools.
- `ListDevices(&devices)` (L115) — Returns all available compute devices.
- `PRunSetup / PRun` (L120-L130) — Partial run support for incremental execution with persistent state.

**Destructor:** Virtual (L60). Derived classes (`DirectSession`, `GrpcSession`) handle per-backend resource release.

### Conventions

- **Abstract interface pattern** — Session defines the contract; backends implement the mechanics. Callers never depend on concrete session types directly.
- **Status-return convention** — Every method returns `Status`; errors are checked immediately by callers. No exceptions.
- **Output-parameter convention** — `Session*` is always returned via pointer parameter, never by `return`.

### Invariants And Boundaries

- Session::Run() blocks until all graph nodes complete or error.
- Create() and Extend() are mutually ordered — Create must precede the first Extend; Extend must not follow Close.
- Close() is terminal — reusing a closed session is undefined behavior.
- The Session owns its graph state; destroying the Session destroys all executors.- **Run() args lifetime** — Named tensors provided to Run() are borrowed, not owned. They must outlive the Run() call.
- **Concurrent Run()** — Multiple threads can call Run() concurrently on the same session. Thread safety is documented but backend-dependent.
- **PRun handle lifetime** — Partial run handles are invalidated by Close() and Extend(). Stale handles cause use-after-free.
- **Close() while running** — Calling Close() while Run() is in progress is explicitly warned against but not enforced. Results in undefined behavior.
- **Factory lifetime** — NewSession() returns an owned Session* that must be deleted. Leaking the session never finalizes devices.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. Session is an internal C++ abstraction not covered by public TF documentation.

## Repo-Internal References

This interface is the C++ API abstraction consumed by graph construction and the public TF session API.

| Finding | Citations | Source Path |
|---|---|---|
| Graph definition loaded into Session via Create/Extend | GraphDef proto | [`tensorflow/core/framework/graph.proto`](../../../core/framework/graph.proto) |
| DirectSession is the local (single-process) implementation | DirectSession class | [`direct_session.h`](../direct_session.h.md) |
| SessionOptions configures device count, GPU options, etc. | SessionOptions proto | [`tensorflow/core/protobuf/config.proto`](../../../core/protobuf/config.proto) |
| Python tf.Session wraps this C++ interface | BaseSession, Session | `tensorflow/python/client/session.py` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template — merged Key Invariants and Traps into `### Invariants And Boundaries`, moved Docs References under `## Code Commentary`, added `### Conventions` and `### Todos`, added `doc_type` metadata.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with Session API surface, factory functions, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
