# File Card — tensorflow/core/framework/op_kernel.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/framework/op_kernel.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/framework/op_kernel.h.md` |
| generated | 2026-05-17T17:00 |

## Classification

| Field | Value |
|---|---|
| Priority | high |
| Role | boundary |
| Risk | complex |
| Suggested action | update onboarding |
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| ancestorOverviews | `onboarding/overview.md` |
| localArea | core |

## Why This File Matters

Defines OpKernel — the base class for all 1000+ TF op implementations. The `Compute(OpKernelContext*)` method is the canonical execution contract. Every op (MatMul, Conv2D, etc.) inherits from this.

## What The Worker Must Explain

- Purpose: op execution contract and lifecycle
- Main logic: OpKernel::Compute(OpKernelContext*) — input/output tensor access, status reporting
- Non-obvious behavior: AsyncOpKernel for non-blocking ops; OpKernelContext manages tensor allocation and error state
- Invariants: Compute() must be thread-safe; output allocation happens via context->allocate_output()
- Interfaces: OpKernelConstruction (init params), OpKernelContext (runtime state), Device
- Failure modes: OOM during output allocation; shape mismatch between declared and actual
- Update risks: kernel registration macros must stay synchronized

## Inputs For Worker

| Input | Path | Required? | Why |
|---|---|---|---|
| Source file | `tensorflow/core/framework/op_kernel.h` | yes | concrete behavior |
| Nearest governing overview | `onboarding/tensorflow/core/overview.md` | yes | local area model |
| Existing onboarding | `onboarding/tensorflow/core/framework/op_kernel.h.md` | yes | preserve/update |
| Area report | `bootstrap/areas/core.md` | yes | local context |

## Known Traps

- Thread safety: Compute() can be called concurrently from multiple threads in the same executor
- Output tensor ownership: context->allocate_output() transfers ownership; caller must not free

## Done When

- Onboarding updated. Commit hash backfilled.
