# StableHLO MLIR Transforms Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/compiler/mlir/stablehlo/transforms/` |
| onboardingRoute | `tensorflow/compiler/mlir/stablehlo/transforms/overview.md` |
| parentOverview | [`tensorflow/compiler/mlir/overview.md`](../../overview.md) |
| lastUpdated | 2026-05-19T03:08:25+02:00 |
| lastVerifiedCommitHash | `0c5506a91576d768dff8dd522d9b5240e64d2666` |
| lastVerifiedCommitDate | 2026-03-16T18:23:06-07:00 |

## What This Area Is

StableHLO-oriented MLIR transformation code. This route contains rewrite
patterns and pass implementations that lower TensorFlow dialect operations
toward MHLO/StableHLO, fold broadcasts, legalize TF XlaCallModule operations,
and assemble the TensorFlow-to-StableHLO pass pipeline.

## Hot Path Summary

Use this route for StableHLO-oriented TensorFlow legalization and export pipeline behavior. Strong anchors are `legalize_tf_patterns.td`, `TF_CheckNumericsOp`, `tf_stablehlo_pass.cc`, `legalize_tf.cc`, `generated_legalize_tf.inc`, and `legalize_tf_xla_call_module_to_stablehlo_pass`.

## What Belongs Here

| Path | Role |
|---|---|
| `legalize_tf_patterns.td` | Declarative TensorFlow-to-MHLO rewrite patterns |
| `legalize_tf.cc` | C++ helper/pass implementation for the generated patterns |
| `tf_stablehlo_pass.cc` | Pipeline wrapper that runs TF-to-MHLO patterns, CHLO-to-HLO, and StableHLO conversion |
| `legalize_tf_xla_call_module_to_stablehlo_pass.*` | XlaCallModule-specific StableHLO legalization |
| `fold_broadcast_pass.*`, `utils.*` | Supporting transforms and utilities |

## What Does Not Belong Here

| Nearby Thing | Belongs Instead In |
|---|---|
| TF2XLA bridge-specific fallback rewriter | `tensorflow/compiler/mlir/tf2xla/transforms/` |
| SavedModel export orchestration | `tensorflow/compiler/mlir/tensorflow_to_stablehlo/` |
| TFLite StableHLO import/export paths | `tensorflow/compiler/mlir/lite/stablehlo/` |

## Structures Found Here

The parent `stablehlo/BUILD` file generates `transforms/generated_legalize_tf.inc`
from `transforms/legalize_tf_patterns.td`, builds a public `legalize_tf` library,
and builds `tf_stablehlo`, which consumes both this route's legalization and the
TF2XLA transform fallback before legalizing HLO to StableHLO.

## Operating Model

This route first lowers TensorFlow dialect ops toward MHLO/CHLO-friendly forms,
then its pipeline converts HLO-family IR toward StableHLO. It shares many pattern
shapes with TF2XLA transform code but uses MHLO naming and the StableHLO package's
pass wrappers.

## Main Flows

### Generated Pattern Legalization

1. `legalize_tf_patterns.td` defines TensorFlow dialect rewrite patterns.
2. The StableHLO package `BUILD` target generates `transforms/generated_legalize_tf.inc`.
3. `legalize_tf.cc` compiles helpers and generated patterns into `legalize_tf`.

### TF To StableHLO Pipeline

1. `tf_stablehlo_pass.cc` populates this route's legalize-TF patterns.
2. It also adds TensorFlow lowering-before-HLO and TF2XLA fallback patterns.
3. The pipeline legalizes CHLO/MHLO forms and then runs HLO-to-StableHLO conversion.

## Load-Bearing Files

| File | Role | Why It Matters | Onboarding |
|---|---|---|---|
| `legalize_tf_patterns.td` | rewrite definitions | Encodes TensorFlow op rewrites used by StableHLO-oriented lowering | covered |
| `legalize_tf.cc` | pass implementation | Owns helper functions and includes generated patterns | deferred |
| `tf_stablehlo_pass.cc` | pipeline wrapper | Wires pattern population, target legality, CHLO lowering, and StableHLO conversion | deferred |
| `../BUILD` | build wiring | Generates rewrite includes and builds StableHLO transform targets | deferred |

## Local Invariants And Traps

- This route's `legalize_tf_patterns.td` targets MHLO/CHLO naming, while the
  TF2XLA transform file targets StableHLO/CHLO naming in the current checkout.
- The pipeline can reuse TF2XLA fallback patterns; behavior may depend on both
  this route and `tensorflow/compiler/mlir/tf2xla/transforms/`.
- The generated include is not checked in; TableGen source and C++ helpers are
  the review surface.

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| The StableHLO package generates `transforms/generated_legalize_tf.inc` from this route's TableGen pattern file. | L74-L92 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |
| The `legalize_tf` library compiles generated patterns and helper code. | L145-L189 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |
| The `tf_stablehlo` target depends on local legalization and TF2XLA transform fallback. | L191-L225 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |
| `tf_stablehlo_pass.cc` populates local legalize-TF patterns, TF lowering-before-HLO patterns, TF2XLA fallback patterns, and CHLO-to-HLO patterns. | L98-L110 | [tensorflow/compiler/mlir/stablehlo/transforms/tf_stablehlo_pass.cc](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/transforms/tf_stablehlo_pass.cc) |

## Cross-Repo References

No sibling memory repo is configured. StableHLO and XLA are consumed through
Bazel targets available to this TensorFlow checkout.

| Finding | Citations | Source Path |
|---|---|---|
| StableHLO route targets depend on StableHLO and XLA MLIR-HLO Bazel deps. | L176-L188; L218-L223 | [tensorflow/compiler/mlir/stablehlo/BUILD](../../../../../../../../../tensorflow/tensorflow/compiler/mlir/stablehlo/BUILD) |

## Docs References

StableHLO documentation is relevant for the target IR. MLIR declarative rewrite
docs explain the TableGen pattern source model used here.

| Finding | Citations | Source Path |
|---|---|---|
| StableHLO is a portable HLO operation set for ML frameworks and compilers. | — | [StableHLO Specification](https://openxla.org/stablehlo/spec) |
| MLIR DRR explains TableGen rewrite rules generated into `RewritePattern` code. | — | [MLIR Declarative Rewrite Rules](https://mlir.llvm.org/docs/DeclarativeRewrites/) |

## File-Level Onboarding Map

| Source File | Onboarding File | Status | Reason |
|---|---|---|---|
| `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td` | [`legalize_tf_patterns.td.md`](legalize_tf_patterns.td.md) | covered | Core declarative rewrite surface |

## Child Overviews

None.

## How To Use This Area

1. Read this overview before editing StableHLO transform files.
2. For `.td` pattern edits, read the file-level onboarding and nearby
   `legalize_tf.cc` helpers.
3. Check `tf_stablehlo_pass.cc` when behavior depends on pass ordering or
   fallback pattern population.

## Needs Verification

- Exact relationship between this route and `tensorflow_to_stablehlo/` export paths.
- Which users enter through `tf_stablehlo` versus other StableHLO conversion APIs.

## Update History

- 2026-05-19T03:08:25+02:00: Added hot path summary for generated route-index discovery hints.
- 2026-05-18T11:02: Added targeted route overview for StableHLO transform coverage.
