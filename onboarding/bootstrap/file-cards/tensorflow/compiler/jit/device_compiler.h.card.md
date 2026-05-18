# File Card — tensorflow/compiler/jit/device_compiler.h

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
| Suggested wave | onboarding-wave-003 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | onboarding/tensorflow/compiler/overview.md |
| ancestorOverviews | onboarding/overview.md |
| localArea | compiler |

## Why This File Matters

DeviceCompiler: caching layer wrapping XlaCompiler with persistence and profiling. Worker must explain: cluster fingerprint→executable cache, CompileIfNeeded(), persistence format, profiling hooks. Trap: cache invalidation on code changes; stale executables produce wrong results.

## Known Traps

See area report ootstrap/areas/compiler.md and boundary pack ootstrap/evidence/cross-repo/compiler.boundary-pack.md. See existing onboarding for file-specific traps.

## Done When

- Onboarding updated. Commit hash backfilled. Cross-repo evidence from boundary pack incorporated.
