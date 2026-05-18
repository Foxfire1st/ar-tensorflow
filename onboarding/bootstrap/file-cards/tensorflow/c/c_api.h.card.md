# File Card — tensorflow/c/c_api.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | entrypoint |
| risk | complex |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-005 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/c/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | c |

## Why This File Matters

Public C API: TF_Session, TF_Graph, TF_Tensor, TF_Status, TF_Buffer. Worker must explain: all function declarations, opaque handle types, error handling via TF_Status, ABI versioning. Trap: backward compatibility is a hard contract; deprecation requires version bumps.

## Known Traps

See area report ootstrap/areas/c.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
