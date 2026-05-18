# File Card — tensorflow/core/framework/device_base.h

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

Base Device class with DeviceContext and field types. Worker must explain: DeviceBase as the minimal device interface, DeviceContext for per-step state, DeviceAttributes for registration. Trap: DeviceContext lifecycle tied to step; leaking contexts = memory leak.

## Known Traps

See area report ootstrap/areas/core.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled.
