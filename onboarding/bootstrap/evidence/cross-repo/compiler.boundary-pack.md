# Cross-Repo Boundary Pack — compiler

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/compiler/` |
| generated | 2026-05-17T17:00 |
| topology | external |
| status | no-boundary-found |

## Allowed Adjacent Repos

| Adjacent Repo | Expected Branch | Actual Branch | Status | Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Boundary Summary

No cross-repo boundaries configured. However, the compiler area has a **critical structural boundary** to the XLA/StableHLO backend in `third_party/xla/`. While vendored, this boundary behaves like a cross-repo interface — XLA has its own project, release cycle, and API surface.

## Vendored Dependencies With Cross-Repo Character

| Dependency | Location | Role | Interface |
|---|---|---|---|
| XLA (HLO IR, service, backends) | `third_party/xla/xla/` | Receives HLO computations from XlaCompiler; provides JIT compilation service | `xla::XlaBuilder`, `xla::XlaComputation`, `xla::Service`, `xla::LocalClient` |
| MLIR/StableHLO | `third_party/llvm-project/mlir/`, `third_party/stablehlo/` | MLIR compilation pathway: TF dialect → StableHLO → backend | MLIR dialects, passes |
| StreamExecutor | `third_party/xla/xla/stream_executor/` | GPU stream management, kernel launches | `se::Stream`, `se::DeviceMemory` |

## Confirmed Interfaces (Vendored Boundary)

| Direction | This Repo | Vendored Component | Interface | Evidence | Confidence |
|---|---|---|---|---|---|
| outbound | `tf2xla/xla_compiler.h` → `XlaCompiler::CompileFunction()` | `xla::XlaBuilder` + `xla::XlaComputation` | Build HLO graph, compile to executable | Header imports + API usage | [HIGH] |
| outbound | `jit/xla_device.h` → `XlaDevice::Compute()` | `xla::LocalClient` + `xla::LocalExecutable` | Execute compiled HLO on device | Header imports | [HIGH] |
| outbound | `aot/compile.h` → `CompileGraph()` | `xla::cpu::CpuAotCompilationResult` | Produce object file from HLO | Header imports | [HIGH] |
| outbound | `jit/device_compiler.h` → `DeviceCompiler` | `xla::LocalClient::Compile()` | Compile and cache executables | Header imports | [HIGH] |

## Shared Contracts

| Contract | This Repo Location | Vendored Location | Sync Requirement | Confidence |
|---|---|---|---|---|
| HLO graph structure (XlaBuilder API) | `tf2xla/xla_compiler.cc` | `xla/client/xla_builder.h` | Must produce valid HLO; API changes break compilation | [HIGH] |
| XlaComputation serialization format | `tf2xla/` | `xla/service/hlo.proto` | HLO proto format must match | [HIGH] |
| StreamExecutor kernel launch API | `compiler/jit/` | `xla/stream_executor/` | GPU kernel launches through StreamExecutor | [MEDIUM] |

## Branch And Topology Notes

- XLA is vendored at a pinned commit in the TF workspace (`tf_http_archive` in WORKSPACE).
- OpenXLA project maintains XLA as a separate entity; TF vendors a snapshot.
- MLIR/StableHLO pathway is under active migration; the TF dialect → StableHLO path may supersede the XlaBuilder path.

## Same-Repo Facts That Must Not Be Classified As Cross-Repo

| Fact | Correct Bucket |
|---|---|
| XlaCompiler calling xla::XlaBuilder | Vendored (same repo checkout, different project) |
| StreamExecutor GPU stream management | Vendored |
| StableHLO dialect usage in MLIR pathway | Vendored |

## Boundary Risks

- [MEDIUM] XLA backend API changes (pin updates in WORKSPACE) can break TF compilation; not caught until pin update
- [MEDIUM] StableHLO migration may change the HLO generation surface; existing XlaBuilder-based code may need migration
- [LOW] StreamExecutor API deprecated in favor of newer XLA runtime APIs

## Needs Developer Confirmation

- Should XLA be configured as a separate adjacent repo in `settings.json`? It is a separate project with its own governance.
- Should StreamExecutor be tracked as a separate interface boundary?
