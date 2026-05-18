# File Card — tensorflow/compiler/tf2xla/graph_compiler.h

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
| Suggested wave | onboarding-wave-003 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/compiler/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | compiler |

## Why This File Matters

Deterministic graph→HLO traversal for reproducible AOT. Worker must explain: topological walk, total input ordering, function call handling, XlaCompilationDevice. Trap: determinism is correctness-critical for AOT. Known TODOs: remove XlaCompilationDevice hack.

## Known Traps

See area report ootstrap/areas/compiler.md and boundary pack ootstrap/evidence/cross-repo/compiler.boundary-pack.md. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Cross-repo evidence from boundary pack incorporated.
