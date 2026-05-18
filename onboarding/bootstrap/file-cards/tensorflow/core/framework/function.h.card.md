# File Card — tensorflow/core/framework/function.h

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
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/core/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | core |

## Why This File Matters

Function definition, library, and runtime. Worker must explain: FunctionDef proto, FunctionBody (graph+args/retvals), FunctionLibraryRuntime (instantiation+optimization), gradient functions. Trap: function inlining vs calling semantics; optimization passes may change behavior.

## Known Traps

See area report ootstrap/areas/core.md for cross-cutting concerns. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled.
