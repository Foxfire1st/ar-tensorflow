# Overview Card — tensorflow/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/` |
| targetOverview | `overview.md` |
| parentOverview | (none — root) |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

Root repository overview. Explains what TensorFlow is, the architecture at a glance, functional areas, build system, key invariants, and glossary. Entry point for all downstream route-local overviews.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `tensorflow/core/` | C++ runtime | covered |
| `tensorflow/python/` | Python API | covered |
| `tensorflow/compiler/` | XLA compiler stack | covered |
| `tensorflow/lite/` | Mobile runtime | covered |
| `tensorflow/c/` | C API | covered |
| `tensorflow/cc/` | C++ API | deferred |
| `tensorflow/dtensor/` | Distributed tensor | deferred |
| `tensorflow/js/` | JS bridge | deferred |

## What This Overview Must Explain

- What TF is and what it's built from
- Architecture diagram showing all layers and data flow
- Code structure table with all areas and tech
- Functional area summaries (core, python, compiler, lite)
- Build system (Bazel, configure.py)
- Key invariants that span areas
- Glossary of architecture terms

## Inputs

| Input | Path | Required? |
|---|---|---|
| Scout report | `bootstrap/scout-report.md` | yes |
| Area briefs | `bootstrap/areas/*.brief.md` | yes |

## Required Links

### Backlinks

- (none — root overview)

### Downlinks

- `onboarding/tensorflow/core/overview.md`
- `onboarding/tensorflow/python/overview.md`
- `onboarding/tensorflow/compiler/overview.md`
- `onboarding/tensorflow/lite/overview.md`
- `onboarding/tensorflow/c/overview.md`

## Status

Already exists. Verified against scout report and area briefs on 2026-05-17. Missing commit hash; to be backfilled.
