# Coverage Plan — tensorflow

Generated: 2026-05-17 | Phase 4A

## Plan Summary

Full-bootstrap coverage for tensorflow. Priorities driven by scout report: core, python, compiler, lite get route-local overviews + file-level onboarding. Smaller and peripheral areas get deferred or reference-only treatment.

## Coverage Decisions

| Area | Source Route | Route-Local Overview | File-Level Onboarding | Docs Evidence | Cross-Repo Boundary |
|---|---|---|---|---|---|
| Core Runtime | `tensorflow/core/` | **yes** | top 10 files | no (domain docs only) | no (vendored xla) |
| Python API | `tensorflow/python/` | **yes** | top 10 files | no (domain docs only) | no |
| Compiler | `tensorflow/compiler/` | **yes** | top 6 files + MLIR legalization expansion | no (domain docs only) | no (xla is vendored) |
| Lite | `tensorflow/lite/` | **yes** | top 8 files | no (domain docs only) | no |
| C API | `tensorflow/c/` | **yes** | top 3 files + StreamExecutor plugin expansion | no | no |
| C++ API | `tensorflow/cc/` | deferred | deferred | no | no |
| DTensor | `tensorflow/dtensor/` | deferred | deferred | no | no |
| JS | `tensorflow/js/` | deferred | deferred | no | no |
| Go | `tensorflow/go/` | skip | skip | no | no |
| Java | `tensorflow/java/` | skip | skip | no | no |
| Security | `tensorflow/security/` | reference-only | skip | no | no |
| Tools | `tensorflow/tools/` | deferred | deferred | no | no |
| Examples | `tensorflow/examples/` | skip | skip | no | no |
| docs_src | `tensorflow/docs_src/` | skip | skip | no | no |

## Route-Local Overview Waves

### Wave 1 (high priority)
- `tensorflow/core/overview.md`
- `tensorflow/python/overview.md`
- `tensorflow/compiler/overview.md`
- `tensorflow/lite/overview.md`

### Wave 2 (medium priority)
- `tensorflow/c/overview.md`
- `tensorflow/cc/overview.md` (deferred)
- `tensorflow/dtensor/overview.md` (deferred)

## File Onboarding Waves

### Wave 1 — Core load-bearing files
| File | Reason |
|---|---|
| `tensorflow/core/graph/graph.h` | Core DAG abstraction |
| `tensorflow/core/framework/tensor.h` | Tensor data container |
| `tensorflow/core/framework/op_kernel.h` | Op execution contract |
| `tensorflow/core/framework/device.h` | Device abstraction |
| `tensorflow/core/common_runtime/executor.h` | Graph execution engine |
| `tensorflow/core/public/session.h` | User-facing Session API |
| `tensorflow/core/framework/op.h` | Op registry |
| `tensorflow/core/common_runtime/direct_session.h` | Local session implementation |
| `tensorflow/core/framework/function.h` | Function library runtime |
| `tensorflow/core/framework/device_base.h` | Device base class |

### Wave 2 — Python load-bearing files
| File | Reason |
|---|---|
| `tensorflow/python/__init__.py` | Main API entry point |
| `tensorflow/python/framework/tensor.py` | Tensor class |
| `tensorflow/python/framework/ops.py` | Graph, Operation classes |
| `tensorflow/python/eager/context.py` | Eager execution context |
| `tensorflow/python/eager/def_function.py` | tf.function decorator |
| `tensorflow/python/eager/backprop.py` | Gradient tape |
| `tensorflow/python/ops/resource_variable_ops.py` | Variable class |
| `tensorflow/python/keras/engine/training.py` | Keras Model |
| `tensorflow/python/framework/framework_lib.py` | Public re-exports |
| `tensorflow/python/ops/standard_ops.py` | Standard ops hub |

### Wave 3 — Compiler load-bearing files
| File | Reason |
|---|---|
| `tensorflow/compiler/tf2xla/xla_compiler.h` | Central XLA compilation orchestrator |
| `tensorflow/compiler/jit/device_compiler.h` | Device compilation wrapper |
| `tensorflow/compiler/jit/mark_for_compilation_pass.h` | Op clustering pass |
| `tensorflow/compiler/jit/xla_device.h` | XLA device abstraction |
| `tensorflow/compiler/tf2xla/graph_compiler.h` | Graph→HLO lowering |
| `tensorflow/compiler/aot/compile.h` | AOT compilation entry |

### Wave 3A — MLIR legalization expansion
| File | Reason |
|---|---|
| `tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td` | Declarative TF→StableHLO/CHLO legalization patterns generated into the TF2XLA legalization pass |
| `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td` | Declarative TF→MHLO legalization patterns used by the StableHLO-oriented pass pipeline before StableHLO conversion |

### Wave 4 — Lite load-bearing files
| File | Reason |
|---|---|
| `tensorflow/lite/core/interpreter.h` | Interpreter — main execution engine |
| `tensorflow/lite/core/subgraph.h` | Subgraph execution unit |
| `tensorflow/lite/core/model_builder.h` | FlatBuffer model deserialization |
| `tensorflow/lite/core/interpreter_builder.h` | Model→interpreter construction |
| `tensorflow/lite/c/common.h` | Stable C ABI types |
| `tensorflow/lite/core/api/op_resolver.h` | Op resolution abstraction |
| `tensorflow/lite/simple_memory_arena.h` | Dynamic tensor allocation |
| `tensorflow/lite/schema/schema_generated.h` | FlatBuffer schema |

### Wave 5 — C API load-bearing files
| File | Reason |
|---|---|
| `tensorflow/c/c_api.h` | Public C API surface |
| `tensorflow/c/c_api.cc` | C API implementation |
| `tensorflow/c/tf_tensor.h` | TF_Tensor type definition |

### Wave 5A — Pluggable device stream priority expansion
| File | Reason |
|---|---|
| `tensorflow/c/experimental/stream_executor/stream_executor.h` | Experimental StreamExecutor C ABI declares stream options and option-aware stream creation |
| `tensorflow/c/experimental/stream_executor/stream_executor.cc` | C-to-StreamExecutor adapter forwards optional stream priority into plugin stream creation |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device.h` | PluggableDevice lifecycle declaration accepts optional stream priority during initialization |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc` | PluggableDevice stream group creation applies optional priority to compute and copy streams |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h` | Factory declaration carries optional priority through private device creation |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc` | Factory parses virtual-device priorities, validates priority shape, and creates executors before device descriptions |

## Docs Evidence

No dedicated docs evidence packs needed. Domain docs (tensorflow.org, openxla.org) are external web resources. Context7 MCP available for on-demand API lookups.

## Cross-Repo Boundaries

No cross-repo boundaries active. `crossRepo.allow` is empty. XLA backends are vendored under `third_party/xla/`, not a separate repo.
