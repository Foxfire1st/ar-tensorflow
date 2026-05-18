# device_compiler.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/jit/device_compiler.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines `DeviceCompiler` — a templated class that wraps the XLA compilation pipeline with caching, persistence, and profiling. It takes a TF function/cluster, lowers it to HLO via `XlaCompiler`, compiles the HLO into an executable via the XLA backend, caches the result in a `DeviceCompilationCache`, and optionally persists/loads serialized executables to/from disk.

Acts as the caching layer between the JIT compilation triggers (`XlaLaunch` ops) and the actual `XlaCompiler` + XLA backend compilation.

## Code Commentary

### Logic

**`DeviceCompiler<ExecutableType, ClientType>`:** Template parameterized on the executable and client types (e.g., `xla::LocalExecutable`, `xla::LocalClient`).

- **`CompileScope`** — enum: `kOp` (single-op compilation), `kFunction` (function-level compilation).
- **`CompileMode`** — enum for cache-miss behavior: `kLazy` (skip if not profitable), `kStrict` (always compile), `kAsync` (compile in background, fall back to non-XLA in foreground).
- **`CompileIfNeeded(scope, function, args, compile_mode, ...)`** — primary entry point. Checks cache; on miss, invokes `XlaCompiler::CompileFunction()` then compiles the resulting HLO into an executable. Sets `*out_compilation_result` and `*out_executable`.
- **`DeviceCompilationCache`** — keyed by cluster signature (function name + argument shapes/dtypes). Stores `XlaCompilationResult` + `ExecutableType`.
- **`DeviceExecutablePersistor`** — serializes/deserializes compiled executables to/from persistent storage (disk). Enables cross-process and cross-run caching.
- **`DeviceCompilationProfiler`** — tracks compilation timing, cache hit rates, and profitability heuristics for the `kLazy` compile mode.

### Conventions

- **Cache-first** — compilation is avoided on cache hit. The cache key includes function name, argument types/shapes, and compile options.
- **Template pattern** — parameterized to support multiple XLA backends (local CPU/GPU/TPU).

### Invariants And Boundaries

- CompilationResult is keyed by cluster fingerprint (set of op types and connectivity).
- CompileAndRun() either returns the compiled executable or falls back to TF execution.
- DeviceCompiler is owned by the XlaDevice; one compiler per device instance.

- **Static shapes required** — a new cache entry is created for each unique set of input shapes. Dynamic shapes cause cache fragmentation.
- **`kLazy` mode heuristics** — small clusters may not be compiled on cold cache, falling back to TF's native executor.
- **Persistence is optional** — disk-based executable caching depends on `DeviceExecutablePersistor` configuration and disk space.
- **Compilation cache is per-device** — GPU 0 and GPU 1 have separate caches. Moving a model between GPUs invalidates all cached compilations.
- **Cluster isolation requirement** — Nodes marked for XLA compilation must form isolated subgraphs. A single TF op between two XLA clusters forces both to fall back to TF.
- **Compilation timeout** — Compilation is asynchronous; if it takes too long, execution proceeds without XLA. This produces correct but slow results with no error.
- **Resource variable ops break clustering** — ReadVariableOp/AssignVariableOp in a cluster cause it to be split, potentially removing the performance benefit.


### Todos

- De-templatize once Device API migration is complete (L65, `TODO(b/255826209)`).

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| XLA JIT compilation for TensorFlow | — | https://www.tensorflow.org/xla#jit_compilation |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XlaCompiler performs the TF→HLO lowering | XlaCompiler::CompileFunction | [`tensorflow/compiler/tf2xla/xla_compiler.h`](../tensorflow/compiler/tf2xla/xla_compiler.h.md) |
| DeviceCompilationCache stores compilation results keyed by cluster signature | cache class | `tensorflow/compiler/jit/device_compilation_cache.h` |
| DeviceExecutablePersistor handles disk-based executable serialization | persistor class | `tensorflow/compiler/jit/device_executable_persistor.h` |
| MarkForCompilationPass identifies which clusters to compile | clustering pass | [`tensorflow/compiler/jit/mark_for_compilation_pass.h`](../tensorflow/compiler/jit/mark_for_compilation_pass.h.md) |
| XLA flags control JIT behavior (e.g., tf_xla_auto_jit) | flags | `tensorflow/compiler/jit/flags.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
