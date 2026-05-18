# File Card — tensorflow/core/graph/graph.h

| Field | Value |
|---|---|
| repo | tensorflow |
| sourceFile | `tensorflow/core/graph/graph.h` |
| targetOnboardingFile | `onboarding/tensorflow/core/graph/graph.h.md` |
| generated | 2026-05-17T17:00 |

## Classification

| Field | Value |
|---|---|
| Priority | high |
| Role | core logic |
| Risk | complex |
| Suggested action | update onboarding |
| Suggested wave | onboarding-wave-001 |

## Governing Context

| Field | Value |
|---|---|
| nearestGoverningOverview | `onboarding/tensorflow/core/overview.md` |
| ancestorOverviews | `onboarding/overview.md` |
| localArea | core |

## Why This File Matters

The Graph DAG is the fundamental computation IR for TensorFlow. All operations, sessions, and compilers operate on Graph instances. Node and Edge are the building blocks.

## What The Worker Must Explain

- Purpose: computation DAG with Node/Edge
- Main logic: add/remove nodes, edge management, iteration
- Non-obvious behavior: control edges vs data edges; node op type binding
- Invariants: DAG (no cycles); each Node belongs to exactly one Graph
- Interfaces: OpKernel construction from nodes; Session traversal
- Failure modes: cycles detected at add time; invalid op type registration
- Update risks: GraphDef serialization must remain backward-compatible

## Inputs For Worker

| Input | Path | Required? | Why |
|---|---|---|---|
| Source file | `tensorflow/core/graph/graph.h` | yes | concrete behavior |
| Nearest governing overview | `onboarding/tensorflow/core/overview.md` | yes | local area model |
| Existing onboarding | `onboarding/tensorflow/core/graph/graph.h.md` | yes | preserve/update |
| Area report | `bootstrap/areas/core.md` | yes | local context |
| Docs pack | `bootstrap/evidence/docs/core.docs-pack.md` | no | no domain docs for C++ internals |
| Boundary pack | `bootstrap/evidence/cross-repo/core.boundary-pack.md` | no | no cross-repo boundaries |

## Known Traps

- Control edges create dependencies without data flow; easy to confuse with data edges
- Graph construction vs Graph execution are separate phases

## Done When

- Onboarding updated with evidence from file card and area report.
- Commit hash backfilled.
