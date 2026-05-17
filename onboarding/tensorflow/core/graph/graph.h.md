# graph.h

| Field | Value |
|---|---|
| repository | tensorflow |
| sourceFile | `tensorflow/core/graph/graph.h` |
| governingOverview | [`tensorflow/core/overview.md`](overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | |
| lastVerifiedCommitDate | |

## Purpose

Defines the core computation graph data structures: `Graph`, `Node`, and `Edge`. The Graph is a directed acyclic graph (DAG) representing a TensorFlow computation. Nodes represent operations; edges represent data flow (tensor outputs→inputs) or control dependencies. The Graph is the primary intermediate representation — all computation, whether from Python eager tracing or C++ graph construction, eventually becomes a `Graph` before execution.

## Code Commentary

### Graph (L66-L531)

The `Graph` class is the top-level container. Key concepts:
- **Source and sink sentinel nodes** — every graph has `_source` (id 0) and `_sink` (id 1) nodes. All source-less nodes connect from _source; all sink-less nodes connect to _sink. This ensures the DAG has single-source/single-sink properties.
- **Node storage** — nodes stored in `nodes_` (ordered by id) and `nodes_by_name_` (name→Node* lookup).
- **Edge storage** — edges stored in `edges_` set.
- **Immutability after construction** — once `Finalize()` is called, the graph becomes immutable. No nodes or edges can be added.
- **GraphDef conversion** — `ToGraphDef()` serializes the graph to a `GraphDef` protocol buffer. `AddNode()` and `AddEdge()` are the building methods.

### Node (L90+)

`Node` represents an operation instance in the graph:
- **NodeDef** — protocol buffer defining op type, name, inputs, attributes, device.
- **Input/output types** — `input_types_` and `output_types_` record DataType for each edge.
- **Device assignment** — `assigned_device_name_` set by the placer after graph construction.
- **Properties** — accessed via `def()`, `op_def()`, `in_edges()`, `out_edges()`, `num_inputs()`, `num_outputs()`.

### Edge (L170+)

`Edge` connects a source node's output to a destination node's input:
- **Data edges** — carry tensor data; have `src_output` and `dst_input` indices.
- **Control edges** — `IsControlEdge()` returns true; no data, just ordering constraint.
- **EdgeSet** — efficient iteration over edges incident to a node.

## Key Invariants

- Graph is a strict DAG — cycles detected and rejected during construction.
- Every non-source node has at least one incoming edge (from _source or another node).
- Every non-sink node has at least one outgoing edge (to _sink or another node).
- Once `Finalize()` is called, no structural changes allowed.
- Node ids are unique and assigned sequentially.

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Graph construction and validation | graph_constructor.h, graph_constructor.cc | `tensorflow/core/graph/graph_constructor.h` |
| Graph partitioning for distributed execution | graph_partition.h | `tensorflow/core/graph/graph_partition.h` |
| Graph optimization (Grappler) consumes Graph | grappler/grappler_item.h | `tensorflow/core/grappler/grappler_item.h` |
| Executor reads Graph to build execution plan | executor.h | `tensorflow/core/common_runtime/executor.h` |
| Session accepts GraphDef (serialized Graph) | session.h | `tensorflow/core/public/session.h` |
| NodeDef protobuf definition | node_def.proto | `tensorflow/core/framework/node_def.proto` |

## Cross-Repo References

No relevant cross-repo evidence found.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Graph concepts in TF guide | — | https://www.tensorflow.org/guide/intro_to_graphs |

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap
