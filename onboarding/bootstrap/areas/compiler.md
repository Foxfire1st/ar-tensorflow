# Area Report — compiler

| Field | Value |
|---|---|
| repo | tensorflow |
| area | compiler |
| sourceRoute | `tensorflow/compiler/` |
| generated | 2026-05-17T17:00 |
| priority | HIGH |
| confidence | [HIGH] |

## Structure

```
jit/        → JIT compilation: clustering, encapsulation, XlaDevice, DeviceCompiler
tf2xla/     → TF→HLO lowering: XlaCompiler, GraphCompiler, XlaOpRegistry
aot/        → Ahead-of-time: CompileGraph → object file
mlir/       → MLIR pathway: TF dialect → StableHLO → backend
tf2tensorrt/ → TensorRT integration
plugin/     → Plugin compiler support
```

## Interfaces

| Interface | Role | Key Files |
|---|---|---|
| XlaCompiler | Central orchestrator: TF subgraph → HLO compilation | `tf2xla/xla_compiler.h` |
| MarkForCompilationPass | Grappler pass: identifies compilable op clusters | `jit/mark_for_compilation_pass.h` |
| DeviceCompiler | Caching layer wrapping XlaCompiler | `jit/device_compiler.h` |
| XlaDevice | Device that routes compute through XLA | `jit/xla_device.h` |
| GraphCompiler | Deterministic TF graph → HLO traversal | `tf2xla/graph_compiler.h` |
| CompileGraph | AOT: TF GraphDef → object file + metadata | `aot/compile.h` |
| XlaOpRegistry | Allowlist of ops with XLA lowering | `tf2xla/xla_op_registry.h` |

## Patterns

- **Clustering** — ops are grouped into XLA clusters via `_XlaCluster` attribute, then wrapped into TF functions
- **Function-level compilation** — XLA compiles one function at a time (cluster = function)
- **Deterministic HLO** — GraphCompiler enforces total input ordering to produce identical HLO from equivalent graphs
- **Caching** — DeviceCompiler caches compiled executables by cluster fingerprint; avoids recompilation
- **Three pathways** — JIT (runtime), AOT (offline), MLIR (modern/portable)

## Concerns

- **Allowlist gaps** — ops without XLA lowering fall back to TF executor, breaking cluster purity
- **Shape requirements** — XLA needs static shapes; dynamic shapes cause compilation failure
- **Cluster boundaries** — resource ops (variables) at cluster edges require special handling
- **XLA backend ownership** — HLO IR and backends in `third_party/xla/`, not compiler/ — version coupling risk
- **Determinism** — GraphCompiler's total ordering is correctness-critical for AOT reproducibility

## Key Files

| File | Role | Onboarding |
|---|---|---|
| `tf2xla/xla_compiler.h` | TF→HLO orchestrator | done |
| `jit/device_compiler.h` | Compilation cache | done |
| `jit/mark_for_compilation_pass.h` | Cluster identification | done |
| `jit/xla_device.h` | XLA execution device | done |
| `tf2xla/graph_compiler.h` | Deterministic graph→HLO traversal | done |
| `aot/compile.h` | AOT compilation entry point | done |

## Cross-Repo Boundary

XLA backend (HLO IR, service, CPU/GPU/TPU backends) lives in `third_party/xla/xla/`. The `compiler/` stack uses it through `xla::XlaBuilder`, `xla::XlaComputation`, `xla::Service`, but does not own these interfaces.

## Confidence

[HIGH] — All 6 load-bearing files source-read and onboarded. Architecture verified from headers. Boundary to `third_party/xla/` noted but not cross-repo-verified.

## Update History

- 2026-05-17: Initial area report from full-bootstrap Phase 2
