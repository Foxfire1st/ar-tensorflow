# Overview Wave 001 — Route-Local Overviews

| Field | Value |
|---|---|
| repo | tensorflow |
| generated | 2026-05-17T17:00 |
| waveType | route-overview |
| mode | full-bootstrap |
| status | complete |

## Goal

Create route-local overviews for 5 priority areas: core, python, compiler, lite, c.

## Included Cards

| Priority | Card | Target | Reason |
|---|---|---|---|
| high | `overview-cards/tensorflow.overview-card.md` | `overview.md` | root overview (already exists) |
| high | `overview-cards/tensorflow/core.overview-card.md` | `onboarding/tensorflow/core/overview.md` | foundation of all execution |
| high | `overview-cards/tensorflow/python.overview-card.md` | `onboarding/tensorflow/python/overview.md` | primary user API |
| high | `overview-cards/tensorflow/compiler.overview-card.md` | `onboarding/tensorflow/compiler/overview.md` | performance-critical complexity |
| high | `overview-cards/tensorflow/lite.overview-card.md` | `onboarding/tensorflow/lite/overview.md` | independent mobile runtime |
| high | `overview-cards/tensorflow/c.overview-card.md` | `onboarding/tensorflow/c/overview.md` | stable ABI boundary |

## Evidence Required

| Evidence Pack | Applies To | Required? |
|---|---|---|
| `docs/core.docs-pack.md` | core overview | yes (no relevant evidence found) |
| `docs/python.docs-pack.md` | python overview | yes |
| `docs/compiler.docs-pack.md` | compiler overview | yes |
| `docs/lite.docs-pack.md` | lite overview | yes |
| `docs/c.docs-pack.md` | c overview | yes |

## Done When

- All 5 route-local overviews exist.
- All commit hashes backfilled.
