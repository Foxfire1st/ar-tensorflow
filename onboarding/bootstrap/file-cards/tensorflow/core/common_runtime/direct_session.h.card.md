# File Card — tensorflow/core/common_runtime/direct_session.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | routine |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/core/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | core |

## Why This File Matters

DirectSession is the local single-machine Session implementation. Worker must explain: how DirectSession builds executor from graph, Run() pipeline (feeds→executor→fetches), device placement, function library integration. Trap: thread synchronization between concurrent Run() calls.

## Known Traps

See area report ootstrap/areas/core.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled.
