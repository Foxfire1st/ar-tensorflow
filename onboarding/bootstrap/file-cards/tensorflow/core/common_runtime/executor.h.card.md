# File Card — tensorflow/core/common_runtime/executor.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/common_runtime/executor.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/common_runtime/executor.h.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | core logic |
| risk | landmine |
| suggested action | update onboarding |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| localArea | core |

## Why This File Matters

Executor is the graph execution engine. Dispatches ready ops to kernels on devices, manages pending-op counts, handles control/data edges, and orchestrates concurrent execution through the executor thread pool.

## What The Worker Must Explain

- Purpose: graph dispatch loop — run ready ops, decrement pending counts, propagate
- Main logic: pending-count-based scheduling; RunAsync for non-blocking execution
- Non-obvious behavior: multiple executor implementations (direct, streaming); barrier nodes for synchronization
- Invariants: must respect data and control edge dependencies; no deadlocks
- Interfaces: Graph, Device, OpKernel, Rendezvous
- Update risks: executor is a hot path; scheduling changes affect performance of all models

## Known Traps

- Thread safety: ready-queue manipulation must be atomic
- Deadlocks possible if control edges are misconfigured
- Executor pool sizing affects throughput vs memory tradeoff

## Done When

- Onboarding updated. Commit hash backfilled.
