# Cross-Repo Boundary Pack — core

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/core/` |
| generated | 2026-05-17T17:00 |
| topology | external |
| status | no-boundary-found |

## Allowed Adjacent Repos

| Adjacent Repo | Expected Branch | Actual Branch | Status | Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Boundary Summary

No cross-repo boundaries are configured in `system/settings.json` (`crossRepo.allow` is empty). The core runtime has significant dependencies on vendored libraries (Eigen, protobuf, abseil, gRPC), but these are in `third_party/` and are not separate repos in the ar-coordination system.

## Vendored Dependencies (NOT cross-repo but worth noting)

| Dependency | Location | Role | Interface |
|---|---|---|---|
| Eigen | `third_party/eigen3/` | Tensor operations, linear algebra | `Tensor::matrix()`, `Tensor::vec()` return Eigen types |
| Protobuf | `third_party/protobuf/` | GraphDef, NodeDef, MetaGraphDef serialization | All `.proto` in `core/protobuf/` generate from protobuf |
| Abseil | `third_party/absl/` | C++ utilities (Status, string_view, flat_hash_map) | Used throughout core/ |
| gRPC | `third_party/grpc/` | Distributed runtime RPC | `distributed_runtime/rpc/` uses gRPC service stubs |

## Same-Repo Facts That Must Not Be Classified As Cross-Repo

| Fact | Correct Bucket |
|---|---|
| Eigen tensor operations through Tensor::matrix() | Repo-Internal (vendored, not cross-repo) |
| Protobuf GraphDef serialization | Repo-Internal (vendored) |
| gRPC Master/Worker service | Repo-Internal (vendored) |

## Boundary Risks

- None at the cross-repo level. Vendored dependencies are pinned via Bazel workspace and carry version lock risks.

## Needs Developer Confirmation

- Should Eigen, protobuf, or gRPC be configured as adjacent repos in `settings.json`? They currently are not.
