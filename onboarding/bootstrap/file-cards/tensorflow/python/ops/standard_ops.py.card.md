# File Card — tensorflow/python/ops/standard_ops.py

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | utility |
| risk | routine |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-002 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/python/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | python |

## Why This File Matters

Bootstrap import that triggers gradient registration via side-effect imports. Worker must explain: imports gen_* modules to register gradients; no own definitions. Trap: side-effect imports; import order matters for gradient availability.

## Known Traps

See area report ootstrap/areas/python.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Docs evidence from ootstrap/evidence/docs/python.docs-pack.md incorporated if relevant.
