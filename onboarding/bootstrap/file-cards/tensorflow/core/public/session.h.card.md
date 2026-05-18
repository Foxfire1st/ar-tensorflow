# File Card — tensorflow/core/public/session.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/public/session.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/public/session.h.md` |
| generated | 2026-05-17T17:00 |

## Classification

| Field | Value |
|---|---|
| Priority | high |
| Role | entrypoint |
| Risk | routine |
| Suggested action | update onboarding |
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| ancestorOverviews | `onboarding/overview.md` |
| localArea | core |

## Why This File Matters

Session is the primary user-facing C++ API: Create/Extend/Run/Close. All execution (Python, C, C++ bindings) eventually calls Session::Run().

## What The Worker Must Explain

- Purpose: user-facing graph execution API
- Main logic: Create(graph) → Extend(graph) → Run(feeds,fetches,targets) → Close
- Non-obvious behavior: Run() blocks until all fetches complete; multi-threaded internally
- Invariants: Session owns graph state; Close() releases all resources
- Interfaces: GraphDef, Tensor, RunOptions/RunMetadata
- Failure modes: invalid graph structure; device not found; OOM

## Inputs For Worker

| Input | Path | Required? | Why |
|---|---|---|---|
| Source file | `tensorflow/core/public/session.h` | yes | concrete behavior |
| Nearest governing overview | `onboarding/tensorflow/core/overview.md` | yes | local area model |
| Existing onboarding | `onboarding/tensorflow/core/public/session.h.md` | yes | preserve/update |
| Area report | `bootstrap/areas/core.md` | yes | local context |

## Known Traps

- Session must be Closed; leaking sessions holds GPU memory
- Extend() adds ops incrementally; order matters for graph construction

## Done When

- Onboarding updated. Commit hash backfilled.
