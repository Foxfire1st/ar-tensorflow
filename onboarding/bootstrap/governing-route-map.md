# Governing Route Map — tensorflow

Generated: 2026-05-17 | Phase 4B

## Active Routes

| Source Route | Overview Path | Status | Confidence | Notes |
|---|---|---|---|---|
| `tensorflow/` | `overview.md` | created | [HIGH] | Root repo overview |
| `tensorflow/core/` | `tensorflow/core/overview.md` | planned | [HIGH] | C++ runtime foundation |
| `tensorflow/python/` | `tensorflow/python/overview.md` | planned | [HIGH] | Primary user API |
| `tensorflow/compiler/` | `tensorflow/compiler/overview.md` | planned | [HIGH] | XLA JIT/AOT/MLIR |
| `tensorflow/compiler/mlir/` | `tensorflow/compiler/mlir/overview.md` | created | [HIGH] | MLIR bridge, dialect, and lowering infrastructure |
| `tensorflow/compiler/mlir/tf2xla/transforms/` | `tensorflow/compiler/mlir/tf2xla/transforms/overview.md` | created | [HIGH] | TF-to-XLA MLIR transform and legalization pass area |
| `tensorflow/compiler/mlir/stablehlo/transforms/` | `tensorflow/compiler/mlir/stablehlo/transforms/overview.md` | created | [HIGH] | TF-to-StableHLO transform and legalization pass area |
| `tensorflow/lite/` | `tensorflow/lite/overview.md` | planned | [HIGH] | Mobile/embedded runtime |
| `tensorflow/c/` | `tensorflow/c/overview.md` | planned | [HIGH] | Stable C ABI |

## Deferred Routes

| Source Route | Reason | Revisit Trigger |
|---|---|---|
| `tensorflow/cc/` | Medium priority; C++ DSL | When core+python coverage complete |
| `tensorflow/dtensor/` | Specialized distributed training area | When core+python coverage complete |
| `tensorflow/js/` | Thin codegen bridge; main code external | When tfjs integration work needed |
| `tensorflow/tools/` | Build infrastructure, not consumer-facing | When build system work needed |

## Skipped Routes

| Source Route | Reason |
|---|---|
| `tensorflow/go/` | Unmaintained; README warns of API instability |
| `tensorflow/java/` | Deprecated; redirects to separate repo |
| `tensorflow/examples/` | Not actively maintained; reference quality |
| `tensorflow/docs_src/` | Empty; moved to github.com/tensorflow/docs |
| `third_party/` | Excluded by pathRules |

## File-Level Mapping

