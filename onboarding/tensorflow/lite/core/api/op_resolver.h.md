# op_resolver.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/core/api/op_resolver.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `OpResolver` — the abstract interface that maps op codes (integers builtin/custom) to `TfLiteRegistration` structs. The `InterpreterBuilder` uses the resolver to find the implementation of each op in the model. `MutableOpResolver` adds the ability to register custom ops at runtime. `BuiltinOpResolver` is the standard implementation that includes all builtin TFLite ops.

## Code Commentary

### Logic

**`OpResolver`:**
- `FindOp(op, version)` → `const TfLiteRegistration*` — returns the registration for an op code and version.
- `GetRegistrationFromOpCode(opcode, version)` — variant taking a flatbuffer `OperatorCode`.

**`MutableOpResolver`:**
- `AddBuiltin(op, registration, min_version, max_version)` — registers a builtin op.
- `AddCustom(name, registration, min_version, max_version)` — registers a custom op by name string.
- `AddAll(builtin_registrations, count)` — bulk registration.

### Conventions

- **Versioned ops** — each op registration supports a version range `[min_version, max_version]`. The interpreter builder resolves the correct version at load time.
- **Chained resolvers** — `OpResolver::Chain()` composes multiple resolvers; first match wins.

### Invariants And Boundaries

- OpResolver maps op name + version → TfLiteRegistration (kernel implementation).
- BuiltinOpResolver includes all built-in ops; MutableOpResolver allows runtime addition.
- One resolver per Interpreter; shared across Subgraphs.- **Version negotiation** — If the model uses op version 2 but the resolver only has version 1, Build() fails. Always use the resolver matching the converter version.
- **Custom op registration order** — registerCustomOp() must be called before Build(). Registrations after Build() are silently ignored.
- **BuiltinOpResolver is large** — Including all built-in ops adds ~300K to binary size. Use SelectiveBuiltinOpResolver for production.
- **Delegate resolvers** — DelegateCreate MUST set all fields of TfLiteRegistration, especially init/free/prepare/invoke. Missing invoke = nullptr causes crash.

### Todos

No known TODOs specific to this file.

## Docs References

TFLite op resolver documentation covers custom op registration and built-in op availability.

| Finding | Citations | Source Path |
|---|---|---|
| TFLite op resolver and custom ops | — | [TFLite: Custom ops](https://www.tensorflow.org/lite/guide/ops_custom) |

## Repo-Internal References

OpResolver is the op lookup interface consumed by the InterpreterBuilder. Every TFLite model load passes through it.

| Finding | Citations | Source Path |
|---|---|---|
| InterpreterBuilder uses OpResolver to find op implementations | builder class | [`interpreter_builder.h`](../interpreter_builder.h.md) |
| TfLiteRegistration is the struct returned by FindOp | registration type | [`common.h`](../c/common.h.md) |
| BuiltinOpResolver includes all 100+ builtin TFLite ops | builtin resolver | `tensorflow/lite/mutable_op_resolver.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template — added `### Invariants And Boundaries` section, merged floating invariants and traps content.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4.
