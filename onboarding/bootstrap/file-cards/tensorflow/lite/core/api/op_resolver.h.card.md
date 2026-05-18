# File Card — tensorflow/lite/core/api/op_resolver.h

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
| Suggested wave | onboarding-wave-004 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/lite/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | lite |

## Why This File Matters

OpResolver: maps op codes→TfLiteRegistration. Worker must explain: FindOp(), AddBuiltin/AddCustom, version ranges, chained resolvers. Trap: resolution failure = crash; custom ops need careful version management.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
