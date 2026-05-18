# Overview Card — tensorflow/lite/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/lite/` |
| targetOverview | `onboarding/tensorflow/lite/overview.md` |
| parentOverview | `overview.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

lite/ is the mobile/embedded runtime — independent from core TF. FlatBuffer-based, different execution model, separate op ecosystem. Delegates for hardware acceleration add significant architectural complexity.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `core/` | Interpreter, Subgraph, ModelBuilder, Builder, OpResolver | 5 files onboarded |
| `c/` | C ABI types | 1 file onboarded |
| `core/api/` | OpResolver interface | 1 file onboarded |
| `schema/` | FlatBuffer schema | 1 file onboarded |

## What This Overview Must Explain

- FlatBuffer model loading vs Protobuf
- Interpreter→Subgraph execution model
- OpResolver and op registration
- Static memory planning (SimpleMemoryArena)
- Delegate plugin architecture
- Separation from micro/ (embedded)

## Inputs

| Input | Path | Required? |
|---|---|---|
| Root overview | `overview.md` | yes |
| Area report | `bootstrap/areas/lite.md` | yes |
| Area brief | `bootstrap/areas/lite.brief.md` | yes |

## Required Links

### Backlinks

- `overview.md`

### Downlinks

- 8 file-level onboardings

## Status

Already exists. Verified against area report on 2026-05-17. Missing commit hash; to be backfilled.
