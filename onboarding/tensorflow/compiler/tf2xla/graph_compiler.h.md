# graph_compiler.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/tf2xla/graph_compiler.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines `GraphCompiler` — a deterministic graph traversal compiler that lowers a TF graph to HLO by walking nodes in topological order in a single thread. Resolves nondeterminism by enforcing a total order on all inputs to each node. Ensures structurally equivalent TF graphs produce identical XLA computations.

Created to decouple XLA compilation from the TF Executor. Used by `XlaCompiler` during symbolic execution of function bodies.

## Code Commentary

### Logic

**`GraphCompiler`:** Key behavior:
- **Topological traversal** — visits nodes in topological order (dependencies before dependents).
- **Deterministic ordering** — enforces a total order on all inputs to each node, eliminating nondeterministic iteration over hash-set edges.
- **Function call handling** — when a function call is visited during traversal, it is compiled through `XlaContext` into a nested XLA computation, and a `Call` op is inserted.
- **`XlaCompilationDevice`** — uses this JIT device for op-to-HLO lowering during traversal.
- **Constructor** takes `XlaCompilationDevice*`, `Graph*`, `FunctionLibraryRuntime*`, and `ScopedStepContainer*`.

### Conventions

- **Single-threaded** — compiles in the calling thread; no thread pool.
- **Deterministic output** — critical for reproducibility and AOT compilation where bit-identical results are expected.

### Invariants And Boundaries

- GraphCompiler converts a TF Graph into one HLO module per function body.
- Symbolic execution: each TF op translates to one or more HLO instructions.
- Compilation is deterministic — same graph always produces the same HLO module.

- **Deterministic by design** — the total input ordering guarantee eliminates nondeterministic HLO generation.
- **Function nesting** — nested function calls are compiled inline, creating nested XLA computations.
- **Unsupported ops cause fallback** — If an op lacks an XLA lowering (no TF2XLA kernel registered for "MyOp"), GraphCompiler creates a call-out to TF execution. These call-outs are serialized and slow.
- **HLO module size** — Large graphs produce equally large HLO modules. XLA backend compilation of huge modules can OOM. Split large subgraphs into multiple compilation clusters.
- **Shape-dependent control flow** — tf.while_loop with dynamic trip count may compile to different HLO depending on the loop condition shape. This causes silent recompilation per shape.


### Todos

- Remove `XlaCompilationDevice` usage (L36, `TODO(yunxing)`).
- Remove the hack wrapping `XlaExpression` in a tensor (L38, `TODO(yunxing)`).
- Make `XlaOpKernel` not a subclass of `OpKernel` so it can handle `XlaExpression` directly (L40, `TODO(yunxing)`).

## Docs References

No relevant domain documentation found.

| Finding | Citations | Source Path |
|---|---|---|
| No relevant documentation found | — | — |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XlaCompiler uses GraphCompiler for deterministic HLO lowering | XlaCompiler::CompileFunction | [`tensorflow/compiler/tf2xla/xla_compiler.h`](../tensorflow/compiler/tf2xla/xla_compiler.h.md) |
| XlaCompilationDevice is the JIT device used during graph traversal | compilation device | `tensorflow/compiler/tf2xla/xla_compilation_device.h` |
| XlaContext tracks builder state during compilation | context class | `tensorflow/compiler/tf2xla/xla_context.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
