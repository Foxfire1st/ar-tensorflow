# Scout Report — tensorflow

Generated: 2026-05-17 | Phase 1 | Confidence: [HIGH] [MEDIUM] [LOW] tags throughout

---

## 1. Repository Overview

TensorFlow is an end-to-end open-source ML platform. The repo is organized into a C++ runtime core (`core/`), Python API layer (`python/`), XLA compiler stack (`compiler/`), mobile runtime (`lite/`), language bindings (`c/`, `cc/`, `go/`, `java/`, `js/`), distributed training (`dtensor/`), and supporting tooling.

**Root-level key files:**
- `README.md` — Platform overview, install guide
- `configure.py` — Interactive build configuration (Bazel, CUDA, etc.)
- `WORKSPACE` — Bazel workspace with third-party deps via `tf_http_archive`
- `MODULE.bazel` — Experimental Bzlmod module config

---

## 2. Area-by-Area Findings

### 2.1 `tensorflow/core/` — C++ Runtime Core [HIGH]

**Scale:** Very large | **Priority:** CRITICAL

The heart of TensorFlow. Contains the graph data structure, execution engine, op framework, kernels, device abstraction, memory management, distributed runtime, and profiling.

**Key Abstractions:**
| Concept | Defining File | Purpose |
|---|---|---|
| Graph | `core/graph/graph.h` | DAG of nodes/edges; the computation IR |
| Tensor | `core/framework/tensor.h` | N-dimensional typed array with ref-counted buffer |
| OpKernel | `core/framework/op_kernel.h` | Base class for op implementations; `Compute()` contract |
| Device | `core/framework/device.h` | CPU/GPU/TPU abstraction; owns allocators, thread pools |
| Session | `core/public/session.h` | User-facing API: Create/Extend/Run graphs |
| Executor | `core/common_runtime/executor.h` | Runs graph; manages pending counts, rendezvous |
| DirectSession | `core/common_runtime/direct_session.h` | Local single-machine session implementation |

**Internal Boundaries:**
```
PUBLIC (public/) → FRAMEWORK (framework/) → GRAPH (graph/) → EXECUTION (common_runtime/) → KERNELS (kernels/)
                                                                  ↓
                                                    DISTRIBUTED (distributed_runtime/)
```

**Sub-areas:** `framework/`, `graph/`, `common_runtime/`, `kernels/`, `ops/`, `distributed_runtime/`, `platform/`, `lib/`, `data/`, `grappler/`, `profiler/`, `protobuf/`, `public/`, `transforms/`, `tfrt/`, `tpu/`, `debug/`, `nccl/`, `summary/`, `ir/`, `function/`

---

### 2.2 `tensorflow/python/` — Python API Layer [HIGH]

**Scale:** Large | **Priority:** CRITICAL

The primary user-facing API. Exposes Tensor, Variable, GradientTape, tf.function, Keras, distribution strategies, data pipelines, and SavedModel.

**Key Abstractions:**
| Concept | Defining Module | Purpose |
|---|---|---|
| Tensor | `framework/tensor.py` | Eager & graph-mode tensor |
| Variable | `ops/resource_variable_ops.py` | Mutable state container |
| Graph | `framework/ops.py` | Computation DAG (graph mode) |
| GradientTape | `eager/tape.py` | Automatic differentiation context |
| tf.function | `eager/def_function.py` | Python→graph tracing decorator |
| Keras Model | `keras/engine/training.py` | High-level fit/predict/evaluate |

**Internal Boundaries:**
```
framework/ (graph abstractions) → eager/ (execution, AD, tf.function)
ops/ (500+ op implementations)  → keras/ (high-level API)
training/ (v1 optimizers)       → distribute/ (multi-GPU/TPU)
autograph/ (Python→graph)       → saved_model/ (serialization)
data/ (tf.data pipelines)       → compiler/ (XLA/MLIR integration)
```

---

