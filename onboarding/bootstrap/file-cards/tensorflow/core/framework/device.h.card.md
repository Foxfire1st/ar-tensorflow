# File Card — tensorflow/core/framework/device.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/framework/device.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/framework/device.h.md` |
| generated | 2026-05-17T17:00 |
| priority | high |
| role | boundary |
| risk | routine |
| suggested action | update onboarding |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| localArea | core |

## Why This File Matters

Device abstracts hardware: CPU, GPU, TPU. Owns allocators, thread pools, and resource managers. Placer assigns ops to devices; Executor dispatches kernels to devices.

## What The Worker Must Explain

- Purpose: hardware abstraction for CPU/GPU/TPU
- Main logic: name, device type, allocator, resource manager, executor
- Non-obvious behavior: per-device Eigen thread pool; GPU streams multiplexed
- Invariants: one Allocator per device; ResourceMgr owns per-device resources (variables, queues)

## Known Traps

- Device type string must match registration exactly
- GPU memory is per-device; cross-device tensors need copying

## Done When

- Onboarding updated. Commit hash backfilled.
