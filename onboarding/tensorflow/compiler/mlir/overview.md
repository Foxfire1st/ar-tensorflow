# MLIR Overview

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `route-local-overview` |
| sourceRoute | `tensorflow/compiler/mlir/` |
| onboardingRoute | `tensorflow/compiler/mlir/overview.md` |
| parentOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-18T11:02 |
| lastVerifiedCommitHash | `1cc65a53e0140d9c1a37071a50f702644c3ed4e0` |
| lastVerifiedCommitDate | 2026-05-15T06:40:10-07:00 |

## What This Area Is

TensorFlow's MLIR integration layer. It owns TensorFlow MLIR dialect definitions,
pass registration, TF dialect transformations, TF-to-XLA bridge utilities,
TF-to-StableHLO conversion, TFLite and quantization MLIR flows, and command-line
tools used to exercise those passes. It is the compiler-facing bridge between
TensorFlow graph/function representations and MLIR dialect conversion pipelines.

## What Belongs Here

| Path | Role |
|---|---|
| `tensorflow/` | TensorFlow MLIR dialect, ops, analyses, transforms, and translation helpers |
| `tf2xla/` | MLIR bridge pieces that lower TensorFlow dialect programs toward XLA/HLO execution |
| `stablehlo/` | StableHLO-facing conversion, Python extension, and pass pipeline wrappers |
| `tensorflow_to_stablehlo/` | SavedModel-to-StableHLO export path |
| `lite/` | TFLite MLIR dialect, conversion, optimization, and tests |
| `quantization/` | Quantization passes for TensorFlow and StableHLO flows |
| `tfrt/`, `tosa/`, `tfr/` | Specialized runtime or dialect integration paths |
| `tools/` | MLIR optimization and kernel-generation tools |

## What Does Not Belong Here

| Nearby Thing | Belongs Instead In |
|---|---|
| Graph-based XLA symbolic execution APIs | `tensorflow/compiler/tf2xla/` |
| JIT clustering and device compilation wrappers | `tensorflow/compiler/jit/` |
| XLA HLO IR and backend code generation | `third_party/xla/` |
| Python eager or `tf.function` frontend behavior | `tensorflow/python/` |

## Structures Found Here

The root `BUILD` file describes this route as TensorFlow/TensorFlow Lite/XLA
MLIR dialects and tools. It exposes all `.td` files through a `td_files`
filegroup, builds the `tf-opt` and `tf-reduce` tools, and assembles shared pass
registrations through `tf_mlir_opt_main`, `passes`, `init_mlir`, and
`register_common_dialects`.

## Operating Model

MLIR routes in this subtree are organized by dialect or conversion target. The
route-level `BUILD` file wires common tools and dialect registration, while child
packages generate pass declarations or rewrite patterns from TableGen files and
compile C++ pass implementations that consume those generated includes.

## Main Flows

### Common Tool And Pass Wiring

1. TableGen and C++ pass definitions live in child routes.
2. The root `tf_mlir_opt_main` target links TensorFlow, TF-to-XLA, quantization,
   and HLO-related pass sets.
3. `tf-opt` wraps that main target for local pass testing.

### TF Toward XLA Or StableHLO

1. TensorFlow dialect IR enters a child conversion route.
2. Declarative rewrite patterns and C++ helpers legalize individual operations.
3. The route-specific pass marks TensorFlow/CHLO operations illegal or legal and
   applies conversion patterns into MHLO, StableHLO, or bridge-specific forms.

## Load-Bearing Files

| File | Role | Why It Matters | Onboarding |
|---|---|---|---|
| `BUILD` | package wiring | Declares route tools, all `.td` file aggregation, and shared pass registration targets | deferred |
| `tf2xla/transforms/legalize_tf_patterns.td` | rewrite pattern source | Generates TensorFlow-to-HLO legalization patterns for the TF2XLA path | covered |
| `stablehlo/transforms/legalize_tf_patterns.td` | rewrite pattern source | Generates TensorFlow-to-MHLO patterns used by the StableHLO conversion pipeline | covered |

## Local Invariants And Traps

- TableGen `.td` files are source files: changing one alters generated C++ rewrite
  code even though the generated `.inc` file is not committed.
- Similar pattern names can exist in multiple child routes; distinguish TF2XLA,
  StableHLO, TFLite, and quantization paths before assuming behavior is shared.
- `third_party/xla/` is vendored in this repository; treat it as same-checkout
  source, not a separate memory repo unless cross-repo settings say otherwise.

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Root MLIR package exports all TableGen files and wires shared tools/pass registration. | L1-L29; L56-L87 | [tensorflow/compiler/mlir/BUILD](../../../../../../../tensorflow/tensorflow/compiler/mlir/BUILD) |
| Shared pass registration links TF2XLA transform targets into the root MLIR tool path. | L73-L77; L89-L103 | [tensorflow/compiler/mlir/BUILD](../../../../../../../tensorflow/tensorflow/compiler/mlir/BUILD) |

## Cross-Repo References

XLA, StableHLO, and MLIR dependencies are referenced through vendored or external
Bazel repositories in the TensorFlow checkout. No separate workspace memory repo
is configured for them.

| Finding | Citations | Source Path |
|---|---|---|
| `crossRepo.allow` is empty, so no sibling source repo is part of this memory context. | — | `system/settings.json` |

## Docs References

The useful domain context is MLIR declarative rewrites plus OpenXLA/StableHLO IR
semantics. These docs explain the compiler framework but do not replace the
TensorFlow source for specific pass behavior.

| Finding | Citations | Source Path |
|---|---|---|
| MLIR supports declarative TableGen rewrite rules that expand to rewrite patterns. | — | [MLIR Declarative Rewrite Rules](https://mlir.llvm.org/docs/DeclarativeRewrites/) |
| StableHLO is a portable operation set used between ML frameworks and ML compilers. | — | [StableHLO Specification](https://openxla.org/stablehlo/spec) |

## File-Level Onboarding Map

| Source File | Onboarding File | Status | Reason |
|---|---|---|---|
| `tensorflow/compiler/mlir/tf2xla/transforms/legalize_tf_patterns.td` | [`tf2xla/transforms/legalize_tf_patterns.td.md`](tf2xla/transforms/legalize_tf_patterns.td.md) | covered | High-value declarative legalization pattern source |
| `tensorflow/compiler/mlir/stablehlo/transforms/legalize_tf_patterns.td` | [`stablehlo/transforms/legalize_tf_patterns.td.md`](stablehlo/transforms/legalize_tf_patterns.td.md) | covered | High-value StableHLO-oriented legalization pattern source |

## Child Overviews

| Route | Why It Has Its Own Overview |
|---|---|
| [`tf2xla/transforms/overview.md`](tf2xla/transforms/overview.md) | Owns TF-to-XLA transform pass wiring and generated legalization patterns |
| [`stablehlo/transforms/overview.md`](stablehlo/transforms/overview.md) | Owns StableHLO-oriented TF legalization and export pass wiring |

## How To Use This Area

1. Start with the parent compiler overview to place MLIR relative to JIT/AOT/XLA.
2. Read this overview to choose the MLIR child route.
3. Read the child route overview nearest the transform file.
4. Read file-level onboarding for concrete `.td`, `.cc`, or `.h` files.

## Needs Verification

- Exhaustive boundaries between `tf2xla/`, `stablehlo/`, and `tensorflow_to_stablehlo/`.
- Which MLIR bridge paths are active by default for each TensorFlow frontend entry point.

## Update History

- 2026-05-18T11:02: Added targeted MLIR route overview as existing-memory slice maintenance.
