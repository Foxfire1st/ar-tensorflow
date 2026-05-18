# Compiler Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/compiler/` |
| onboardingRoute | `tensorflow/compiler/overview.md` |
| parentOverview | [`overview.md`](../../overview.md) |
| lastUpdated | 2026-05-18T11:19:29+02:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## What This Area Is

The XLA (Accelerated Linear Algebra) compiler stack optimizes TensorFlow graphs for hardware-accelerated execution. It provides **three compilation pathways**: (1) **JIT** — runtime clustering of compatible ops, encapsulation into functions, then XLA compilation; (2) **AOT** — offline compilation to standalone object files via `tfcompile`; (3) **MLIR** — modern pathway converting TF SavedModel to StableHLO for cross-framework portability.

This area is the TF-side bridge to XLA. The HLO IR and XLA backends (CPU/GPU/TPU code generation) live in `third_party/xla/xla/` (vendored). The compiler stack handles op clustering, graph legalization, control flow functionalization, and resource variable handling.

## What Belongs Here

| Path | Role |
|---|---|
| `jit/` | JIT integration: op clustering (MarkForCompilationPass), function encapsulation, device compilation |
| `tf2xla/` | Bridge: TF graph/function→HLO conversion, op legalization, resource handling |
| `aot/` | Ahead-of-time compilation: graph→object file+metadata |
| `mlir/` | MLIR pathway: TF dialect, StableHLO, quantization, MLIR bridge |
| `tf2tensorrt/` | Separate: TF→TensorRT conversion (not XLA) |
| `plugin/` | External plugin system (minimal) |
| `tests/` | Integration tests |

## What Does Not Belong Here

| What | Where |
|---|---|
| HLO IR definition | `third_party/xla/xla/hlo/ir/` |
| XLA backends (CPU/GPU/TPU codegen) | `third_party/xla/xla/service/` |
| XLA client (LocalClient) | `third_party/xla/xla/client/` |
| Core TF runtime | `tensorflow/core/` |
| Python eager execution | `tensorflow/python/eager/` |

## Structures Found Here

| Structure | Defining File | Purpose |
|---|---|---|
| XlaCompiler | `tf2xla/xla_compiler.h` | Central orchestrator: TF→HLO compilation via symbolic execution |
| DeviceCompiler | `jit/device_compiler.h` | Template wrapper around XlaCompiler; manages cache and persistence |
| MarkForCompilationPass | `jit/mark_for_compilation_pass.h` | Identifies compilable op clusters via allowlist+heuristics |
| EncapsulateSubgraphsPass | `jit/encapsulate_subgraphs_pass.h` | Wraps clusters into TF functions for compilation |
| XlaDevice | `jit/xla_device.h` | XLA execution device (CPU/GPU/TPU); manages tensor buffers |
| GraphCompiler | `tf2xla/graph_compiler.h` | Converts TF graphs to HLO |
| CompileGraph | `aot/compile.h` | AOT compilation entry point |
| TfToStablehlo | `mlir/tensorflow_to_stablehlo/` | TF SavedModel→StableHLO (modern pathway) |

## Operating Model

### JIT Compilation Pathway
```
TF Graph (ops marked @device=XLA_CPU/GPU/TPU)
  ↓ MarkForCompilationPass (identify + cluster)
TF Graph with _XlaCluster attributes
  ↓ EncapsulateSubgraphsPass (wrap clusters → functions)
TF Graph with _XlaCompiledKernel calls
  ↓ XlaLaunch op at runtime (first execution)
XlaCompiler::CompileFunction() (symbolic execution)
  ↓
HLO Module → XlaClient::Compile() → Native Executable
  ↓ XlaDevice::Compute()
Runtime execution
```

### AOT Compilation Pathway
```
TF Graph/SavedModel
  ↓ CompileGraph() [aot/compile.h]
  ↓ tf2xla bridge (graph→HLO)
  ↓ XlaClient::BuildAotCompilationResult()
Object File + Metadata → deployable library
```

### MLIR Pathway
```
TF SavedModel
  ↓ TfToStablehlo()
StableHLO Module (portable IR)
  ↓ MLIR transforms (optimize, quantize)
  ↓ Backend code generation
```

## Load-Bearing Files

| File | Role | Onboarding |
|---|---|---|
| `tf2xla/xla_compiler.h` | Central XLA compilation orchestrator | planned |
| `jit/device_compiler.h` | Device compilation wrapper | planned |
| `jit/mark_for_compilation_pass.h` | Op clustering pass | planned |
| `jit/xla_device.h` | XLA device abstraction | planned |
| `tf2xla/graph_compiler.h` | Graph→HLO lowering | planned |
| `aot/compile.h` | AOT compilation entry | planned |

## Local Invariants And Traps

- **XLA requires static shapes** at compile time; dynamic shapes cause recompilation
- **Stateful ops** (variables, resources) require special handling in tf2xla
- **Clustering is conservative** — only ops on the allowlist can be compiled
- **JIT compilation on first execution** — subsequent calls use cached executable
- **Control flow must be functionalized** — imperative if/while converted to functional form
- **Three separate pathways** — JIT, AOT, MLIR — share tf2xla bridge but have different entry points

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| JIT clustering triggered from Python @tf.function(jit_compile=True) | def_function.py | `tensorflow/python/eager/def_function.py` |
| Core Executor calls XlaLaunch op which triggers DeviceCompiler | executor.cc | `tensorflow/core/common_runtime/executor.cc` |

## Cross-Repo References

| Finding | Citations | Source Path |
|---|---|---|
| HLO IR and XLA backends in third_party/xla/xla/ (vendored) | — | `third_party/xla/xla/` |
| StableHLO is cross-framework standard | — | https://github.com/openxla/stablehlo |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| XLA architecture | — | https://openxla.org/xla |
| MLIR documentation | — | https://mlir.llvm.org/ |
| XLA JIT compilation in TF | — | https://www.tensorflow.org/xla |

## File-Level Onboarding Map

(6 files — all planned per coverage plan wave 3)

## Child Overviews

Candidates: `jit/`, `mlir/`, `tf2xla/`

## How To Use This Area

1. Read `tf2xla/xla_compiler.h` — the central orchestrator
2. Read `jit/mark_for_compilation_pass.h` — understand clustering
3. Read `jit/device_compiler.h` — compilation pipeline
4. Read `aot/compile.h` — offline workflow
5. Read `mlir/tensorflow_to_stablehlo/` — modern pathway

## Needs Verification

- Exhaustive JIT allowlist of compilable ops
- MLIR bridge vs graph-based path interaction details

## Update History

- 2026-05-18: Corrected parent overview link after reference-health check; source verification unchanged
- 2026-05-17: Initial route-local overview from full-bootstrap
