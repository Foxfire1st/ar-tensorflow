# model_builder.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/model_builder.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `FlatBufferModel` — the TFLite model loader. Takes a `.tflite` file (FlatBuffer binary) or a raw memory buffer and deserializes it into an in-memory representation. Provides verification and error reporting. The loaded model is immutable and serves as the input to `InterpreterBuilder` for constructing an `Interpreter`.

## Code Commentary

### Logic

**`FlatBufferModel`:** Inherits from `FlatBufferModelBase<FlatBufferModel>` (defined in MLIR-based core). Key static factory methods (from the base):
- `BuildFromFile(filename)` — loads from disk.
- `BuildFromBuffer(data, size)` — loads from memory.
- `VerifyAndBuildFromFile(filename)` / `VerifyAndBuildFromBuffer(...)` — loads with verification.

The base class handles:
- FlatBuffer parsing and validation.
- Model metadata extraction (version, description, subgraph count).
- The `GetModel()` method returns a pointer to the parsed `tflite::Model` struct.

### Conventions

- **Immutable after loading** — no mutation methods.
- **Verification via FlatBuffer** — `VerifyAndBuild` variants check the model's FlatBuffer integrity.

### Invariants And Boundaries

- FlatBufferModel wraps a read-only buffer — no mutation of model structure at runtime.
- BuildFromFile() mmaps the file on supported platforms; BuildFromBuffer() copies.
- The model buffer contains the complete TFLite graph: ops, tensors, metadata.

- **.tflite = FlatBuffer** — the serialized format is FlatBuffers, not protobuf. This enables zero-copy parsing on embedded devices.
- **Model outlives Interpreter** — the `FlatBufferModel` must remain alive for the lifetime of the `Interpreter` that references it.
- **mmap vs copy semantics** — BuildFromFile() uses mmap on POSIX systems; the file must not be truncated while interpreters exist. On Windows it copies.
- **Verification is partial** — BuildFromFile() verifies the FlatBuffer format, not semantic correctness. Invalid models (wrong op versions, impossible shapes) pass verification and fail at Build() or Invoke().
- **64-bit size limits** — Models >2GB are theoretically supported in the FlatBuffer format but may fail allocation on 32-bit systems.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite model conversion and format | — | https://www.tensorflow.org/lite/models/convert |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| InterpreterBuilder consumes FlatBufferModel to create Interpreter | builder class | [`tensorflow/lite/core/interpreter_builder.h`](../tensorflow/lite/core/interpreter_builder.h.md) |
| FlatBufferModelBase is the MLIR-based core implementation | base class | `tensorflow/compiler/mlir/lite/core/model_builder_base.h` |
| tflite::Model is the parsed FlatBuffer schema | schema | `tensorflow/lite/schema/schema_generated.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
