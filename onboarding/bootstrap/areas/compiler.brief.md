# Area Brief — compiler

| Field | Value |
|---|---|
| repo | tensorflow |
| area | compiler |
| priority | HIGH |
| confidence | [HIGH] |

Three compilation pathways: JIT (op clustering → XlaCompiler → HLO), AOT (CompileGraph → object file), MLIR (TF dialect → StableHLO). XlaCompiler is the central orchestrator. MarkForCompilationPass identifies what can be compiled.

**6 load-bearing files** — all onboarded. Key boundary: XLA backends live in `third_party/xla/`.

**Traps:** Allowlist gaps force fallback, static shape requirement, HLO backend version coupling.