### 2.3 `tensorflow/compiler/` — XLA Compiler Stack [HIGH]

**Scale:** Large | **Priority:** HIGH

Multi-pathway compiler stack: JIT (runtime clustering→compilation), AOT (offline to object files), and MLIR (modern IR pathway via StableHLO).

**Key Abstractions:**
| Concept | Defining File | Purpose |
|---|---|---|
| XlaCompiler | `tf2xla/xla_compiler.h` | Central orchestrator: TF→HLO compilation |
| MarkForCompilationPass | `jit/mark_for_compilation_pass.h` | Identifies compilable op clusters |
| EncapsulateSubgraphsPass | `jit/encapsulate_subgraphs_pass.h` | Wraps clusters into TF functions |
| XlaDevice | `jit/xla_device.h` | XLA execution device (CPU/GPU/TPU) |
| TfToStablehlo | `mlir/tensorflow_to_stablehlo/` | Modern MLIR: TF→StableHLO |

**Compilation Pathways:**
```
JIT:  TF Graph → clustering → wrapped functions → XlaCompiler → HLO → native executable
AOT:  TF Graph → CompileGraph() → HLO → object file + metadata
MLIR: TF SavedModel → MLIR TF dialect → StableHLO → backend code
```

**Sub-areas:** `jit/`, `tf2xla/`, `aot/`, `mlir/`, `tf2tensorrt/`, `plugin/`, `tests/`

**Key boundary:** HLO IR and XLA backends live in `third_party/xla/xla/` (external), not in `compiler/`.

---

### 2.4 `tensorflow/lite/` — TensorFlow Lite [HIGH]

**Scale:** Large | **Priority:** HIGH

Mobile and embedded ML runtime. Loads `.tflite` FlatBuffer models, resolves ops via OpResolver, schedules subgraph execution, supports hardware acceleration via delegates.

**Key Abstractions:**
| Concept | Defining File | Purpose |
|---|---|---|
| Interpreter | `core/interpreter.h` | Top-level graph execution engine |
| Subgraph | `core/subgraph.h` | Single execution unit with nodes/tensors |
| FlatBufferModel | `core/model_builder.h` | Deserializes `.tflite` files |
| InterpreterBuilder | `core/interpreter_builder.h` | Model→interpreter construction |
| OpResolver | `core/api/op_resolver.h` | Maps op codes→implementations |
| TfLiteContext/Tensor/Registration | `c/common.h` | Stable C ABI for ops and delegates |
| Memory Arena | `simple_memory_arena.h` | Dynamic tensor allocation |

**Internal Boundaries:**
```
core/ (runtime) ↔ kernels/ (100+ builtin ops) via OpResolver + TfLiteRegistration
core/ (runtime) ↔ delegates/ (GPU, NNAPI, XNNPACK, etc.) via TfLiteDelegate
schema/ (FlatBuffer) ↔ core/ via model_builder.h
tools/ (converter, optimizer) ↔ core/ (consumer of .tflite files)
micro/ (embedded) — separate minimal codebase, no shared code with core/
```

**Sub-areas:** `core/`, `c/`, `kernels/`, `delegates/`, `schema/`, `profiling/`, `tools/`, `async/`, `micro/`, `python/`, `java/`, `swift/`, `objc/`, `experimental/`

---

### 2.5 `tensorflow/c/` — C API [HIGH]

**Scale:** Medium | **Priority:** HIGH

Stable, language-agnostic C bindings. Foundation for all other language bindings. Wraps core TF in `c_api.h`/`c_api.cc`.

**Key abstractions:** `TF_Graph`, `TF_OperationDescription`, `TF_Tensor`, `TF_Status`, `TF_Session`, `TF_Buffer`

---

### 2.6 `tensorflow/cc/` — C++ API [MEDIUM]

**Scale:** Medium | **Priority:** MEDIUM

Modern C++ DSL with RAII, operator overloading, and `Scope` builder pattern. Wraps the C API.

