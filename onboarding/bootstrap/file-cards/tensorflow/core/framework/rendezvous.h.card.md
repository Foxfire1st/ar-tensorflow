# File Card — tensorflow/core/framework/rendezvous.h

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
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/core/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | core |

## Why This File Matters

Rendezvous is producer-consumer tensor passing. Worker must explain: Send/Recv semantics, local vs remote implementations, key format, timeout handling. Trap: tensors not consumed = memory leak; remote rendezvous has network failure modes.

## Known Traps

See area report ootstrap/areas/core.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled.
