# graph.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/graph/graph.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `Graph` (L60-L400) — the in-memory representation of a TensorFlow computation graph. A Graph holds nodes (ops), edges (tensor connections), a function library, and version counters. `GraphDef` ↔ `Graph` conversion happens here via `GraphConstructor`. The Graph is the canonical representation consumed by placement, partitioning, optimization, and executor construction.

Also defines `Node` (L50-L350), `Edge` (L55-L80), and `NodeProperties` (L45-L60) — the building blocks of the graph data structure. Graph is not thread-safe for mutation.

## Code Commentary

### Logic

**Node (L50-L350):** A graph node representing one operation:
- `def()` — Returns the `NodeDef` proto (op type, name, attributes, input names).
- `in_edges()` / `out_edges()` — Iterator ranges over incoming/outgoing edges.
- `assigned_device_name()` — The device set by the Placer. Empty until placement.
- `num_inputs()` / `num_outputs()` — Tensor counts from the OpDef signature.
- `Properties()` — Returns `NodeProperties` (OpDef + DataTypes per input/output).
- `AddAttr(name, value)` — Set node-specific attributes after construction.
- `input_types()` / `output_types()` — DataType per tensor index.

**Edge (L55-L80):** Directed connection: `src` → `dst` with `src_output` index and `dst_input` index. Edges are owned by the Graph.

**Graph (L60-L400):** The graph container:
- `AddNode(node_def, &node)` (L200) — Creates a Node from a NodeDef. Returns Status + Node*. Takes ownership.
- `RemoveNode(node)` (L210) — Removes a node and all its edges. Node is invalidated.
- `AddEdge(src, src_output, dst, dst_input)` (L220) — Adds a directed edge. CHECK-fails on invalid indices.
- `FindNodeId(id)` (L230) — Lookup by node ID. Returns nullptr if not present.
- `ToGraphDef(&graph_def)` (L240) — Serializes the graph to GraphDef proto.
- `ToGraphDefSubRange(start, end, &graph_def)` (L245) — Serializes a node ID range.
- `versions()` (L250) — Returns VersionDef tracking graph mutation count. Check this to detect changes.
- `flib_def()` (L260) — Returns the FunctionLibraryDefinition for functions referenced by this graph.
- `IsValidNode(node)` (L270) — True if the node belongs to this graph.
- `op_registry()` (L280) — Returns the OpRegistryInterface for looking up OpDefs.
- `num_nodes()` / `num_edges()` / `nodes()` / `edges()` — Sizing and iteration.
- `CopyNode(src)` (L300) — Deep-copies a node into this graph.

**EdgeSet:** Custom hash set for edges using `std::vector` + hash table hybrid. Optimized for small sets (most nodes have few edges).

### Conventions

- **Graph owns nodes and edges** — Nodes and edges are heap-allocated and freed when the Graph is destroyed.
- **Node ID stability** — Node IDs are stable after AddNode(). Use FindNodeId() for lookup.
- **EdgeSet performance** — Edge iteration is fast; adding edges while iterating is unsafe.

### Invariants And Boundaries

- Every edge connects two nodes in the same Graph.
- Node ID 0 is reserved (sentinel).
- Edge src_output and dst_input indices must be in range for the respective nodes' OpDefs.
- Graph construction is single-threaded; concurrent AddNode/AddEdge causes data races.
- Once finalized, graph structure is immutable — optimization passes create new graphs.- **Node ownership** — Nodes added via AddNode() are owned by the Graph. Deleting a raw Node* obtained from FindNodeId() is undefined behavior.
- **Edge invalidation** — Adding edges while iterating over in_edges() or out_edges() invalidates iterators. Collect changes then apply after iteration.
- **NodeDef vs Node** — ToGraphDef() produces a snapshot. Modifying the graph after does not update the proto. Call ToGraphDef() again.
- **Version counter semantics** — Not all changes bump the version count (e.g., internal optimizations). Don't rely on the version counter for fine-grained change detection.
- **Unused function library entries** — Adding a FunctionDef without adding its call site produces a valid GraphDef but wastes memory. Some optimizers skip unused functions; some don't.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. Graph is an internal C++ data structure not covered by public TF documentation.

## Repo-Internal References

Graph is the intermediate representation between GraphDef serialization and execution. Every pipeline stage (construction, placement, partitioning, optimization) reads or modifies a Graph.

| Finding | Citations | Source Path |
|---|---|---|
| GraphConstructor converts GraphDef to Graph | GraphConstructor | [`tensorflow/core/graph/graph_constructor.h`](graph_constructor.h) |
| Placer assigns devices to nodes in the graph | Placer | [`placer.h`](../common_runtime/placer.h.md) |
| Executor runs the finalized graph | Executor | [`executor.h`](../common_runtime/executor.h.md) |
| OpRegistry used to resolve op types during construction | OpRegistry | [`op.h`](../framework/op.h.md) |
| FunctionLibraryDefinition for function references | FunctionLibraryDefinition | [`function.h`](../framework/function.h.md) |
| NodeDef protobuf for serialized node representation | NodeDef proto | [`tensorflow/core/framework/node_def.proto`](../framework/node_def.proto) |
| GraphDef protobuf for serialized graph representation | GraphDef proto | [`tensorflow/core/framework/graph.proto`](../framework/graph.proto) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added traps section for Node ownership, edge invalidation, version counter semantics, unused function entries.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
