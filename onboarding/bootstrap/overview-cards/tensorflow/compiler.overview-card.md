# Overview Card — tensorflow/compiler/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/compiler/` |
| targetOverview | `onboarding/tensorflow/compiler/overview.md` |
| parentOverview | `overview.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

compiler/ is the XLA stack — critical for performance. Multiple compilation pathways (JIT, AOT, MLIR) add complexity. Boundary to `third_party/xla/` is essential context.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `jit/` | JIT clustering & compilation | 3 files onboarded |
| `tf2xla/` | TF→HLO lowering | 2 files onboarded |
| `aot/` | AOT compilation | 1 file onboarded |

## What This Overview Must Explain

- Three compilation pathways: JIT, AOT, MLIR
- Clustering pipeline: MarkForCompilationPass → EncapsulateSubgraphsPass → XlaCompiler
- XlaCompiler as central orchestrator
- DeviceCompiler caching
- GraphCompiler determinism
- Boundary to `third_party/xla/` (HLO IR, backends)

## Inputs

| Input | Path | Required? |
|---|---|---|
| Root overview | `overview.md` | yes |
| Area report | `bootstrap/areas/compiler.md` | yes |
| Area brief | `bootstrap/areas/compiler.brief.md` | yes |

## Required Links

### Backlinks

- `overview.md`

### Downlinks

- 6 file-level onboardings

## Status

Already exists. Verified against area report on 2026-05-17. Missing commit hash; to be backfilled.
