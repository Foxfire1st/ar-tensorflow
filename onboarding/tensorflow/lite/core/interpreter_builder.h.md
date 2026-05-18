# interpreter_builder.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/interpreter_builder.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `InterpreterBuilder` — the factory that constructs an `Interpreter` from a `FlatBufferModel` and an `OpResolver`. It walks the FlatBuffer model, creates `Subgraph` instances, resolves op codes to `TfLiteRegistration` entries, builds the execution plan, and wires up tensor connections. The result is a fully-constructed but not-yet-allocated `Interpreter`.

## Code Commentary

### Logic

**`InterpreterBuilder` class:**
- Constructor takes `FlatBufferModel&` and `OpResolver&`. Optional `ErrorReporter`.
- `operator()(std::unique_ptr<Interpreter>*)` — builds and returns the interpreter.
- Steps during build:
  1. Parse subgraphs from the FlatBuffer model.
  2. Resolve each op code via `OpResolver::FindOp()`.
  3. Create `TfLiteNode` structs with resolved registration pointers.
  4. Setup tensor metadata (type, shape, quantization params).
  5. Build the execution plan (topological sort of nodes).

### Conventions

- **Builder pattern** — separate construction from execution. Call `AllocateTensors()` after building.
- **OpResolver abstraction** — the interpreter builder doesn't know about specific ops; it asks the resolver.

### Invariants And Boundaries

- Builder takes a FlatBufferModel (read-only) and produces an Interpreter.
- A single model buffer can build multiple Interpreter instances.
- Builder owns the delegate registry — delegates applied during Build().

- **Construction is pure setup** — no memory allocation for tensors during building.
- **Op resolution failure** — if an op is not found in the resolver, construction fails with an error.
- **Model buffer must outlive the interpreter** — The FlatBufferModel is referenced, not copied. Freeing the underlying buffer while interpreters still exist causes crashes on Invoke().
- **Build() is single-shot** — Calling Build() twice from the same builder is not supported. Create a new builder.
- **Error fallback** — If a custom op is not found, Build() may succeed (with placeholder ops) and fail at Invoke() time with cryptic errors.
- **NumThreads is a hint** — Setting num_threads=1 does not guarantee single-threaded execution if a delegate spawns its own threads.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite inference guide | — | https://www.tensorflow.org/lite/guide/inference |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Interpreter is the constructed output | Interpreter class | [`tensorflow/lite/core/interpreter.h`](../tensorflow/lite/core/interpreter.h.md) |
| OpResolver maps op codes to registrations | resolver | [`tensorflow/lite/core/api/op_resolver.h`](../tensorflow/lite/core/api/op_resolver.h.md) |
| FlatBufferModel provides the model schema | model | [`tensorflow/lite/core/model_builder.h`](../tensorflow/lite/core/model_builder.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
