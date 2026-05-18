# File Card — tensorflow/core/common_runtime/placer.h

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
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/core/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | core |

## Why This File Matters

Placer assigns ops to devices using cost model and constraints. Worker must explain: placement algorithm (colocation groups, device constraints, cost estimation), Placer::Run() contract, supported device types. Trap: Placer can fail silently; wrong placement causes cross-device copies.

## Known Traps

See area report ootstrap/areas/core.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled.
