# xla_op_registry.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/tf2xla/xla_op_registry.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Registry types and options for TensorFlow ops that can compile through XLA. For
the check-numerics hot path, the key field is
`elide_assert_and_checknumerics`, which records whether Assert and CheckNumerics
may be clustered by eliding them because XLA has no native support.

## Code Commentary

### Logic

- Compilation device registrations fill an `XlaOpRegistry::DeviceRegistration`.
- The registration contains clustering policy flags for resources, control
  triggers, RNG, variants, slow ops, and numeric/assert elision.
- `elide_assert_and_checknumerics` defaults false in the struct and is enabled by
  XLA CPU/GPU/TPU device registration files.

### Conventions

- Use this file for the registry contract; use the device registration files for
  per-device policy values.
- The wording "eliding them" is the source-level explanation for why explicit
  XLA may not perform runtime numeric checks.

### Invariants And Boundaries

- The registry option is about clustering/compilation policy, not the Python API
  or CPU/GPU runtime kernels.
- Default false means device registrations must opt into the behavior.

### Todos

No local TODOs in the `elide_assert_and_checknumerics` field block.

## Docs References

No external documentation proves this internal registry option. Use source
citations for behavior.

| Finding | Citations | Source Path |
|---|---|---|
| `elide_assert_and_checknumerics` is an internal XLA registry policy. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The registry option says Assert and CheckNumerics may be clustered by eliding them because XLA does not natively support them. | L168-L170 | `tensorflow/compiler/tf2xla/xla_op_registry.h` |
| XLA CPU registration enables `elide_assert_and_checknumerics`. | L70-L83 | `tensorflow/compiler/jit/xla_cpu_device.cc` |
| XLA GPU registration enables `elide_assert_and_checknumerics`. | L88-L100 | `tensorflow/compiler/jit/xla_gpu_device.cc` |
| XLA TPU registration enables `elide_assert_and_checknumerics`. | L355-L366 | `tensorflow/compiler/jit/xla_tpu_device.cc` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

