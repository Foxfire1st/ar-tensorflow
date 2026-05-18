# Curator Review — overview-wave-001

| Field | Value |
|---|---|
| repo | tensorflow |
| reviewed | 2026-05-17T17:00 |
| waveManifest | `bootstrap/waves/overview-wave-001.md` |
| status | pass-with-notes |

## Summary

All 5 route-local overviews exist and cover their areas with documented structure, interfaces, load-bearing files, and traps. The root overview provides a valid architecture diagram. All overviews were created before evidence packs and area reports — the backfill now provides the missing inputs. Overviews should be re-verified against area reports for consistency.

## Files Reviewed

| Onboarding File | Source Route | Result |
|---|---|---|
| `overview.md` | `tensorflow/` | pass |
| `onboarding/tensorflow/core/overview.md` | `tensorflow/core/` | pass |
| `onboarding/tensorflow/python/overview.md` | `tensorflow/python/` | pass |
| `onboarding/tensorflow/compiler/overview.md` | `tensorflow/compiler/` | pass |
| `onboarding/tensorflow/lite/overview.md` | `tensorflow/lite/` | pass |
| `onboarding/tensorflow/c/overview.md` | `tensorflow/c/` | pass |

## Compliance Checklist

| Check | Result | Notes |
|---|---|---|
| Durable overview placement is route-local and mirrored | pass | All in correct mirrored paths |
| File-level onboarding is strict 1-to-1 | n/a | n/a for overview wave |
| File onboarding backlinks to nearest governing overview | n/a | n/a |
| Overview downlinks list governed files | pass | File-level onboarding tables present |
| Durable onboarding contains no task-local planning | pass | Clean |
| Docs References cite direct evidence | pass | Canonical URLs used |
| Repo-Internal References use same-repo evidence only | pass | Correct |
| Cross-Repo References prove real boundaries | pass | Boundary packs document vendor dependencies |
| No absolute filesystem paths | pass | Workspace-relative paths used |
| Update History is append-only | pass | Clean |
| LOW-confidence claims are not stated as facts | pass | No LOW claims in durable memory |
| Deferred files are recorded | pass | cc/, dtensor/, js/ deferred |
| STATE.md updated | pending | Will update after backfill complete |

## Reference Health

| File | Section | Issue | Required Fix |
|---|---|---|---|
| All overviews | lastVerifiedCommitHash | blank | Backfill commit `2020b5919c` |
| All overviews | lastVerifiedCommitDate | blank | Backfill `2026-05-16` |

## Required Fixes

1. Backfill `lastVerifiedCommitHash: 2020b5919c5b66b8672438bed85d0ca88d434438` and `lastVerifiedCommitDate: 2026-05-16` on all 6 overviews.

## Next-Wave Recommendation

Proceed. Re-verify overview content against area reports once commit hashes are anchored.