| Source File | Onboarding File | Wave | Status |
|---|---|---|---|
| `tensorflow/core/graph/graph.h` | `tensorflow/core/graph/graph.h.md` | 1 | planned |
| `tensorflow/core/framework/tensor.h` | `tensorflow/core/framework/tensor.h.md` | 1 | planned |
| `tensorflow/core/framework/op_kernel.h` | `tensorflow/core/framework/op_kernel.h.md` | 1 | planned |
| `tensorflow/core/framework/device.h` | `tensorflow/core/framework/device.h.md` | 1 | planned |
| `tensorflow/core/common_runtime/executor.h` | `tensorflow/core/common_runtime/executor.h.md` | 1 | planned |
| `tensorflow/core/public/session.h` | `tensorflow/core/public/session.h.md` | 1 | planned |
| `tensorflow/core/framework/op.h` | `tensorflow/core/framework/op.h.md` | 1 | planned |
| `tensorflow/core/common_runtime/direct_session.h` | `tensorflow/core/common_runtime/direct_session.h.md` | 1 | planned |
| `tensorflow/core/framework/function.h` | `tensorflow/core/framework/function.h.md` | 1 | planned |
| `tensorflow/core/framework/device_base.h` | `tensorflow/core/framework/device_base.h.md` | 1 | planned |
| `tensorflow/python/__init__.py` | `tensorflow/python/__init__.py.md` | 2 | planned |
| `tensorflow/python/framework/tensor.py` | `tensorflow/python/framework/tensor.py.md` | 2 | planned |
| `tensorflow/python/framework/ops.py` | `tensorflow/python/framework/ops.py.md` | 2 | planned |
| `tensorflow/python/eager/context.py` | `tensorflow/python/eager/context.py.md` | 2 | planned |
| `tensorflow/python/eager/def_function.py` | `tensorflow/python/eager/def_function.py.md` | 2 | planned |
| `tensorflow/python/eager/backprop.py` | `tensorflow/python/eager/backprop.py.md` | 2 | planned |
| `tensorflow/python/ops/resource_variable_ops.py` | `tensorflow/python/ops/resource_variable_ops.py.md` | 2 | planned |
| `tensorflow/python/keras/engine/training.py` | `tensorflow/python/keras/engine/training.py.md` | 2 | planned |
| `tensorflow/python/framework/framework_lib.py` | `tensorflow/python/framework/framework_lib.py.md` | 2 | planned |
| `tensorflow/python/ops/standard_ops.py` | `tensorflow/python/ops/standard_ops.py.md` | 2 | planned |
| `tensorflow/compiler/tf2xla/xla_compiler.h` | `tensorflow/compiler/tf2xla/xla_compiler.h.md` | 3 | planned |
| `tensorflow/compiler/jit/device_compiler.h` | `tensorflow/compiler/jit/device_compiler.h.md` | 3 | planned |
| `tensorflow/compiler/jit/mark_for_compilation_pass.h` | `tensorflow/compiler/jit/mark_for_compilation_pass.h.md` | 3 | planned |
| `tensorflow/compiler/jit/xla_device.h` | `tensorflow/compiler/jit/xla_device.h.md` | 3 | planned |
| `tensorflow/compiler/tf2xla/graph_compiler.h` | `tensorflow/compiler/tf2xla/graph_compiler.h.md` | 3 | planned |
| `tensorflow/compiler/aot/compile.h` | `tensorflow/compiler/aot/compile.h.md` | 3 | planned |
| `tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td` | `tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td.md` | 3A | created |
| `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td` | `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td.md` | 3A | created |
| `tensorflow/lite/core/interpreter.h` | `tensorflow/lite/core/interpreter.h.md` | 4 | planned |
| `tensorflow/lite/core/subgraph.h` | `tensorflow/lite/core/subgraph.h.md` | 4 | planned |
| `tensorflow/lite/core/model_builder.h` | `tensorflow/lite/core/model_builder.h.md` | 4 | planned |
| `tensorflow/lite/core/interpreter_builder.h` | `tensorflow/lite/core/interpreter_builder.h.md` | 4 | planned |
| `tensorflow/lite/c/common.h` | `tensorflow/lite/c/common.h.md` | 4 | planned |
| `tensorflow/lite/core/api/op_resolver.h` | `tensorflow/lite/core/api/op_resolver.h.md` | 4 | planned |
| `tensorflow/lite/simple_memory_arena.h` | `tensorflow/lite/simple_memory_arena.h.md` | 4 | planned |
| `tensorflow/lite/schema/schema_generated.h` | `tensorflow/lite/schema/schema_generated.h.md` | 4 | planned |
| `tensorflow/c/c_api.h` | `tensorflow/c/c_api.h.md` | 5 | planned |
| `tensorflow/c/c_api.cc` | `tensorflow/c/c_api.cc.md` | 5 | planned |
| `tensorflow/c/tf_tensor.h` | `tensorflow/c/tf_tensor.h.md` | 5 | planned |
| `tensorflow/c/experimental/stream_executor/stream_executor.h` | `tensorflow/c/experimental/stream_executor/stream_executor.h.md` | 5A | created |
| `tensorflow/c/experimental/stream_executor/stream_executor.cc` | `tensorflow/c/experimental/stream_executor/stream_executor.cc.md` | 5A | created |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device.h` | `tensorflow/core/common_runtime/pluggable_device/pluggable_device.h.md` | 5A | created |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc` | `tensorflow/core/common_runtime/pluggable_device/pluggable_device.cc.md` | 5A | created |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h` | `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.h.md` | 5A | created |
| `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc` | `tensorflow/core/common_runtime/pluggable_device/pluggable_device_factory.cc.md` | 5A | created |
