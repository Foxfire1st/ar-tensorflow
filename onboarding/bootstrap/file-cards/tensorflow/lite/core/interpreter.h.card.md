# File Card — tensorflow/lite/core/interpreter.h

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
| Suggested wave | onboarding-wave-004 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/lite/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | lite |

## Why This File Matters

Top-level TFLite execution engine. Worker must explain: model loading, AllocateTensors(), Invoke(), delegates, signature runners, cancellation. Trap: thread safety (concurrent Invoke); delegate must be applied before AllocateTensors.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
