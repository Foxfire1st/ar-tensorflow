# xla_tpu_device.cc

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/jit/xla_tpu_device.cc` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-19T04:21:36+02:00 |
| lastVerifiedCommitHash | `12bd4f195c6504935d9dda753781ad9730ffbf71` |
| lastVerifiedCommitDate | 2026-05-18T01:15:20-07:00 |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Registers TPU XLA device behavior and policy flags. For the numeric-check hot
path, this file shows TPU XLA also enables `elide_assert_and_checknumerics`.

## Code Commentary

### Logic

- TPU registration chooses autoclustering policy based on local compile-on-demand
  conditions.
- Resource variables, tensor arrays, RNG, control triggers, variants, slow ops,
  and inaccurate ops are allowed according to the registration block.
- `elide_assert_and_checknumerics` is set true in the TPU registration.

### Conventions

- Use this file for TPU policy. CPU and GPU XLA device files are separate.
- TPU setup contains additional platform and device registration code outside
  this hot path.

### Invariants And Boundaries

- The policy affects XLA compilation behavior, not normal core runtime kernels.
- The reason for numeric/assert elision is defined by shared registry and
  compilability comments.

### Todos

No local TODOs in the TPU elision registration block.

## Docs References

No external documentation proves this internal registration flag. Use source
citations.

| Finding | Citations | Source Path |
|---|---|---|
| XLA TPU registration policy is TensorFlow compiler implementation detail. | — | — |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| XLA TPU registration enables `elide_assert_and_checknumerics`. | L355-L366 | `tensorflow/compiler/jit/xla_tpu_device.cc` |
| Registry comments explain that elision is used because XLA does not natively support Assert or CheckNumerics. | L168-L170 | `tensorflow/compiler/tf2xla/xla_op_registry.h` |

## Cross-Repo References

No sibling memory repo is configured.

## Update History

- 2026-05-19T04:21:36+02:00: Added file-level onboarding for TensorFlow XLA/check-numerics benchmark coverage.

