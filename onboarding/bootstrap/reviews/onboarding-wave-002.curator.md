# Curator Review — onboarding-wave-002

| Field | Value |
|---|---|
| repo | tensorflow |
| reviewed | 2026-05-17T17:00 |
| waveManifest | `bootstrap/waves/onboarding-wave-002.md` |
| status | pass-with-notes |

## Summary

All 10 python/ file onboardings exist. Created from source reading of top-of-file sections. The python area has actual domain documentation (Python API docs, TF guides) — the docs evidence pack confirms this. The onboardings should be updated to incorporate these docs references where applicable.

## Files Reviewed (sampled)

| Onboarding File | Result | Notes |
|---|---|---|
| `__init__.py.md` | pass | Minimal bootstrap file correctly described |
| `tensor.py.md` | pass | Tensor/TensorSpec distinguished |
| `ops.py.md` | pass | Large file; Graph/Operation/gradient registration explained |
| `context.py.md` | pass | EagerContext singleton pattern noted |
| `def_function.py.md` | pass | Re-export shim correctly identified |
| `backprop.py.md` | pass | GradientTape semantics clear |
| `resource_variable_ops.py.md` | pass | ResourceVariable handle model noted |
| `training.py.md` | pass | fit/predict/evaluate pipeline clear |
| `framework_lib.py.md` | pass | Re-export hub correctly described |
| `standard_ops.py.md` | pass | Side-effect import pattern noted |

## Compliance Checklist

| Check | Result | Notes |
|---|---|---|
| File-level onboarding is strict 1-to-1 | pass | |
| File onboarding backlinks to nearest governing overview | pass | All link to python/overview.md |
| Docs References cite direct evidence | partial | Domain docs exist but were not fully incorporated during creation |
| Repo-Internal References use same-repo evidence only | pass | |
| Cross-Repo References prove real boundaries | pass | Correctly empty |
| No absolute filesystem paths | pass | |
| Update History is append-only | pass | |
| LOW-confidence claims are not stated as facts | pass | |

## Required Fixes

1. Backfill commit hash and date on all 10 file onboardings.
2. Incorporate docs evidence from `docs/python.docs-pack.md` — add canonical URLs for tf.Tensor, tf.GradientTape, tf.function, tf.Variable, Keras Model

## Next-Wave Recommendation

Proceed. Fix docs references during commit hash backfill pass.
