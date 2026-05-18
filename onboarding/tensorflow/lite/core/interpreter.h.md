# interpreter.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/interpreter.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `Interpreter` — the top-level execution engine for TensorFlow Lite. It loads a `.tflite` FlatBuffer model, resolves ops via an `OpResolver`, allocates tensors, and provides the `Invoke()` method to run inference. Supports multi-subgraph models (e.g., for control flow), signature-based execution, delegate-based hardware acceleration (GPU, NNAPI, XNNPACK), and cancellation.

This is the main entry point for TFLite inference — the equivalent of `Session` in the full TF runtime, but for mobile/embedded use.

## Code Commentary

### Logic

**`Interpreter` class:** Key methods:
- Construction from `FlatBufferModel` + `OpResolver`. Optional `ErrorReporter` for logging.
- `AllocateTensors()` — memory-plans and allocates all tensor buffers.
- `Invoke()` — runs the primary subgraph. Thread-safe for concurrent calls (with caveats).
- `ResizeInputTensor(index, shape)` — resizes an input tensor (triggers re-allocation on next `AllocateTensors`).
- `SetTensorParametersReadWrite(index, type, ...)` — customizes tensor allocation.
- `ModifyGraphWithDelegate(delegate)` — applies a hardware delegate (GPU, NNAPI, etc.).
- `AddCustomOp(name, registration)` — registers a custom op.
- `UseNNAPI(enable)` / `SetNumThreads(n)` — configuration.
- `GetSignatureRunner(key)` — returns a `SignatureRunner` for named-signature execution.

### Conventions

- **Model → Interpreter → Invoke** is the standard inference pipeline.
- **Delegates are plugged in after construction** but before `AllocateTensors()`.

### Invariants And Boundaries

- AllocateTensors() must be called before any Invoke().
- Delegates modify the graph before AllocateTensors(); ordering matters.
- Input tensor dimensions must match the model's expected shapes after ResizeInputTensor.
- Output tensors are stable between Invoke() calls; their data is overwritten, not reallocated.

- **Independent from full TF** — does not use `core/` Executor, Session, or Graph. Has its own graph representation, memory planner, and op dispatch.
- **FlatBuffer model is immutable** — the model is read-only after loading; tensors are mutable.
- **Thread safety** — multiple `Invoke()` calls can run concurrently, but tensor data access must be externally synchronized.
- **Thread safety is limited** — Invoke() is not safe to call concurrently. Multiple threads calling Invoke() on the same Interpreter causes data races on tensor buffers. Use one Interpreter per thread.
- **Delegate must be applied before AllocateTensors()** — Calling ModifyGraphWithDelegate() after AllocateTensors() has no effect (or causes inconsistent state). The delegate rewrites the execution plan, which is computed during allocation.
- **Zero-copy input has lifetime constraints** — Tensors set via typed_input_tensor() must outlive the Interpreter invocation that reads them. The Interpreter does not copy input data.
- **ResizeInputTensor invalidates allocations** — Calling ResizeInputTensor() after AllocateTensors() requires calling AllocateTensors() again. Any tensor pointers obtained before resize are invalidated.
- **SignatureDef vs index-based access** — Using signature-based methods (GetSignatureRunner) is safer than index-based (input(), output()). Indices change if the model graph changes.
- **Cancellation is cooperative** — SetCancellationFunction() sets a callback, but Invoke() only checks it between op executions. A slow single op cannot be interrupted.
- **Memory arena is fixed-size** — After AllocateTensors(), the arena is fully allocated. Adding a delegate that requires more scratch memory fails at apply time.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite inference: loading models, running, delegates | — | https://www.tensorflow.org/lite/guide/inference |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Subgraph handles per-subgraph execution | Subgraph class | [`tensorflow/lite/core/subgraph.h`](../tensorflow/lite/core/subgraph.h.md) |
| FlatBufferModel loads the .tflite file | model building | [`tensorflow/lite/core/model_builder.h`](../tensorflow/lite/core/model_builder.h.md) |
| OpResolver maps op codes to TfLiteRegistration | resolver interface | [`tensorflow/lite/core/api/op_resolver.h`](../tensorflow/lite/core/api/op_resolver.h.md) |
| TfLiteContext/TfLiteTensor are the stable C ABI types | C common types | [`tensorflow/lite/core/c/common.h`](../tensorflow/lite/core/c/common.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
