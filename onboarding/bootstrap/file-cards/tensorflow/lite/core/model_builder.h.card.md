# File Card — tensorflow/lite/core/model_builder.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | entrypoint |
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

FlatBufferModel: deserializes .tflite files. Worker must explain: BuildFromFile/Buffer, verification, FlatBufferModelBase inheritance. Trap: model must outlive Interpreter; zero-copy means buffer must stay alive.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
