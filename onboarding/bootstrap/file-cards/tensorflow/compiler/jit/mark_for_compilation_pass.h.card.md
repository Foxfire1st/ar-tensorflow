# File Card — tensorflow/compiler/jit/mark_for_compilation_pass.h

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

Grappler pass: identifies compilable op clusters via allowlist. Worker must explain: Run() pipeline, CompilabilityChecker, _XlaCluster attribute, allowlist table. Trap: allowlist gaps force fallback; cluster connectivity constraints.

## Known Traps

See area report ootstrap/areas/compiler.md and boundary pack ootstrap/evidence/cross-repo/compiler.boundary-pack.md. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Cross-repo evidence from boundary pack incorporated.
