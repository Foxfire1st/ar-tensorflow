# Overview Card — tensorflow/python/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/python/` |
| targetOverview | `onboarding/tensorflow/python/overview.md` |
| parentOverview | `overview.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

python/ is the primary user-facing API. All developers interact with TF through this layer. Understanding eager/graph duality, tf.function tracing, GradientTape, and Keras is essential.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `framework/` | Tensor, Graph, Operation | 3 files onboarded |
| `eager/` | Context, GradientTape, tf.function | 3 files onboarded |
| `ops/` | ResourceVariable, standard_ops | 2 files onboarded |
| `keras/engine/` | Keras Model training API | 1 file onboarded |
| `__init__.py` | Package bootstrap | 1 file onboarded |

## What This Overview Must Explain

- Eager-first execution model vs graph mode
- tf.function tracing and autograph
- GradientTape and automatic differentiation
- Keras as the high-level modeling API
- Pywrap bridge to C++ core
- Internal boundaries: framework → eager → ops → keras → distribute

## Inputs

| Input | Path | Required? |
|---|---|---|
| Root overview | `overview.md` | yes |
| Area report | `bootstrap/areas/python.md` | yes |
| Area brief | `bootstrap/areas/python.brief.md` | yes |

## Required Links

### Backlinks

- `overview.md`

### Downlinks

- 10 file-level onboardings

## Status

Already exists. Verified against area report on 2026-05-17. Missing commit hash; to be backfilled.
