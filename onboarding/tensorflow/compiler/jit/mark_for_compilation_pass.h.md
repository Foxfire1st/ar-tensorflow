# mark_for_compilation_pass.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/jit/mark_for_compilation_pass.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines the `MarkForCompilationPass` — the Grappler optimization pass that identifies subgraphs (clusters) of ops suitable for XLA compilation. It annotates nodes with the `_XlaCluster` attribute, assigning the same cluster ID to nodes that should be compiled together. The `EncapsulateSubgraphsPass` later wraps each cluster into a TF function, which is then handed to `XlaCompiler` for HLO lowering.

This is the entry point for automatic JIT compilation (`TF_XLA_FLAGS=--tf_xla_auto_jit=1`). It determines what gets compiled vs what falls back to TF's native executor.

## Code Commentary

### Logic

**`MarkForCompilationPass`:** Implements `GraphOptimizationPass`.

- `Run(options)` — main entry point. Scans the graph, determines which ops are compilable (via `CompilabilityChecker` / allowlist), clusters them into connected components, and assigns cluster IDs via `_XlaCluster` attributes.
- `RunForTest(...)` — test-only variant with deterministic cluster names and optional deadness analysis.
- `GetAllowlistTable()` — returns the mapping of compilable op types to their priority/device constraints.
- **`kXlaClusterAttr`** — the constant `"_XlaCluster"` — attribute name set on each node to be compiled.

### Conventions

- **GraphOptimizationPass** — runs as part of Grappler's optimization pipeline during graph construction/session creation.
- **Cluster ID as attribute** — nodes with the same cluster ID value are compiled into one XLA computation.

### Invariants And Boundaries

- Runs as a Grappler optimization pass before device placement.
- Only marks nodes; does not compile. Actual compilation happens during execution.
- Cluster boundaries: nodes must form a connected subgraph of compatible ops.

- **Not all ops are compilable** — the allowlist gates which ops can be XLA-compiled. Ops without XLA lowering (e.g., string ops) are excluded.
- **Clusters must be connected** — only ops that form a connected subgraph can be clustered together.
- **Resource ops are handled specially** — variable reads/writes must be within the cluster or at cluster boundaries with explicit resource passing.
- **Marking is heuristic, not guaranteed** — Nodes that should be clustered may not be marked if the heuristics disagree (op not whitelisted, shape unknown, resource dependency). Use XLA_DEBUG_LOG to inspect.
- **Control flow breaks clustering** — tf.while_loop and tf.cond boundaries prevent clustering across them. Each loop body/cond branch is clustered independently.
- **Marking order sensitivity** — The mark-for-compilation pass is sensitive to node traversal order. Adding an unrelated op can change which nodes get clustered.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| XLA JIT compilation auto-clustering | — | https://www.tensorflow.org/xla#jit_compilation |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| CompilabilityCheckUtil determines whether individual ops can be compiled | util class | `tensorflow/compiler/jit/compilability_check_util.h` |
| EncapsulateSubgraphsPass wraps marked clusters into functions | encapsulate pass | `tensorflow/compiler/jit/encapsulate_subgraphs_pass.h` |
| XlaCompiler compiles the resulting function into HLO | XlaCompiler | [`tensorflow/compiler/tf2xla/xla_compiler.h`](../tensorflow/compiler/tf2xla/xla_compiler.h.md) |
| XlaOpRegistry provides the allowlist of ops with XLA lowering | registry | `tensorflow/compiler/tf2xla/xla_op_registry.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
