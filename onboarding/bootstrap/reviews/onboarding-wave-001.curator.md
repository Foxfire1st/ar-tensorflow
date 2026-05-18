# Curator Review — onboarding-wave-001

| Field | Value |
|---|---|
| repo | tensorflow |
| reviewed | 2026-05-17T17:00 |
| waveManifest | `bootstrap/waves/onboarding-wave-001.md` |
| status | pass-with-notes |

## Summary

All 12 core/ file onboardings exist and follow C-05 format. Created before evidence packs and file cards existed. The onboardings correctly identify key structures, interfaces, and invariants. Repo-internal references cross-link to other core onboardings. Cross-repo and docs references are correctly empty (C++ internals have no domain docs or cross-repo boundaries).

## Files Reviewed (sampled)

| Onboarding File | Result | Notes |
|---|---|---|
| `graph.h.md` | pass | Good DAG explanation; note control vs data edges |
| `op_kernel.h.md` | pass | Compute() contract well explained |
| `session.h.md` | pass | Session lifecycle clear |
| `tensor.h.md` | pass | Ref-counted buffer pattern noted |
| `executor.h.md` | pass | Pending-count scheduling noted |
| `direct_session.h.md` | pass | Local execution path clear |
| `op.h.md` | pass | REGISTER_OP macro documented |
| `function.h.md` | pass | FunctionLibraryRuntime noted |
| `device_base.h.md` | pass | DeviceContext lifecycle noted |
| `placer.h.md` | pass | Placement algorithm noted |
| `rendezvous.h.md` | pass | Send/Recv semantics clear |
| `device.h.md` | pass | Allocator ownership noted |

## Compliance Checklist

| Check | Result | Notes |
|---|---|---|
| File-level onboarding is strict 1-to-1 | pass | One onboarding per source file |
| File onboarding backlinks to nearest governing overview | pass | All link to core/overview.md |
| Durable onboarding contains no task-local planning | pass | Clean |
| Docs References cite direct evidence | pass | All correctly state no relevant docs found |
| Repo-Internal References use same-repo evidence only | pass | Cross-links verified |
| Cross-Repo References prove real boundaries | pass | Correctly empty |
| No absolute filesystem paths | pass | Workspace-relative |
| Update History is append-only | pass | Single initial entry |
| LOW-confidence claims are not stated as facts | pass | No LOW claims |
| STATE.md updated | pending | After backfill complete |

## Required Fixes

1. Backfill commit hash and date on all 12 file onboardings.

## Next-Wave Recommendation

Proceed. No quality issues detected beyond missing commit hashes.
