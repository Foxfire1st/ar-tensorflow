# common.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/c/common.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines the stable C ABI for TensorFlow Lite ops, delegates, and tensor types. This is the foundational header that all TFLite ops and delegate implementations include. Defines `TfLiteTensor` (the runtime tensor struct), `TfLiteContext` (the op execution context interface), `TfLiteRegistration` (the op registration/init/prepare/invoke function pointers), `TfLiteNode` (a node in the execution plan), `TfLiteDelegate` (delegate interface), and the status/type enums.

This is the boundary between the TFLite runtime core and the op/delegate ecosystem.

## Code Commentary

### Logic

Key types:
- **`TfLiteTensor`** — runtime tensor: `data` (union of raw/eigen int8/float/int32/int64 pointers), `dims`, `type`, `params` (scale/zero_point for quantization), `allocation_type`, `quantization`.
- **`TfLiteContext`** — execution context: `GetTensor()`, `ResizeTensor()`, `GetExternalContext()`, `ReportError()`. Passed to op `prepare()`/`invoke()`.
- **`TfLiteRegistration`** — function pointer table: `init`, `free`, `prepare`, `invoke`, `profiling_string`. Each op/kernel provides one.
- **`TfLiteNode`** — op instance: `inputs`, `outputs`, `builtin_data`, `custom_initial_data`, `delegate`, `registration`.
- **`TfLiteDelegate`** — delegate interface: `Prepare()`, `CopyFromBufferHandle()`, `FreeBufferHandle()`.
- **`kTfLiteOk` / `kTfLiteError`** — status codes.

### Conventions

- **Plain C ABI** — all types are C structs/enums, enabling bindings from any language.
- **Function pointer dispatch** — op behavior is defined entirely through function pointers, not virtual methods.

### Invariants And Boundaries

- TfLiteTensor is valid only between AllocateTensors() and the next ResizeInputTensor or interpreter destruction.
- TfLiteNode::inputs/outputs arrays are indexed by TfLiteRegistration-specified positions.
- TfLiteDelegate::Prepare is called once, before AllocateTensors. All node replacements happen here.

- **Stable ABI** — struct layouts and enum values are versioned. Breaking changes require new API versions.
- **Delegates must respect the ABI** — custom delegates implement `TfLiteDelegate` with the same function pointer contracts.
- **ABI stability is a hard contract** — All struct layouts defined here are part of the stable C ABI. Changing field order, adding fields in the middle, or changing sizes breaks all pre-compiled delegate libraries. Only append to the end of structs with version guards.
- **TfLiteTensor::data is a union** — The same bytes represent different types depending on TfLiteTensor::type. Reading data.f when type is kTfLiteInt32 reads garbage. Always check type before accessing data.
- **TfLiteRegistration::version** — Each op has a version. The runtime calls init/prepare/invoke for the specific version. Custom ops must handle version negotiation.
- **Allocation type determines ownership** — kTfLiteArenaRw tensors are owned by the interpreter's arena. kTfLiteMmapRo tensors are read-only model buffer pointers. Freeing the model buffer invalidates mmap tensors.
- **TfLiteContext::GetExternalContext** — Used by delegates to inject backend-specific state (e.g., TfLiteGpuAcceleratorInstance). Not all delegates set this; checking for null is mandatory.
- **Error reporting is limited** — TfLiteContext::ReportError writes to stderr by default. In production, redirect via a custom error reporter or errors are lost.
- **Struct size compatibility** — The sizeof() these structs is checked at delegate registration time. A delegate compiled against a different TFLite version is rejected at load.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite custom operators guide | — | https://www.tensorflow.org/lite/guide/ops_custom |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Subgraph uses TfLiteNode/TfLiteTensor for execution | Subgraph class | [`tensorflow/lite/core/subgraph.h`](../tensorflow/lite/core/subgraph.h.md) |
| OpResolver returns TfLiteRegistration for each op | resolver | [`tensorflow/lite/core/api/op_resolver.h`](../tensorflow/lite/core/api/op_resolver.h.md) |
| All builtin ops (100+) implement TfLiteRegistration | kernels | `tensorflow/lite/kernels/` |
| Delegates (GPU, NNAPI, XNNPACK) implement TfLiteDelegate | delegate classes | `tensorflow/lite/delegates/` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
