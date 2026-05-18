# xla_compiler.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/tf2xla/xla_compiler.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines `XlaCompiler` — the central orchestrator for compiling TensorFlow subgraphs into XLA (Accelerated Linear Algebra) computations. It performs symbolic execution of a self-contained TF subgraph (starting from known input shapes), converting TF ops into XLA HLO (High-Level Optimizer) IR operations. The resulting `XlaCompilationResult` can be further compiled by the XLA backend into executable code for CPU, GPU, or TPU.

This is the bridge between the TF graph world (GraphDef, OpKernel) and the XLA compiler world (HLO, `xla::XlaBuilder`, `xla::LocalClient`). Used by JIT (`XlaLaunch` / `XlaCompile` ops), AOT compilation, and the TF-to-HLO lowering pipeline.

## Code Commentary

### Logic

**`XlaCompiler` class:** Key structures and methods:

- **`Argument`** — describes one input to the compiled subgraph. Three kinds: `kConstant` (compile-time constant), `kParameter` (runtime tensor), `kResource` (variable/tensorarray/stack). Resources are packed (stack → (array, size) tuple; tensor array → backing array + gradients).
- **`CompileOptions`** — configuration for compilation: `generate_hlo_graph`, `shape_representation_fn`, `alias_passthrough_params`, `return_updated_values_for_all_resources`, etc.
- **`CompileSingleOp(graph, context, options)`** — compiles a single TF op (used by `XlaCompile` for explicit op-level compilation).
- **`CompileFunction(options, function, args)`** — compiles a TF function body into an XLA computation. Primary entry point for most JIT/AOT pathways.
- **`CompilationResult`** — output struct containing the `XlaComputation` proto (serialized HLO), input/output mappings, resource updates, and tensor layout information.
- **`XlaContext`** — per-compilation context stored in thread-local `OpKernelContext`. Tracks the `XlaBuilder`, current HLO state, and resource handles during symbolic execution.

The symbolic execution engine walks the TF subgraph's nodes, looks up each op's XLA lowering via `XlaOpRegistry`, and emits corresponding HLO instructions into the `XlaBuilder`.

### Conventions

- **Symbolic execution** — ops are not actually executed; instead, HLO instructions are emitted into the XLA computation builder.
- **Known shapes required** — all input shapes must be known at compile time. Dynamic shapes are handled via the `shape_representation_fn` reshape mechanism.
- **_Arg/_Retval nodes** — the TF subgraph boundary uses `_Arg` nodes for inputs and `_Retval` nodes for outputs, wrapping the subgraph as a callable function.

### Invariants And Boundaries

- CompileFunction() produces exactly one XlaComputation per function instance.
- XlaContext must be active for the entire duration of symbolic execution.
- Resource variables touched during compilation must be declared to XlaResource.
- The compilation device (XlaCompilationDevice) is not a real device — it exists only for tracing.

- **Static shapes only** — XLA computations require fully-known input shapes. A new XLA computation is generated for each new set of shapes.
- **Resource ordering** — Updated resource values (variables) are placed at the end of the computation output, after `_Retval` values.
- **Entry vs non-entry computations** — Shape representation functions only apply to entry computations and variable resource arguments.
- **Packed resources** — Resources are packed into tuples for passing as parameters and return values.
- **Static shape requirement** — XlaCompiler requires all tensor shapes to be fully known at compile time. Dynamic shapes cause compilation failure.
- **XlaContext is global-per-compilation** — Symbolic execution uses a global XlaContext. Nested compilations require careful context push/pop; missing this causes HLO corruption.
- **Resource variables need special handling** — Read/WriteVariable ops must go through XlaResource tracking. Forgetting to call XlaOpRegistry::RegisterCompilationDevice for resource ops means they silently fall back to TF execution.
- **Compilation is expensive** — Compiling a large subgraph can take seconds. Use DeviceCompiler's caching to avoid recompilation.
- **Deadness propagation** — Control flow dead tensors (is_dead=true) must propagate through XLA computations. Missing deadness handling produces wrong results for cond/while_loop gradients.
- **Cross-repo boundary** — XlaBuilder, XlaComputation, and LocalClient are defined in third_party/xla/. They follow XLA's own API stability guarantees, not TF's.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| XLA architecture overview | — | https://www.tensorflow.org/xla |
| XLA operation semantics | — | https://www.tensorflow.org/xla/operation_semantics |



## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XlaOpRegistry maps TF ops to XLA lowering functions | registry | `tensorflow/compiler/tf2xla/xla_op_registry.h` |
| XlaExpression tracks the current HLO representation during symbolic execution | expr class | `tensorflow/compiler/tf2xla/xla_expression.h` |
| XlaCompilationDevice is the JIT device used during symbolic execution | JIT device | `tensorflow/compiler/tf2xla/xla_compilation_device.h` |
| XlaBuilder is the XLA C++ API for building HLO computations | xla_builder.h | `third_party/xla/xla/hlo/builder/xla_builder.h` |
| XlaComputation is the serialized HLO proto result | proto | `third_party/xla/xla/hlo/builder/xla_computation.h` |
| DeviceCompiler uses XlaCompiler and caches results | DeviceCompiler | [`tensorflow/compiler/jit/device_compiler.h`](../tensorflow/compiler/jit/device_compiler.h.md) |

## Cross-Repo References

No meaningful cross-repo references found. XLA backends live in `third_party/xla/`, which is vendored within the TensorFlow repo, not a separate workspace repository.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
