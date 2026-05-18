# legalize_tf_patterns.td

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T11:02 |
| lastVerifiedCommitHash | `573bbe2b4163fb4349c8c662635f99cca40c8406` |
| lastVerifiedCommitDate | 2025-12-22T09:54:11-08:00 |
| governingOverview | [`tensorflow/compiler/mlir/stablehlo/transforms/overview.md`](overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/mlir/stablehlo/transforms/overview.md`](overview.md)

## Purpose

Declarative rewrite pattern source for the StableHLO-oriented TensorFlow
legalization path. This TableGen file maps TensorFlow dialect operations toward
MHLO/CHLO forms, with helper logic supplied by `legalize_tf.cc`. The StableHLO
package generates `transforms/generated_legalize_tf.inc` from this file, then
the `tf_stablehlo` pipeline composes these patterns with additional TensorFlow
lowering, TF2XLA fallback, CHLO-to-HLO lowering, and StableHLO conversion.

## Code Commentary

### Logic

- Imports MLIR core dialect TableGen files, CHLO ops, TensorFlow op definitions,
  and MHLO op definitions.
- Defines TensorFlow-compatible tensor constraints and NativeCodeCall helpers
  for axis conversion, element conversion, precision config, and shape-dependent
  legalization logic.
- Uses `MHLO_*` and `CHLO_*` targets for direct TensorFlow op rewrites.
- Expands TensorFlow semantic differences where HLO semantics differ, for
  example integer floor division, floor mod, concat axis normalization, and
  padding conversion.
- Removes or forwards operations when no runtime HLO construct is represented in
  this lowering path.

### Conventions

- This file is similar in shape to the TF2XLA transform pattern file but uses
  MHLO target names in this checkout.
- `TF_*Op` names are TensorFlow dialect sources; `MHLO_*Op` and `CHLO_*Op` names
  are target operations.
- `NativeCodeCall` helper names must remain available in the StableHLO
  `legalize_tf.cc` helper scope.
- `replaceWithValue` forwards the original operand and erases the matched op.

### Invariants And Boundaries

- Keep the TableGen file, generated include target, and C++ helper functions in
  sync.
- Do not assume behavior matches TF2XLA transforms exactly; this route feeds a
  StableHLO-oriented pipeline and may compose extra patterns or conversions.
- Target dialect dependencies belong in `tensorflow/compiler/mlir/stablehlo/BUILD`.
- Comments about unsupported HLO semantics are review-relevant and should be
  preserved unless the actual lowering support changes.

### Todos

- TODO markers record unsupported or incomplete legalization behavior, including
  assertions and CheckNumerics support in HLO.

### Docs References

StableHLO docs explain the target portability layer, while MLIR DRR docs explain
the generated rewrite model.

| Finding | Citations | Source Path |
|---|---|---|
| StableHLO is a portable HLO operation set for ML frameworks and compilers. | — | [StableHLO Specification](https://openxla.org/stablehlo/spec) |
| MLIR Declarative Rewrite Rules describe TableGen rewrite rules expanded into rewrite pattern code. | — | [MLIR Declarative Rewrite Rules](https://mlir.llvm.org/docs/DeclarativeRewrites/) |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The file declares itself as TF-to-XLA legalization pattern definitions and imports TensorFlow, CHLO, and MHLO TableGen definitions. | L16-L24 | [tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td) |
| Reusable helpers and constraints handle element type casts, TensorFlow axis conversion, and tensor type classification. | L26-L67 | [tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td) |
| Direct binary, logical, comparison, concat, collective, FFT, gather, and pad rewrites are represented as TableGen patterns. | L78-L391 | [tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td) |
| The StableHLO BUILD target generates `transforms/generated_legalize_tf.inc` from this `.td` file and compiles it into `legalize_tf`. | L74-L92; L145-L189 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |
| `tf_stablehlo_pass.cc` composes these patterns with TensorFlow lowering, TF2XLA fallback patterns, and CHLO/HLO conversion. | L98-L110; L155-L165 | [tensorflow/compiler/mlir/stablehlo/transforms/tf_stablehlo_pass.cc](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/transforms/tf_stablehlo_pass.cc) |

## Cross-Repo References

No sibling memory repo is configured. StableHLO and XLA are consumed through
Bazel dependencies in the TensorFlow checkout.

| Finding | Citations | Source Path |
|---|---|---|
| The StableHLO transform targets depend on StableHLO and XLA MLIR-HLO Bazel deps. | L176-L188; L218-L223 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |

## Update History

- 2026-05-18T11:02: Added file-level onboarding during targeted MLIR legalization slice expansion.
