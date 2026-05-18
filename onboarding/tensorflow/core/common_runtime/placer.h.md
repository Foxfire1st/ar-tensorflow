# placer.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/common_runtime/placer.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `Placer` (L45-L98) — the placement algorithm that assigns every node in a Graph to a specific Device in a DeviceSet before execution. Placement is a constraint-satisfaction problem: each node may specify a device, nodes in the same colocation group must share a device, and reference-typed edges force colocation. Placer builds a constraint graph, identifies connected components, finds the intersection of valid devices for each component, and assigns one.

## Code Commentary

### Logic

**Placer (L45-L98):** Constructor takes all placement-relevant state; `Run()` performs assignment in-place on the graph.

**Constructor parameters:**
- `Graph* graph` (L72) — The graph to place. Modified in-place by Run().
- `function_name` (L73) — If the graph is a function body, the function name for error messages.
- `FunctionLibraryDefinition* flib_def` (L73) — Needed to resolve function calls referenced by ops in the graph.
- `DeviceSet* devices` (L74) — The pool of available devices.
- `default_local_device` (L75) — Preferred device for nodes without explicit device; must be local so its FunctionLibraryRuntime is directly accessible.
- `allow_soft_placement` (L76) — If true, nodes requesting a non-existent device are placed on a fallback instead of failing.
- `log_device_placement` (L76) — If true, logs every placement decision at INFO level.

**Placement algorithm (L33-L43):**
1. Existing device assignments are preserved (partial placement support).
2. Nodes with explicit device specs get that device or error.
3. Reference-typed edges force colocation.
4. Colocation groups (`@loc:A` in node B means B must share A's device).
5. Constraint graph built: nodes are vertices, colocation constraints are edges.
6. Connected components each get an intersection of valid devices.
7. Each component assigned one device from the valid set.
8. `CanAssignToDevice(device_name, devices)` (L90) checks if a named device type exists in the device set.

**Run()** (L83) — Executes placement. Not thread-safe. Must be called at most once. Modifies each NodeDef's device field in the graph.

### Conventions

- **Single-shot execution** — Run() is called once per graph; calling it twice is undefined behavior.
- **Borrowed pointers** — All pointers (graph_, flib_def_, devices_, default_local_device_) are borrowed; Placer does not own them.
- **In-place mutation** — Graph nodes are modified directly, not cloned.

### Invariants And Boundaries

- Every node gets exactly one device after Run() succeeds.
- Existing device assignments are never changed (partial placement guarantee).
- Colocation groups are transitive — `@loc:A` on B and `@loc:B` on C means A, B, C all share a device.
- Reference edges between different device types fail placement (incompatible memory).
- Graph must outlive Placer — Placer holds raw pointers.- **Run() is single-shot** — Calling Run() twice is undefined behavior. Placer modifies the graph in-place.
- **Soft placement masks errors** — `allow_soft_placement=true` silently moves ops to CPU. A GPU-optimized graph running entirely on CPU may be 100x slower with no warning.
- **Colocation groups can cause impossible constraints** — If node A requires GPU and node B requires CPU, but B has `@loc:A`, placement fails.
- **default_local_device must be local** — If set to a remote device, function calls fail because the FLR isn't directly accessible.
- **Graph must outlive Placer** — Placer holds raw pointers; if the graph is freed, Run() is a use-after-free.
- **Device type matching is string-based** — `"GPU"` won't match `"gpu"` (case-sensitive device_type comparison).

### Todos

No known TODOs specific to this file.

## Docs References

TF documentation covers the soft placement behavior that this file implements.

| Finding | Citations | Source Path |
|---|---|---|
| TF guide on using GPU — soft placement behavior | — | [TF Guide: GPU](https://www.tensorflow.org/guide/gpu) |

## Repo-Internal References

Placer is the bridge between graph construction and device execution. DirectSession calls it during Create/Extend.

| Finding | Citations | Source Path |
|---|---|---|
| Graph holds Node objects with device fields set by Placer | Graph::Node | [`graph.h`](../graph/graph.h.md) |
| DeviceSet provides the candidate device pool | DeviceSet | [`tensorflow/core/common_runtime/device_set.h`](../common_runtime/device_set.h) |
| Device represents a single compute target | Device | [`device.h`](../framework/device.h.md) |
| Colocation groups stored as node attributes | _class attr | [`tensorflow/core/graph/graph_constructor.h`](../../../tensorflow/core/graph/graph_constructor.h) |
| DirectSession runs Placer as part of CreateGraph() | DirectSession | [`direct_session.h`](direct_session.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with placement algorithm, Placer constructor parameters, Run() contract, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
