# File Card — tensorflow/lite/simple_memory_arena.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | $src |
| targetOnboardingFile | $tgt |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | routine |
| suggested action | update onboarding |
| Suggested wave | onboarding-wave-004 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/lite/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | lite |

## Why This File Matters

Dynamic tensor memory allocator: greedy interval allocation. Worker must explain: Allocate() with GraphInfo, ResolveAllocations(), memory reuse strategy. Trap: fixed arena size; no dynamic growth during inference.

## Known Traps

See area report ootstrap/areas/lite.md. See existing onboarding.

## Done When

- Onboarding updated. Commit hash backfilled.
