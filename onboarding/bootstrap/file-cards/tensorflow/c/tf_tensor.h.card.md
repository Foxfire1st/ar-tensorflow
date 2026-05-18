# File Card — tensorflow/c/tf_tensor.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | boundary |
| risk | routine |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-005 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/c/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | c |

## Why This File Matters

TF_Tensor: C-compatible tensor type with deallocator function pointer. Worker must explain: data union, shape/dtype, deallocation contract. Trap: deallocator must match allocation method; wrong deallocator = double-free.

## Known Traps

See area report ootstrap/areas/c.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
