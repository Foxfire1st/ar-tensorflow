# Overview Card — tensorflow/core/

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceRoute | `tensorflow/core/` |
| targetOverview | `onboarding/tensorflow/core/overview.md` |
| parentOverview | `overview.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| confidence | [HIGH] |

## Why This Overview Exists

core/ is the C++ runtime foundation. Every other layer depends on it. Understanding the Graph → Executor → OpKernel pipeline is essential for any TF development.

## Governs

| Path | Role | Coverage Status |
|---|---|---|
| `framework/` | Tensor, OpKernel, Device, Op, Function | 6 files onboarded |
| `graph/` | Graph DAG, Node, Edge | 1 file onboarded |
| `common_runtime/` | Executor, DirectSession, Placer, Rendezvous | 4 files onboarded |
| `public/` | Session API | 1 file onboarded |

## What This Overview Must Explain

- The execution pipeline: Session → Executor → OpKernel::Compute → Device
- Key abstractions and how they connect
- Internal boundaries (public → framework → graph → common_runtime)
- Cross-cutting concerns: device placement, rendezvous, memory management
- Load-bearing file inventory
- Traps: thread safety, BFCAllocator, shape inference, backward compatibility

## Inputs

| Input | Path | Required? |
|---|---|---|
| Root overview | `overview.md` | yes |
| Area report | `bootstrap/areas/core.md` | yes |
| Area brief | `bootstrap/areas/core.brief.md` | yes |

## Required Links

### Backlinks

- `overview.md`

### Downlinks

- 12 file-level onboardings (graph.h, op_kernel.h, session.h, tensor.h, device.h, executor.h, op.h, direct_session.h, function.h, device_base.h, placer.h, rendezvous.h)

## Status

Already exists. Verified against area report on 2026-05-17. Missing commit hash; to be backfilled.
