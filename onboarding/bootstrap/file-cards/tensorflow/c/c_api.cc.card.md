# File Card — tensorflow/c/c_api.cc

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
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

C API implementation: thin wrappers around C++ Session/Graph/Tensor. Worker must explain: C→C++ type conversion, session lifecycle, graph building, tensor marshalling. Trap: C memory management vs C++ RAII; ownership transfer points.

## Known Traps

See area report ootstrap/areas/c.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
