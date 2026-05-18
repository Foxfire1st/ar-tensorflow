# File Card — tensorflow/python/ops/resource_variable_ops.py

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

tf.Variable (ResourceVariable): mutable state container. Worker must explain: ResourceVariable lifecycle, handle-based backend, assign/assign_add/read_value, distributed variable. Trap: variable handle is ref-counted; eager vs graph lifecycle differs.

## Known Traps

See area report ootstrap/areas/python.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Docs evidence from ootstrap/evidence/docs/python.docs-pack.md incorporated if relevant.
