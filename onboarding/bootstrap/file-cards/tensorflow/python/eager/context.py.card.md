# File Card — tensorflow/python/eager/context.py

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | complex |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-002 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/python/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | python |

## Why This File Matters

EagerContext singleton: execution mode, device management, config. Worker must explain: context() singleton, executing_eagerly(), device placement, async execution, config_proto. Trap: context is global; tests must reset between cases.

## Known Traps

See area report ootstrap/areas/python.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Docs evidence from ootstrap/evidence/docs/python.docs-pack.md incorporated if relevant.
