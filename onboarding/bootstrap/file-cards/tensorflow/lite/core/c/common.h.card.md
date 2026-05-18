# File Card — tensorflow/lite/core/c/common.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | boundary |
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

Stable C ABI: TfLiteTensor, TfLiteContext, TfLiteRegistration, TfLiteDelegate. Worker must explain: all struct layouts, function pointer contracts, allocation types, quantization params. Trap: ABI versioning; struct sizes must remain compatible.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
