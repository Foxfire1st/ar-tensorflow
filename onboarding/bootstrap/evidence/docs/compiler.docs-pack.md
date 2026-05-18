# Docs Evidence Pack — compiler

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/compiler/` |
| generated | 2026-05-17T17:00 |
| sourcesResolvedFrom | `system/sources.md` |
| status | partial |

## Scope

Find domain documentation covering the XLA compiler stack: JIT clustering, XlaCompiler, DeviceCompiler, GraphCompiler, AOT compilation.

## Documentation Sources Checked

| Source Name | Source Type | Location | Result |
|---|---|---|---|
| XLA Docs | canonical online | https://openxla.org/xla | relevant (XLA architecture, HLO, compilation pipeline) |
| TensorFlow Guide | canonical online | https://www.tensorflow.org/guide | partially relevant (XLA JIT usage guide for TF users) |
| TensorFlow API Docs | canonical online | https://www.tensorflow.org/api_docs | not relevant (Python-level compile API only; no compiler internals) |
| Context7 MCP | API reference | context7-mcp skill | not checked (search unavailable) |

## Confirmed Documentation Findings

| Finding | Evidence | Source Path | Applies To | Confidence |
|---|---|---|---|---|
| XLA compiles StableHLO graphs from ML frameworks into machine instructions | openxla.org/xla/architecture (web_fetch, 2026-05-17) | https://openxla.org/xla/architecture | compiler/ area | [HIGH] |
| Compilation steps: target-independent optimizations (CSE), target-specific lowering, backend codegen | openxla.org/xla/architecture | — | XlaCompiler, HLO pipeline | [HIGH] |
| XLA supports JIT and AOT compilation modes | XLA docs | https://openxla.org/xla | `jit/`, `aot/` sub-areas | [HIGH] |
| XLA improves execution speed via op fusion, memory via buffer analysis, portability via StableHLO | openxla.org/xla/architecture | — | compiler/ area | [HIGH] |

## Documentation Constraints

- [MEDIUM] XLA documentation on openxla.org covers the HLO-level pipeline but not the TF-specific layers (XlaCompiler, MarkForCompilationPass, GraphCompiler). These are TensorFlow-internal concepts.
- [MEDIUM] The boundary between TF's `compiler/` and the standalone XLA project (`third_party/xla/`) is not documented in external docs.

## Terms And Definitions

| Term | Meaning | Evidence |
|---|---|---|
| HLO | High-Level Operations — XLA's intermediate representation | openxla.org/xla/architecture |
| StableHLO | Versioned, standardized HLO subset for cross-framework portability | openxla.org/xla/architecture |
| JIT compilation | Runtime clustering and compilation of TF subgraphs | XLA docs |
| AOT compilation | Offline compilation of TF graphs to standalone object files | XLA docs |

## Files Likely Affected

| File | Why Docs Matter |
|---|---|
| `tf2xla/xla_compiler.h` | XLA architecture docs explain the HLO pipeline this orchestrates |
| `aot/compile.h` | XLA docs document AOT concepts |
| `jit/mark_for_compilation_pass.h` | JIT auto-clustering is user-facing via TF_XLA_FLAGS |

## No Relevant Evidence Found

| Topic | Sources Checked | Notes |
|---|---|---|
| MarkForCompilationPass clustering algorithm | XLA docs, TF Guide | JIT clustering is mentioned at usage level; no algorithm documentation |
| DeviceCompiler caching strategy | XLA docs | Caching is an implementation detail; not in external docs |
| GraphCompiler deterministic ordering algorithm | XLA docs | Determinism is a TF-specific concern for AOT reproducibility |
| XlaCompilationDevice internals | XLA docs | The compilation device is a TF-internal bridge; no external docs |

## Open Questions

- Is there a TF RFC documenting the JIT clustering algorithm and allowlist maintenance?
- What is the current status of the StableHLO migration for the TF→XLA pathway?
