# tensorflow — Onboarding Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `repo-overview` |
| sourceRoute | `tensorflow/` |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## What This Repo Is

TensorFlow is an end-to-end open-source machine learning platform. It provides a comprehensive ecosystem for building, training, and deploying ML models across servers, edge devices, and browsers. The repo is the canonical implementation — a C++ runtime core with Python, C, C++, Java, Go, and JavaScript bindings.

**Core responsibilities:**
- Graph-based computation engine with eager and graph execution modes
- Automatic differentiation for training
- XLA (Accelerated Linear Algebra) compiler for hardware-optimized execution
- TensorFlow Lite for mobile/embedded deployment
- Keras high-level API for model building
- Distributed training across multiple devices and machines

**Technologies:** C++, Python, Bazel (build), Protocol Buffers (serialization), FlatBuffers (Lite models), MLIR (compiler IR), gRPC (distributed communication)

## Architecture At A Glance

```text
┌─────────────────────────────────────────────────────────────────┐
│                     USER API SURFACE                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Python   │  │ C API    │  │ C++ API  │  │ Go/Java/JS    │  │
│  │ (Keras,  │  │ (c_api)  │  │ (Scope)  │  │ (bindings)    │  │
│  │  eager)  │  │          │  │          │  │               │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └───────┬───────┘  │
│       │             │             │                  │          │
│       ▼             ▼             ▼                  ▼          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              PYTHON API LAYER (tensorflow/python/)       │   │
│  │  framework/ │ eager/ (tf.function, GradientTape)         │   │
│  │  ops/ (500+) │ keras/ │ distribute/ │ autograph/         │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │ pywrap (C++ bridge)                  │
└───────────────────────────┼─────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                C++ RUNTIME CORE (tensorflow/core/)              │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌───────────┐  │
│  │ Graph    │  │ Executor │  │ DirectSession│  │ Kernels   │  │
│  │ (DAG)    │─▶│ (run)    │◀─│ (local)      │─▶│ (1000+)   │  │
│  └──────────┘  └──────────┘  └──────────────┘  └───────────┘  │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌───────────┐  │
│  │ Device   │  │ Placer   │  │ Rendezvous   │  │ Allocator │  │
│  │ (CPU/GPU)│  │ (assign) │  │ (sync)       │  │ (BFC)     │  │
│  └──────────┘  └──────────┘  └──────────────┘  └───────────┘  │
│                                                                 │
│  ┌──────────────────────┐  ┌────────────────────────────────┐  │
│  │ Distributed Runtime  │  │ Grappler (graph optimization)  │  │
│  │ (Master/Worker, gRPC)│  │ (CSE, const fold, layout)      │  │
│  └──────────────────────┘  └────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│   XLA COMPILER           │  │   TENSORFLOW LITE        │
│   (tensorflow/compiler/) │  │   (tensorflow/lite/)     │
│                          │  │                          │
│  JIT: cluster→compile    │  │  Interpreter → Subgraph  │
│  AOT: offline→object     │  │  OpResolver → Kernels    │
│  MLIR: TF→StableHLO      │  │  Delegates (GPU/NNAPI)   │
│                          │  │  FlatBuffer schema       │
│  → XLA backends (xla/)   │  │  Micro (embedded)        │
└──────────────────────────┘  └──────────────────────────┘
```

## Code Structure

| Area | Source Route | Tech | Purpose | Local Overview |
|---|---|---|---|---|
| Core Runtime | `tensorflow/core/` | C++ | Graph, execution, kernels, devices, distributed | planned |
| Python API | `tensorflow/python/` | Python | User-facing API, Keras, eager, tf.function | planned |
| Compiler | `tensorflow/compiler/` | C++/MLIR | XLA JIT/AOT/MLIR compilation pipelines | planned |
| Lite | `tensorflow/lite/` | C/C++ | Mobile/embedded runtime, delegates | planned |
| C API | `tensorflow/c/` | C | Stable language-agnostic bindings | planned |
| C++ API | `tensorflow/cc/` | C++ | Modern C++ graph DSL | deferred |
| DTensor | `tensorflow/dtensor/` | C++/Python/MLIR | Distributed tensor mesh sharding | deferred |
| JS | `tensorflow/js/` | JS/TS | TF.js codegen bridge | deferred |
| Go | `tensorflow/go/` | Go | Go bindings (unmaintained) | deferred |
| Java | `tensorflow/java/` | Java | Java bindings (deprecated) | skip |
| Security | `tensorflow/security/` | Docs/fuzz | Vulnerability advisories, OSS-Fuzz | reference |
| Tools | `tensorflow/tools/` | Python/Shell | Build, pip packaging, CI | deferred |
| Root config | `WORKSPACE`, `MODULE.bazel`, `configure.py` | Bazel/Python | Build system configuration | n/a |

## Functional Areas

### Core Runtime (`tensorflow/core/`)

The C++ foundation. Owns the computation Graph (DAG of Nodes), the Executor that runs it, the Session API that users interact with, the OpKernel framework for operation implementations, the Device abstraction for CPU/GPU/TPU placement, and the distributed runtime for multi-machine execution. The `framework/` subdirectory defines abstractions; `common_runtime/` implements execution; `kernels/` contains 1000+ concrete op implementations.

