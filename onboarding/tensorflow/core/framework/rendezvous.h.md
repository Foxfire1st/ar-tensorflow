# rendezvous.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/rendezvous.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `RendezvousInterface` (L35-L147) — the abstract communication channel for passing tensors between graph nodes that may reside on different devices or different machines. The Rendezvous is a table of named channels keyed by `<producer, consumer>` pairs. Producers call `Send()` (non-blocking), consumers call `Recv()` (blocking or callback-based). This is the core tensor transport layer beneath the graph executor — every cross-device edge in the graph flows through a Rendezvous.

## Code Commentary

### Logic

**RendezvousInterface (L35-L147):** Abstract base with four core methods:

- `Send(key, args, tensor, is_dead)` (L80) — Non-blocking send. Producer deposits a tensor into the channel named by `key`. `is_dead` flags a terminated channel (control flow dead branch). If a consumer is already waiting, the tensor is immediately delivered; otherwise it's stored until a consumer arrives.
- `Recv(key, args, tensor*, is_dead*, callback)` (L90) — Callback-based receive. If the tensor is available, calls `callback(OK())` synchronously. Otherwise, queues the callback to be invoked when `Send()` arrives. The callback is called at most once.
- `RecvAsync(key, args, done)` (L100) — Variant using `DoneCallback`. Same semantics as Recv.
- `StartAbort(err)` (L130) — Aborts all pending Send/Recv operations with the given error. All pending callbacks receive the abort error.

**Supporting types:**
- `Args` (L40) — Carries `DeviceContext`, `AllocatorAttributes`, and `CancellationManager` for both Send and Recv operations.
- `ParsedKey` (L50) — Parses the `CreateKey` encoding into source and destination device names for routing and tracing.
- `CreateKey(parsed_src, parsed_dst, frame_and_iter, tensor_name)` — Static factory encoding a rendezvous key from device and tensor identifiers.

### Conventions

- **Non-blocking Send** — Producers never stall. This is critical for avoiding executor deadlocks.
- **At-most-once Recv** — Each callback fires exactly once (success or error). Callers must handle both outcomes.
- **Ordered channels** — Recv() calls on the same key receive tensors in the same order they were Sent.

### Invariants And Boundaries

- Send() never blocks — producers cannot stall.
- Recv() delivers at most once — each callback is invoked exactly once.
- Channels are ordered — Recv() calls on the same key receive tensors in send order.
- StartAbort() is idempotent — only the first error matters.
- Rendezvous outlives all Send/Recv operations — callers ensure the Rendezvous is alive until all callbacks fire.
- If `is_dead` is true on Recv, the tensor is unspecified — consumers must check before reading.- **Memory ownership across devices** — The Rendezvous may need to copy tensors between devices (GPU→CPU→GPU). The allocator in Args determines where the received tensor lives.
- **Dead tensors and control flow** — `is_dead` signals that a tensor-producing branch was not executed (while_loop, cond). Missing deadness handling causes NaN propagation.
- **Cancellation races** — If Recv's CancellationManager fires between the Send arriving and the callback executing, the callback receives a cancelled error. Callbacks must be cancellation-safe.
- **Rendezvous leaks** — Callbacks captured by the Rendezvous must not capture raw pointers to stack-allocated objects that die before the callback fires.
- **Recv timeouts** — The interface does not define timeouts; implementations handle deadlock differently. In distributed settings, a slow Send stalls all waiting Recvs indefinitely.
- **StartAbort is immediate** — Any Recv callbacks queued but not yet invoked fire with the abort error in the current thread. Don't call StartAbort from inside a callback (deadlock risk).

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. Rendezvous is an internal communication layer not covered by public TF documentation.

## Repo-Internal References

The Rendezvous is the tensor transport layer beneath the graph executor. Every cross-device edge flows through it.

| Finding | Citations | Source Path |
|---|---|---|
| IntraProcessRendezvous is the single-process implementation | IntraProcessRendezvous | [`tensorflow/core/common_runtime/rendezvous_mgr.h`](../../common_runtime/rendezvous_mgr.h) |
| RendezvousMgr manages Rendezvous instances per step | RendezvousMgr | [`tensorflow/core/common_runtime/rendezvous_mgr.h`](../../common_runtime/rendezvous_mgr.h) |
| Executor::RunAsync passes Rendezvous in Args for cross-device edges | Executor::Args::rendezvous | [`executor.h`](../common_runtime/executor.h.md) |
| Send/Recv ops use Rendezvous as the graph-level transport | SendOp, RecvOp | [`tensorflow/core/kernels/sendrecv_ops.h`](../../kernels/sendrecv_ops.h) |
| CancellationManager passed in Args; Recv listens for cancellation | CancellationManager | [`tensorflow/core/framework/cancellation.h`](cancellation.h) |
| DeviceContext needed for stream-safe tensor copies | DeviceContext | [`device_base.h`](device_base.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added Code Commentary with Send/Recv semantics, ParsedKey, Args, dead tensors, cancellation, traps, and invariants.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
