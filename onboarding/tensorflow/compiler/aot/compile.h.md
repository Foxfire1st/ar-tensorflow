# compile.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/aot/compile.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines the AOT (Ahead-Of-Time) compilation entry point for TensorFlow. `CompileGraph()` takes a `GraphDef` and an XLA `Config`, compiles it into a native object file via XLA, and returns a `CompileResult` containing the object file data, program shape, and entry point name.

This is the `tfcompile` tool's underlying library — it produces a `.o` file and a header that can be linked into a C++ binary, enabling inference without the full TF runtime.

## Code Commentary

### Logic

**`CompileResult`** struct:
- `aot` — `unique_ptr<CpuAotCompilationResult>` containing the compiled object file and metadata.
- `program_shape` — `ProgramShapeProto` with the static shapes of all arguments and results.
- `entry_point` — the generated function name (e.g., `"entry"`).
- `pointer_size` — architecture pointer size in bytes.

**`CompileGraph(graph_def, config, flags, *compile_result)`** — main entry point:
1. Takes a `GraphDef` (serialized TF graph).
2. Configures XLA compilation via `tf2xla::Config` (feed/fetch specs).
3. Compiles through XLA's `CpuAotCompilationResult` pipeline.
4. Returns the compiled object file and shape metadata.
5. Output is architecture-specific (x86-64, ARM, etc.) and does not require the TF runtime.

**`Main(flags)`** — full compilation method for reuse in library settings.

### Conventions

- **Feed/Fetch via config** — the `Config` proto specifies which graph nodes are feeds (inputs) and fetches (outputs), analogous to `Session::Run()`.
- **CPU-only** — AOT compilation currently only supports CPU (no GPU/TPU AOT).

### Invariants And Boundaries

- AOT compilation produces a standalone binary; no TF runtime needed at inference.
- Input/output tensor shapes must be fully known at compile time.
- The compiled model is an .o file linked into the user application; it uses XLA runtime.

- **Static shapes required** — all input shapes must be fully known at compile time. Dynamic shapes are not supported in AOT mode.
- **Single entry point** — produces one function. Multi-signature models are not supported.
- **Architecture-locked** — the output object file is for a specific CPU architecture. Cross-compiling requires matching XLA backend configuration.
- **Output artifact is a .o file, not a library** — The compiled output is a single object file with (name, shape) bound at compile time. Deploying to a different platform requires recompilation.
- **Resource variables not supported** — AOT mode does not support models with tf.Variable. Must freeze the graph first.
- **Static shapes only** — Unlike JIT which can handle some dynamic shapes via retracing, AOT requires every dimension to be concrete at tfcompile invocation.
- **Limited op coverage** — Not all TF ops are supported in AOT mode. Unsupported ops fail at compile time with opaque XLA errors.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| tfcompile: AOT compilation for TensorFlow graphs | — | https://www.tensorflow.org/xla/tfcompile |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XlaCompiler is used internally by CompileGraph for TF→HLO lowering | XlaCompiler class | [`tensorflow/compiler/tf2xla/xla_compiler.h`](../tensorflow/compiler/tf2xla/xla_compiler.h.md) |
| tf2xla::Config proto specifies feed/fetch nodes | proto definition | `tensorflow/compiler/tf2xla/tf2xla.proto` |
| xla::cpu::CpuAotCompilationResult produces the object file | AOT result | `third_party/xla/xla/service/cpu/cpu_aot_compilation_result.h` |
| MainFlags control compilation options (target triple, output paths) | flags | `tensorflow/compiler/aot/flags.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