**Key files:** `client_session.h` (execution), `scope.h` (graph construction), auto-generated `ops::*` wrappers.

---

### 2.7 `tensorflow/dtensor/` — Distributed Tensor [HIGH]

**Scale:** Medium | **Priority:** MEDIUM

Tensor sharding across device meshes for large-scale distributed training. Has `cc/`, `mlir/`, `proto/`, `python/` sub-areas. Used by Keras for multi-GPU/TPU.

---

### 2.8 Smaller/Peripheral Areas

| Area | Purpose | Priority | Notes |
|---|---|---|---|
| `js/` | JS codegen bridge for TF.js | LOW | Thin wrapper; main code in external tfjs repo |
| `go/` | Go bindings | LOW | Unmaintained; warns of API instability |
| `java/` | Java bindings | SKIP | Deprecated; redirects to github.com/tensorflow/java |
| `security/` | Vulnerability advisories + fuzzing | REFERENCE | 100+ TFSA advisories; OSS-Fuzz integration |
| `tools/` | Build/packaging/CI infrastructure | DEPENDS | pip_package, ci_build, benchmark, graph_transforms |
| `examples/` | C++ example code | REFERENCE | Not actively maintained; reference quality |
| `docs_src/` | (empty) | SKIP | Moved to github.com/tensorflow/docs |
| `ci/` (root) | CI/CD configs | LOW | Build/test infrastructure |
| `tools/` (root) | Repo-level tools | LOW | Bazel helpers, build scripts |

---

## 3. Area Priority Ranking

| Rank | Area | Rationale |
|---|---|---|
| 1 | `core/` | Foundation of everything; all other layers depend on it |
| 2 | `python/` | Primary user API; most developer-facing code |
| 3 | `compiler/` | Performance-critical; multi-pathway complexity |
| 4 | `lite/` | Independent runtime; growing mobile/edge importance |
| 5 | `c/` | Stable ABI; all bindings foundation |
| 6 | `cc/` | C++ developer API |
| 7 | `dtensor/` | Distributed training; growing in importance |
| 8 | `js/` | JS ecosystem bridge |
| 9-14 | `go/`, `java/`, `security/`, `tools/`, `examples/`, `docs_src/` | Peripheral, deprecated, or reference only |

---

## 4. Cross-Cutting Concerns

| Concern | Spans |
|---|---|
| **Automatic differentiation** | `python/eager/` (GradientTape), `core/common_runtime/gradients.h`, `python/ops/*_grad.py` |
| **Graph optimization** | `core/grappler/`, `compiler/jit/`, `compiler/mlir/` |
| **Device placement** | `core/common_runtime/placer.*`, `compiler/jit/`, `python/framework/device.py` |
| **Serialization** | `python/saved_model/`, `lite/schema/`, `core/protobuf/` |
| **C/C++ binding layer** | `python/pywrap_*.py`, `c/c_api.*`, `cc/` |
| **Distributed execution** | `core/distributed_runtime/`, `dtensor/`, `python/distribute/` |

---

## 5. Confidence Summary

| Finding | Confidence |
|---|---|
| Core architecture (Graph→Executor→Kernel pipeline) | [HIGH] — verified from headers |
| Python API organization (framework/eager/ops/keras) | [HIGH] — verified from __init__.py and module structure |
| Compiler multi-pathway (JIT/AOT/MLIR) | [HIGH] — verified from headers and BUILD files |
| Lite runtime structure (Interpreter→Subgraph→OpResolver) | [HIGH] — verified from core headers |
| C API as stable binding foundation | [HIGH] — verified from c_api.h |
| dtensor mesh sharding model | [MEDIUM] — from directory structure, not code-verified |
| Go/Java binding status | [MEDIUM] — from README warnings; not verified against maintenance history |
| Exact op count and kernel coverage | [LOW] — approximate from directory listing, not exhaustive |