### Python API (`tensorflow/python/`)

The primary developer surface. `framework/` provides Tensor, Graph, Operation, dtype, and device abstractions. `eager/` implements eager execution, `GradientTape` for automatic differentiation, and `tf.function` for graph compilation. `ops/` contains 500+ operation implementations. `keras/` is the high-level model API (layers, optimizers, callbacks). `distribute/` handles multi-GPU/TPU strategies. `autograph/` compiles Python control flow to graph ops.

### Compiler (`tensorflow/compiler/`)

Three compilation pathways: (1) **JIT** — runtime clustering of compatible ops, encapsulation into functions, then XLA compilation via `XlaCompiler`; (2) **AOT** — offline compilation to standalone object files via `tfcompile`; (3) **MLIR** — modern pathway converting TF SavedModel to StableHLO for cross-framework portability. The HLO IR and XLA backends live externally in `third_party/xla/xla/`.

### Lite (`tensorflow/lite/`)

Mobile and embedded runtime. Loads `.tflite` FlatBuffer models via `FlatBufferModel`, resolves ops via `OpResolver`, schedules execution on `Subgraph` with `Interpreter`, manages memory via a `SimpleMemoryArena`. Hardware acceleration through pluggable `Delegate` interface (GPU, NNAPI, XNNPACK, CoreML). `micro/` is a separate minimal implementation for embedded systems.

## Cross-Repo References

No cross-repo dependencies configured in `settings.json`. The HLO IR and XLA backend live in `third_party/xla/xla/` (vendored, not cross-repo). Keras has its own repository for non-TF backends but TF's Keras is integrated here.

| Finding | Citations | Source Path |
|---|---|---|
| No relevant cross-repo evidence found. CrossRepo.allow is empty. | — | — |

## Build & Dev

- Configure build: `python configure.py`
- Build with Bazel: `bazel build //tensorflow/tools/pip_package:build_pip_package`
- Python package: `bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg`
- Test: `bazel test //tensorflow/...`

## Key Invariants

- Graph is a DAG; no cycles allowed in op dependencies
- Tensors are ref-counted; TensorBuffer owns the underlying memory
- OpKernel::Compute() is the canonical op execution contract
- The C API (`c/c_api.h`) is the stable ABI boundary; all bindings go through it
- Models for Lite are FlatBuffer-serialized; no Protobuf dependency at runtime
- XLA compilation requires static shapes at compile time
- Eager execution is the default in TF 2.x; graph mode is opt-in

## Glossary Terms

| Term | Meaning | Notes |
|---|---|---|
| Graph | DAG of Node objects representing a computation | Defined in `core/graph/graph.h` |
| Tensor | N-dimensional typed array with ref-counted buffer | `core/framework/tensor.h` |
| OpKernel | Base class for operation implementations | `Compute()` is the core contract |
| Session | User-facing API to create and run graphs | `core/public/session.h` |
| Rendezvous | Producer-consumer tensor passing mechanism | Local/remote implementations |
| HLO | High-Level Operations — XLA's intermediate representation | Lives in `third_party/xla/` |
| MLIR | Multi-Level Intermediate Representation | Compiler infrastructure |
| StableHLO | Standardized, versioned HLO subset for MLIR ecosystem | Portable IR |
| tf.function | Decorator that traces Python to a TF graph | `python/eager/def_function.py` |
| GradientTape | Automatic differentiation context in eager mode | `python/eager/tape.py` |
| Delegate | Hardware accelerator plugin for TFLite | GPU, NNAPI, XNNPACK, etc. |
| FlatBuffer | Serialization format for TFLite models | Zero-copy deserialization |

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TensorFlow API reference (Python, C++, Java) | — | https://www.tensorflow.org/api_docs |
| TensorFlow guide (architecture, concepts) | — | https://www.tensorflow.org/guide |
| XLA architecture and HLO semantics | — | https://openxla.org/xla |
| Context7 MCP for library/API lookups | — | context7-mcp skill |

## What To Explore Next

| Priority | Area / Path | Why Next | Suggested Artifact |
|---|---|---|---|
| high | `tensorflow/core/` | Foundation of all execution | route-local overview |
| high | `tensorflow/python/` | Primary user API | route-local overview |
| high | `tensorflow/compiler/` | Performance-critical complexity | route-local overview |
| high | `tensorflow/lite/` | Independent runtime ecosystem | route-local overview |
| medium | `tensorflow/c/` | Stable ABI boundary | route-local overview |
| medium | `tensorflow/cc/` | C++ developer API | route-local overview |
| medium | `tensorflow/dtensor/` | Distributed training mesh | route-local overview |

## Needs Verification

- Exact op and kernel count (scout reports approximate from directory listing)
- Go/Java binding maintenance status (from README warnings, not external confirmation)
- HLO ownership boundary between `compiler/` and `third_party/xla/`

## Update History

- 2026-05-17: Initial root overview from full-bootstrap scout phase
