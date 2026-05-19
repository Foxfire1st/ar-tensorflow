# compilability_check_util.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/jit/compilability_check_util.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

JIT compilability policy helper declarations. For `CheckNumerics`, this header
records the distinction between implicit auto-clustering and explicit XLA: the
utility avoids surprising users by not auto-clustering checks unless the policy
allows elision, but explicit XLA may use dummy implementations.

## Code Commentary

### Logic

- `OpFilter` contains flags for stateful RNG, control triggers, and whether
  eliding Assert/CheckNumerics is allowed.
- The comment explains that Assert and CheckNumerics have dummy XLA
  implementations and should not be implicitly removed by auto-clustering.
- `IsAssertOrCheckNumerics` recognizes the exact op names `Assert` and
  `CheckNumerics`.

### Conventions

- Use this file when a question is about whether an op is considered compilable
  by JIT policy.
- Use XLA device registration files to see what concrete devices allow.

### Invariants And Boundaries

- This file declares policy structures and helper checks; implementation details
  may live in the corresponding `.cc` files.
- The policy is about JIT clustering, not Python dispatch or core kernels.

### Todos

The local comments include a TODO for `ControlTrigger`, unrelated to numeric
checks.

## Docs References

No external documentation proves this internal JIT policy. Use source citations.

| Finding | Citations | Source Path |
|---|---|---|
| JIT elision policy for `CheckNumerics` is internal TensorFlow compiler behavior. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The policy flag allows eliding Assert and CheckNumerics, but comments say auto-clustering avoids surprising users unless explicit XLA is requested. | L108-L115 | `tensorflow/compiler/jit/compilability_check_util.h` |
| The helper recognizes exactly `Assert` and `CheckNumerics`. | L248-L250 | `tensorflow/compiler/jit/compilability_check_util.h` |
| Device registrations enable the registry-side elision flag for XLA CPU/GPU/TPU. | L70-L83; L88-L100; L355-L366 | `tensorflow/compiler/jit/xla_cpu_device.cc`, `tensorflow/compiler/jit/xla_gpu_device.cc`, `tensorflow/compiler/jit/xla_tpu_device.cc` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

