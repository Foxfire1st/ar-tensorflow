# File Card — tensorflow/compiler/jit/xla_device.h

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
| Suggested wave | onboarding-wave-003 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/compiler/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | compiler |

## Why This File Matters

XlaDevice: executes TF ops via XLA compilation. Worker must explain: LocalDevice subclass, Compute() dispatch through XLA, XlaTensor wrapping ScopedShapedBuffer. Trap: all ops on XlaDevice go through XLA; no TF executor fallback.

## Known Traps

See area report ootstrap/areas/compiler.md and boundary pack ootstrap/evidence/cross-repo/compiler.boundary-pack.md. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Cross-repo evidence from boundary pack incorporated.
