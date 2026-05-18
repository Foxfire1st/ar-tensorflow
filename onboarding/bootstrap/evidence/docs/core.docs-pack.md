# Docs Evidence Pack — core

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/core/` |
| generated | 2026-05-17T17:00 |
| sourcesResolvedFrom | `system/sources.md` |
| status | no-relevant-evidence |

## Scope

Find domain documentation covering the C++ runtime internals (Graph, Executor, OpKernel, Device, Session, DirectSession, Placer, Rendezvous, Function, Tensor).

## Documentation Sources Checked

| Source Name | Source Type | Location | Result |
|---|---|---|---|
| TensorFlow API Docs | canonical online | https://www.tensorflow.org/api_docs | not relevant (covers Python API only; no C++ internals) |
| TensorFlow Guide | canonical online | https://www.tensorflow.org/guide | not relevant (covers high-level concepts for Python users; no C++ implementation details) |
| XLA Docs | canonical online | https://openxla.org/xla | not relevant (covers XLA compiler, not core runtime) |
| Context7 MCP | API reference | context7-mcp skill | not checked (search provider unavailable, but C++ runtime internals rarely appear in library API references) |

## No Relevant Evidence Found

| Topic | Sources Checked | Notes |
|---|---|---|
| Graph DAG data structure and Node/Edge model | TF API docs, TF Guide | Python-level Graph API documented; C++ internals not covered |
| Executor dispatch loop and ready-queue management | TF Guide | Execution model described at high level for users; no C++ implementation docs |
| OpKernel::Compute contract and lifecycle | TF API docs | Custom op creation guide covers Python wrapper; C++ kernel interface is header-comment-only |
| Session::Create/Run/Close lifecycle | TF Guide | Session API documented for Python/Java; C++ DirectSession internals not covered |
| BFCAllocator memory management | TF Guide | Memory growth options documented for users; BFCAllocator algorithm not documented |
| Device abstraction and placement | TF Guide | Device placement for users documented; Placer algorithm not in domain docs |
| Rendezvous producer-consumer model | (none found) | Cross-device tensor passing is an internal mechanism; no external docs |
| Function library and instantiation | TF Guide | tf.function documented for Python; C++ FunctionLibraryRuntime not covered |

## Conclusion

The TensorFlow domain documentation covers the Python API and high-level concepts for end users. C++ runtime internals are not part of the domain documentation surface. For the core/ area, code comments and header-level documentation are the primary reference. This is expected — the C++ runtime is an implementation detail, not a documented API surface.

## Files Affected

All 12 core/ onboarded files. Domain docs provide no additional coverage.

## Open Questions

- Are there internal Google design docs (TF RFCs) that cover core runtime architecture decisions? These would be more useful than public domain docs.
