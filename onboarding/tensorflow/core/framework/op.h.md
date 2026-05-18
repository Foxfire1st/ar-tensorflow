# op.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/op.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `OpRegistry` (L60-L145) — the thread-safe, global singleton registry mapping operation type names to their `OpDef` (semantic definition) and `OpRegistrationData`. This is where `REGISTER_OP("MatMul").Input(...).Output(...)` registrations land. Also defines `OpRegistryInterface` (L36-L49) for read-only lookups and `OpListOpRegistry` (L147-L165) for loading from a serialized `OpList`. The `REGISTER_OP` / `REGISTER_SYSTEM_OP` macros (L224-L236) are the primary mechanism for op authors to define new operations.

## Code Commentary

### Logic

**OpRegistryInterface (L36-L49):** Abstract read interface. `LookUp(op_type_name, &op_reg_data)` returns the full registration data (OpDef + shape inference + type functions). `LookUpOpDef(name, &op_def)` is a convenience wrapper returning just the OpDef.

**OpRegistry (L60-L145):** Thread-safe singleton with deferred registration and watcher patterns. Key methods:

- `Register(factory)` (L70) — Takes a factory lambda `Status(OpRegistrationData*)`. If registrations are deferred, pushes to `deferred_` (L136). Otherwise, calls immediately and notifies watcher.
- `LookUp(name, &data)` (L74) — First calls `MustCallDeferred()` to process any pending registrations, then does a `flat_hash_map` lookup in `registry_`. Thread-safe (`mu_` guarded).
- `Export(include_internal, OpList*)` (L80) — Exports all registered OpDefs, excluding internal ops (names starting with `_`) unless `include_internal=true`. Sorted alphabetically.
- `DeferRegistrations()` (L118) — Suspends immediate registration processing for batch-loading.
- `ProcessRegistrations()` (L114) — Forces processing of deferred registrations; also invoked implicitly by LookUp, Export, and DebugString.
- `SetWatcher / Watcher` (L99-L104) — A callback invoked on every op registration. Not thread-safe; clients must atomically `SetWatcher → register → ProcessRegistrations → SetWatcher(nullptr)`.
- `Global()` (L84) — Returns the process-wide singleton registry. Available from static initialization time.

**OpListOpRegistry (L147-L165):** Adapter making a serialized `OpList` behave like an `OpRegistryInterface`. Does NOT provide shape inference functions — only useful for op-name lookups where shape inference is not needed.

**REGISTER_OP / REGISTER_SYSTEM_OP Macros (L224-L236):** Fluent API using `OpDefBuilderWrapper`:

```cpp
REGISTER_OP("MyOp")
    .Input("x: float")
    .Output("y: float")
    .SetShapeFn(shape_inference::UnchangedShape)
    .Doc("My custom operation");
```

- `REGISTER_OP` — Conditional registration; may be omitted under selective registration (`SHOULD_REGISTER_OP` check).
- `REGISTER_SYSTEM_OP` — Always registered, even under selective registration.
- Uses `InitOnStartupMarker` to guarantee registration before `main()`.

### Conventions

- **Global singleton** — `OpRegistry::Global()` is the process-wide instance, accessed exclusively through that function.
- **Deferred registration pattern** — Registrations can be batched via `DeferRegistrations()` / `ProcessRegistrations()`, reducing hashtable write overhead during library loading.
- **Internal op convention** — Op names starting with `_` are considered internal and excluded from `Export()` by default. This is a convention, not enforced structurally by the registry.

### Invariants And Boundaries

- Op names are globally unique — registering a duplicate name returns an error.
- `LookUp` always processes deferred registrations first; callers never see stale state.
- The `Global()` singleton is available from static initialization time.
- `OpDefBuilderWrapper::operator()` triggers registration via the `InitOnStartupMarker` chain.
- Watcher is single-client and not thread-safe; concurrent watcher sets are undefined behavior.- **Static initialization order** — `REGISTER_OP` macros use `InitOnStartupMarker`; they run before `main()` but ordering between translation units is undefined. Op registrations depending on other ops being registered first can fail silently.
- **Deferred registrations must be processed** — If you call `DeferRegistrations()` and forget to call `ProcessRegistrations()`, subsequent `LookUp` calls implicitly process them, but watcher callbacks are delayed.
- **OpListOpRegistry has no shape inference** — Using it where shape functions are expected produces invalid `OpRegistrationData`. Only use for display/export.
- **Watcher is single-client** — Setting a watcher overwrites the previous one. Designed for test instrumentation, not production monitoring.
- **Internal ops (`_` prefix)** — Excluded from `Export` unless `include_internal=true`. This is a convention, not enforced by the registry.

### Todos

No known TODOs specific to this file.

## Docs References

TensorFlow's public documentation covers the op registration API used by this file, including the guide for adding new operations and the C++ REGISTER_OP macro reference.

| Finding | Citations | Source Path |
|---|---|---|
| Guide to adding a new op (Python + C++), covering REGISTER_OP usage | — | [TF Guide: Create an op](https://www.tensorflow.org/guide/create_op) |
| REGISTER_OP macro documentation in TF C++ API reference | — | [TF C++: ops group](https://www.tensorflow.org/api_docs/cc/group/ops) |

## Repo-Internal References

This file is the central op registry. It is consumed by the kernel registration system, the function system, and the graph construction pipeline.

| Finding | Citations | Source Path |
|---|---|---|
| OpDef protobuf — the schema for op definitions | op_def.proto | [`tensorflow/core/framework/op_def.proto`](../../../../tensorflow/core/framework/op_def.proto) |
| OpDefBuilder fluent API used by REGISTER_OP | OpDefBuilder | [`tensorflow/core/framework/op_def_builder.h`](../../../../tensorflow/core/framework/op_def_builder.h) |
| FunctionDef uses OpDef for function signatures | FunctionDef | [`function.h`](function.h.md) |
| Shape inference functions registered per op | shape_inference | [`tensorflow/core/framework/shape_inference.h`](../../../../tensorflow/core/framework/shape_inference.h) |
| Kernel registration maps (op, device) → OpKernel factory | KernelDef | [`op_kernel.h`](op_kernel.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template — merged Key Invariants and Traps into `### Invariants And Boundaries`, moved `Docs References` under `## Code Commentary`, added `### Conventions` and `### Todos` subsections, added `doc_type` metadata.
- 2026-05-17T17:30: Deep-improvement pass — added implementation-level Code Commentary with OpRegistry internals (deferred reg, watcher), REGISTER_OP macros, OpListOpRegistry, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
