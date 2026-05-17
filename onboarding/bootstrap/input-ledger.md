# Bootstrap Input Ledger — tensorflow

| Field | Value |
|---|---|
| targetRepo | tensorflow |
| controlMode | automated |
| bootstrapMode | full-bootstrap |
| memoryRoot | ar-coordination/memory-repos/ar-tensorflow |
| onboardingRoot | ar-coordination/memory-repos/ar-tensorflow/onboarding |
| topology | external |
| targetBranch | master |
| generated | 2026-05-17T00:00 |
| sourceInventoryGate | accepted |

## Presented Source Inventory

| Source | Category | Location | Status | Planned Use | User Decision |
|---|---|---|---|---|---|
| TensorFlow API Docs | Domain Documentation | tensorflow.org/api_docs | readable | API surface verification, symbol intent | approved |
| TensorFlow Guide | Domain Documentation | tensorflow.org/guide | readable | Architecture concepts, high-level design | approved |
| XLA Docs | Domain Documentation | openxla.org/xla | readable | Compiler stack understanding | approved |
| Context7 MCP | API Reference | context7-mcp skill | readable | Library/API lookups during file onboarding | approved |
| README.md | Project Docs | repo root | readable | Repo overview, build instructions | approved |
| CONTRIBUTING.md | Project Docs | repo root | readable | Dev workflow, conventions | approved |
| SECURITY.md | Project Docs | repo root | readable | Security boundaries | approved |
| RELEASE.md | Project Docs | repo root | readable | Versioning, release process | approved |
| ISSUES.md | Project Docs | repo root | readable | Issue tracking guidance | approved |
| CITATION.cff | Project Docs | repo root | readable | Citation metadata | approved |
| CODE_OF_CONDUCT.md | Project Docs | repo root | readable | Community norms | approved |
| tensorflow/python/ | Source Code | tensorflow/ | readable | High-level Python API, Keras, eager execution | approved |
| tensorflow/core/ | Source Code | tensorflow/ | readable | C++ runtime, graph execution, kernels | approved |
| tensorflow/compiler/ | Source Code | tensorflow/ | readable | XLA, MLIR, HLO, JIT compilation | approved |
| tensorflow/c/ | Source Code | tensorflow/ | readable | Stable C API surface | approved |
| tensorflow/cc/ | Source Code | tensorflow/ | readable | C++ API, saved model, session | approved |
| tensorflow/lite/ | Source Code | tensorflow/ | readable | Mobile/embedded runtime | approved |
| tensorflow/js/ | Source Code | tensorflow/ | readable | TensorFlow.js | approved |
| tensorflow/go/ | Source Code | tensorflow/ | readable | Go bindings | approved |
| tensorflow/java/ | Source Code | tensorflow/ | readable | Java bindings | approved |
| tensorflow/dtensor/ | Source Code | tensorflow/ | readable | Distributed tensor/mesh support | approved |
| tensorflow/security/ | Source Code | tensorflow/ | readable | Security fuzzing, vuln response | approved |
| tensorflow/tools/ | Source Code | tensorflow/ | readable | Internal tooling, pip packaging | approved |
| tensorflow/examples/ | Source Code | tensorflow/ | readable | Example models and tutorials | approved |
| tensorflow/docs_src/ | Source Code | tensorflow/ | readable | Documentation source files | approved |
| tools/ | Source Code | repo root | readable | Repo-level build/dev tools | approved |
| ci/ | Source Code | repo root | readable | CI/CD configuration | approved |

## Sources I Will Not Use

| Source | Category | Reason |
|---|---|---|
| third_party/ | Vendored Code | Excluded by pathRules; third-party dependencies |
| .git/ | VCS | Excluded by pathRules |

## Missing Or Weak Source Categories

| Category | Why It May Matter | Action |
|---|---|---|
| Internal design docs / RFCs | TensorFlow design decisions | parked; domain docs on tensorflow.org cover most concepts |
| Cross-repo dependencies | May interact with Keras, JAX | crossRepo.allow is empty; no cross-repo reads planned |

## Additional Sources Provided By User

| Source | Category | Location | Reason |
|---|---|---|---|
| (none) | — | — | — |

## Source Corrections

| Original Finding | Correction | Reason |
|---|---|---|
| (none) | — | — |

## Settings Path-Rule Exclude Review

Standard excludes present in resolved `system/settings.json`:
- ✅ node_modules/**
- ✅ vendor/**
- ✅ dist/**
- ✅ build/**
- ✅ coverage/**
- ✅ .cache/**
- ✅ .pytest_cache/**
- ✅ .venv/**
- ✅ .idea/**
- ✅ .vscode/**
- ✅ .env / .env.*
- ✅ **/generated/** / **/*.generated.*
- ✅ **/*.Zone.Identifier / **/*:Zone.Identifier
- Additional TF-specific: third_party/**, **/*.textproto, **/*.lds, opensource_only.files, **/target/**

## Cross-Repo Context From Settings

| Adjacent Repo | Expected Branch | Actual Branch | Status | Allowed Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Bootstrap Assumptions

- The `tensorflow/tensorflow/` directory is the main source tree, organized by subsystem (python, core, compiler, lite, c, cc, etc.)
- Build system is Bazel; BUILD files carry build graph structure
- Python package lives under `tensorflow/python/`
- C++ runtime core under `tensorflow/core/`
- XLA compiler under `tensorflow/compiler/`
- No cross-repo dependencies are active for this bootstrap

## Hard Stop Conditions For This Run

- memory root cannot be resolved
- required source cannot be read
- multiple plausible source meanings exist
- cross-repo branch mismatch for a required boundary
- LOW-confidence claim would become durable fact
- docs and code conflict in a way the agent cannot resolve
- output would require updating a non-target repo

## Operator Decision

Selected control mode: automated
Additional sources supplied: no
Proceed: yes
