# File Card — tensorflow/lite/core/interpreter_builder.h

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
| Suggested wave | onboarding-wave-004 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/lite/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | lite |

## Why This File Matters

Factory: FlatBufferModel + OpResolver → Interpreter. Worker must explain: op resolution, tensor wiring, execution plan building. Trap: missing ops fail at build time; resolution order matters.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
