# Overview Card — tensorflow/c/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/c/` |
| targetOverview | `onboarding/tensorflow/c/overview.md` |
| parentOverview | `overview.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

c/ is the stable ABI boundary. All non-Python language bindings (C++, Go, Java, JS, Swift) go through it. ABI stability is a hard contract.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `c_api.h` | Public C API declarations | 1 file onboarded |
| `c_api.cc` | C API implementation | 1 file onboarded |
| `tf_tensor.h` | TF_Tensor type | 1 file onboarded |

## What This Overview Must Explain

- C wrapping pattern (thin wrappers around C++ core)
- TF_Session, TF_Graph, TF_Tensor, TF_Status, TF_Buffer
- Error handling via TF_Status
- Memory ownership (caller-controlled deallocators)
- ABI versioning contract
- Opaque handle pattern

## Inputs

| Input | Path | Required? |
|---|---|---|
| Root overview | `overview.md` | yes |
| Area report | `bootstrap/areas/c.md` | yes |
| Area brief | `bootstrap/areas/c.brief.md` | yes |

## Required Links

### Backlinks

- `overview.md`

### Downlinks

- 3 file-level onboardings

## Status

Already exists. Verified against area report on 2026-05-17. Missing commit hash; to be backfilled.
