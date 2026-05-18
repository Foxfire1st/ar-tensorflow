# legalize_tf_patterns.td

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:02 |
| lastVerifiedCommitHash | `573bbe2b4163fb4349c8c662635f99cca40c8406` |
| lastVerifiedCommitDate | 2025-12-22T09:54:11-08:00 |
| governingOverview | [`tensorflow/compiler/mlir/tf2xla/transforms/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/mlir/tf2xla/transforms/overview.md`](overview.md)

## Purpose

Declarative rewrite pattern source for the TF2XLA MLIR legalization path. This
TableGen file maps TensorFlow dialect operations to StableHLO, CHLO, and related
HLO-family operations, while delegating nontrivial calculations to C++ helper
functions through `NativeCodeCall`. Bazel generates `generated_legalize_tf.inc`
from this file and `legalize_tf.cc` compiles the generated patterns into the
`legalize_tf` library.

## Code Commentary

### Logic

- Includes MLIR core dialect TableGen files, StableHLO/CHLO ops, TensorFlow op
  definitions, and XLA HLO op definitions before declaring local constraints and
  helper calls.
- Defines tensor type constraints such as signed/unsigned integer tensors and
  IEEE floating tensors.
- Uses small reusable TableGen classes for comparison direction attributes,
  direct binary rewrites, logical/bitwise rewrites, and compare rewrites.
- Handles TensorFlow-vs-HLO semantic mismatches with helper calls and expanded
  patterns, such as axis conversion, integer floor division correction, padding
  conversion, and precision configuration.
- Some TensorFlow ops are lowered to their input value or removed when HLO/XLA
  has no corresponding runtime construct.

### Conventions

- Section comments divide patterns by operation family.
- `TF_*Op` names are TensorFlow dialect sources; `StableHLO_*Op` and
  `CHLO_*Op` names are target operations.
- `NativeCodeCall` entries must resolve to helper functions visible from
  `legalize_tf.cc`.
- `replaceWithValue` means the original operation is erased and its operand is
  forwarded.

### Invariants And Boundaries

- Keep this file aligned with `legalize_tf.cc` helper symbols and BUILD deps.
- Pattern ordering matters when multiple patterns could match related op shapes.
- Do not add target ops without verifying the relevant target dialect and Bazel
  dependency are available.
- Comments about unsupported HLO/XLA semantics are part of the compatibility
  contract and should not be removed casually.

### Todos

- TODO markers record unsupported or incomplete legalization behavior, including
  assertions and CheckNumerics support in HLO.

### Docs References

MLIR DRR documentation is the relevant external model: this file is source code
for generated rewrite patterns, not just metadata.

| Finding | Citations | Source Path |
|---|---|---|
| MLIR Declarative Rewrite Rules describe TableGen rewrite rules expanded into rewrite pattern code. | — | [MLIR Declarative Rewrite Rules](https://mlir.llvm.org/docs/DeclarativeRewrites/) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The file declares itself as TF-to-XLA legalization pattern definitions and imports TensorFlow, StableHLO/CHLO, and HLO TableGen definitions. | L16-L25 | [tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td) |
| Reusable helpers and constraints handle element type casts, TensorFlow axis conversion, and tensor type classification. | L27-L75 | [tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td) |
| Direct binary, logical, comparison, concat, collective, FFT, gather, and pad rewrites are represented as TableGen patterns. | L86-L399 | [tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td) |
| The BUILD target generates `generated_legalize_tf.inc` from this `.td` file and compiles it with `legalize_tf.cc`. | L16-L31; L120-L170 | [tensorflow/compiler/mlir/tf2xla/transforms/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/BUILD) |
| `legalize_tf.cc` supplies helper scope and describes this route as lowering TensorFlow dialect to XLA dialect. | L16-L17; L61-L75 | [tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf.cc](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf.cc) |

## Cross-Repo References

No sibling memory repo is configured. StableHLO and XLA are consumed through
Bazel dependencies in the TensorFlow checkout.

| Finding | Citations | Source Path |
|---|---|---|
| The pattern generation target depends on TensorFlow op definitions, StableHLO/CHLO op definitions, and XLA MLIR-HLO TableGen definitions. | L22-L30 | [tensorflow/compiler/mlir/tf2xla/transforms/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/tf2xla/transforms/BUILD) |

## Update History

- 2026-05-18T11:02: Added file-level onboarding during targeted MLIR legalization slice expansion.
