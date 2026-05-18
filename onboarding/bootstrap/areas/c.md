# Area Report — c

| Field | Value |
|---|---|
| repo | tensorflow |
| area | c |
| sourceRoute | `tensorflow/c/` |
| generated | 2026-05-17T17:00 |
| priority | HIGH |
| confidence | [HIGH] |

## Structure

```
c_api.h/c_api.cc → Stable C bindings wrapping core TF (Session, Graph, Tensor)
tf_tensor.h       → TF_Tensor: C-compatible tensor type with deallocator
c_api_internal.h  → Internal helpers (not part of public ABI)
```

## Interfaces

| Interface | Role | Key Files |
|---|---|---|
| TF_Session / TF_Graph / TF_Tensor | C-compatible wrappers around C++ Session/Graph/Tensor | `c_api.h` |
| TF_Status | Error status; set on failure, checked by caller | `c_api.h` |
| TF_Buffer | Opaque byte buffer for serialized protos | `c_api.h` |
| TF_OperationDescription | Builder pattern for adding ops to graphs | `c_api.h` |

## Patterns

- **Thin wrapping** — C functions call into C++ core, convert C types to C++ and back
- **Thread-safe** — TF_SessionRun is reentrant; caller manages TF_Tensor lifecycle
- **Error via TF_Status** — no exceptions; errors returned through status objects
- **Stable ABI** — struct sizes versioned; new fields added at end

## Concerns

- **ABI stability** — backward compatibility is the primary contract; deprecation is slow
- **Memory ownership** — TF_Tensor has caller-controlled deallocation function pointer
- **Opaque handles** — TF_Graph*, TF_Session* are opaque pointers; internal layout hidden

## Key Files

| File | Role | Onboarding |
|---|---|---|
| `c_api.h` | Public C API header | done |
| `c_api.cc` | C API implementation | done |
| `tf_tensor.h` | TF_Tensor type definition | done |

## Confidence

[HIGH] — All 3 files onboarded from initial bootstrap. Architecture verified from headers.

## Update History

- 2026-05-17: Initial area report from full-bootstrap Phase 2
