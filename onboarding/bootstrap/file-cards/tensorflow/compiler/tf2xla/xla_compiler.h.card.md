# File Card — tensorflow/compiler/tf2xla/xla_compiler.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | landmine |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-003 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/compiler/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | compiler |

## Why This File Matters

Central orchestrator: TF subgraph → XLA HLO compilation. Worker must explain: CompileFunction(), symbolic execution, XlaContext, resource management, XlaOpRegistry integration. Trap: requires static shapes; compilation failure modes are opaque. Boundary: xla::XlaBuilder from third_party/xla/.

## Known Traps

See area report ootstrap/areas/compiler.md and boundary pack ootstrap/evidence/cross-repo/compiler.boundary-pack.md. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Cross-repo evidence from boundary pack incorporated.
