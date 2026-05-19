# xla_gpu_device.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/jit/xla_gpu_device.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Registers the XLA GPU compilation device and its JIT policy flags. For the
numeric-check hot path, it opts GPU XLA into `elide_assert_and_checknumerics`.

## Code Commentary

### Logic

- GPU XLA registration sets the compilation device name to `DEVICE_GPU_XLA_JIT`.
- Autoclustering policy is `kAlways`.
- `elide_assert_and_checknumerics` is set true before registering
  `DEVICE_XLA_GPU`.

### Conventions

- Use this file for GPU XLA device policy.
- Compare with CPU and TPU files before claiming a behavior is device-independent.

### Invariants And Boundaries

- The field enables XLA registry policy; it does not alter normal GPU runtime
  kernels in `tensorflow/core/kernels/check_numerics_op.cc`.

### Todos

No local TODOs in the XLA GPU registration block.

## Docs References

No external documentation proves this internal registration flag. Use source
citations.

| Finding | Citations | Source Path |
|---|---|---|
| XLA GPU registration policy is TensorFlow compiler implementation detail. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XLA GPU registration enables `elide_assert_and_checknumerics` before registering `DEVICE_XLA_GPU`. | L88-L100 | `tensorflow/compiler/jit/xla_gpu_device.cc` |
| Registry comments explain that elision is used because XLA does not natively support Assert or CheckNumerics. | L168-L170 | `tensorflow/compiler/tf2xla/xla_op_registry.h` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

