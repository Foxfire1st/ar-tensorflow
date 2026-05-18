# File Card — tensorflow/python/framework/tensor.py

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

tf.Tensor, TensorSpec, BoundedTensorSpec. Worker must explain: eager vs graph tensors, shape/dtype/device, numpy() bridge, TensorSpec for shape constraints. Trap: Tensor is a thin handle; backing memory may be on GPU.

## Known Traps

See area report ootstrap/areas/python.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Docs evidence from ootstrap/evidence/docs/python.docs-pack.md incorporated if relevant.
