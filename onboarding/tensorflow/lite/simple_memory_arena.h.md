# simple_memory_arena.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/lite/simple_memory_arena.h` |
| governingOverview | [`tensorflow/lite/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/lite/overview.md`](../overview.md)

## Purpose

Defines `SimpleMemoryArena` — the dynamic tensor memory allocator for TFLite. During `AllocateTensors()`, the memory planner computes the lifetime of each tensor and the arena allocates offsets within a single shared buffer, reusing memory for tensors with non-overlapping lifetimes. This minimizes total memory footprint — critical for mobile/embedded deployment.

Implements `MemoryPlanner` interface.

## Code Commentary

### Logic

**`SimpleMemoryArena`:**
- `Allocate(graph_info, arena_size, node_order)` — given a `GraphInfo` and execution order, plan tensor memory allocation.
- `ResolveAllocations(tensor_indices)` — commits planned allocations.
- `GetBuffer()` — returns the underlying buffer for direct tensor data access.
- **Allocation algorithm** — greedy interval allocation: for each tensor, find the first free offset large enough. If no space, grow the arena.
- **Tensors with overlapping lifetimes** — are placed at different offsets.
- **Tensors with non-overlapping lifetimes** — may share the same offset (memory reuse).

### Conventions

- **Single buffer** — all tensors share one contiguous buffer. No per-tensor malloc/free.
- **Greedy algorithm** — simple and fast, though not optimal for all allocation patterns.

### Invariants And Boundaries

- Arena allocation is monotonic — once the plan is computed, memory is never freed.
- Tensors share memory when lifetimes don't overlap.
- The arena is a flat buffer; all offsets are computed before first Invoke.

- **Fixed arena size** — after `AllocateTensors()`, the arena size is fixed. Dynamic resizing during inference is not supported.
- **Memory reuse is deterministic** — given the same model and execution plan, the same offsets are always produced.
- **Plan is computed once** — SimpleMemoryArena finalizes its allocation plan when Commit() is called. Adding tensors after Commit() is UB.
- **Alignment requirements** — Tensors with alignment > default (16 bytes on ARM) may be allocated with wasted gaps. Check for fragmentation when using custom ops with unusual alignment.
- **Persistence through invocation** — The arena is not cleared between Invoke() calls. Output tensors from one call remain until overwritten.
- **Address stability** — Offsets are absolute, not relative. Moving the arena buffer after Commit() invalidates all tensor pointers.


### Todos

No known TODOs specific to this file.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TFLite performance and memory optimization | — | https://www.tensorflow.org/lite/performance |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Subgraph uses SimpleMemoryArena for tensor allocation | Subgraph::AllocateTensors | [`tensorflow/lite/core/subgraph.h`](../tensorflow/lite/core/subgraph.h.md) |
| MemoryPlanner is the abstract interface implemented | planner interface | `tensorflow/lite/memory_planner.h` |
| GraphInfo provides tensor node assignment for lifetime analysis | graph info | `tensorflow/lite/graph_info.h` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 4
