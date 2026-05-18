# Curator Review — onboarding-wave-003

| Field | Value |
|---|---|
| repo | tensorflow |
| reviewed | 2026-05-17T17:00 |
| waveManifest | `bootstrap/waves/onboarding-wave-003.md` |
| status | pass-with-notes |

## Summary

All 6 compiler/ file onboardings exist. The compiler area has the most significant vendor boundary (XLA in third_party/xla/) and domain documentation (openxla.org). The cross-repo boundary pack identifies specific interfaces (XlaBuilder, XlaComputation, LocalClient) that the onboardings should reference.

## Files Reviewed

| Onboarding File | Result | Notes |
|---|---|---|
| `xla_compiler.h.md` | pass | Central orchestrator role clear; XlaBuilder boundary noted |
| `device_compiler.h.md` | pass | Caching layer explained |
| `mark_for_compilation_pass.h.md` | pass | Clustering + _XlaCluster attribute noted |
| `xla_device.h.md` | pass | Compute() routing through XLA clear |
| `graph_compiler.h.md` | pass | Determinism requirement and TODOs noted |
| `compile.h.md` | pass | AOT pipeline and static shape requirement clear |

## Compliance Checklist

| Check | Result | Notes |
|---|---|---|
| File-level onboarding is strict 1-to-1 | pass | |
| File onboarding backlinks to nearest governing overview | pass | All link to compiler/overview.md |
| Docs References cite direct evidence | partial | XLA architecture docs (openxla.org) not yet incorporated |
| Repo-Internal References use same-repo evidence only | pass | Cross-links verified |
| Cross-Repo References prove real boundaries | partial | Boundary pack identifies specific XLA interfaces; onboardings say "no relevant cross-repo evidence" — should reference vendor boundary |
| Update History is append-only | pass | |
| LOW-confidence claims are not stated as facts | pass | |

## Required Fixes

1. Backfill commit hash and date on all 6 file onboardings.
2. Update Cross-Repo References to document XLA vendor boundary interfaces (XlaBuilder, XlaComputation, LocalClient, CpuAotCompilationResult).
3. Add openxla.org architecture link to Docs References.

## Next-Wave Recommendation

Proceed. Fix cross-repo references during backfill pass.
