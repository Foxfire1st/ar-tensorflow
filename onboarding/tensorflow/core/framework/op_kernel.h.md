# op_kernel.h

| Field | Value |
|---|---|
| repository | tensorflow |
| sourceFile | `tensorflow/core/framework/op_kernel.h` |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T00:00 |
| lastVerifiedCommitHash | |
| lastVerifiedCommitDate | |

## Purpose

Defines `OpKernel` — the base class for all TensorFlow operation implementations. Every op (Add, MatMul, Conv2D, etc.) is a subclass of `OpKernel` that implements the `Compute(OpKernelContext*)` method. This is the **fundamental execution contract** in TensorFlow: the Executor calls `Compute()` for each node, passing an `OpKernelContext` that provides input tensors, output allocation, and device resources.

Also defines `AsyncOpKernel` for operations that may block (I/O, RPC, queue operations) and the kernel registration infrastructure (`KernelDef`, `KernelDefBuilder`).

## Code Commentary

### OpKernel (L107+)

The base class:
- **Constructor** — takes `OpKernelConstruction*` context for reading attributes, allocating persistent tensors, checking input/output count and types.
- **Compute(OpKernelContext* ctx)** — the core contract. Called by the Executor when all inputs are ready. Must:
  - Read inputs via `ctx->input(i)`
  - Allocate outputs via `ctx->allocate_output(i, shape)`
  - Compute results into output tensors
  - Return (synchronous; all work complete on return)
- **IsExpensive()** — hint for scheduling; expensive ops may be scheduled differently.
- **MatchSignature()** — validates that input/output types match the NodeDef.

### AsyncOpKernel (L150+)

For operations that must not block the thread pool:
- **ComputeAsync(OpKernelContext* ctx, DoneCallback done)** — starts async work, calls `done()` when complete. The Executor does not proceed past this node until `done()` is called.
- Used by: Send/Recv ops, queue ops, RPC ops, TPU ops.

### OpKernelContext (L200+)

The execution context passed to Compute():
- **Input access** — `input(i)`, `num_inputs()`, `input_dtype(i)`
- **Output allocation** — `allocate_output(i, shape)`, `allocate_temp(dtype, shape)`
- **Resource management** — `resource_manager()`, `step_container()`
- **Device access** — `device()`, `op_device_context()`
- **Cancellation** — `cancellation_manager()`
- **Parallelism** — `runner()` for scheduling work on the device thread pool

### KernelDef and Registration

- **KernelDef** — protobuf describing an op kernel: op name, device type, constraint, label, priority.
- **KernelDefBuilder** — fluent API for building KernelDef: `.Name("Add").Device(DEVICE_CPU)`.
- **REGISTER_KERNEL_BUILDER** macro — registers a kernel factory for (op, device) pair.

## Key Invariants

- `Compute()` must be **thread-safe** — may be called concurrently from different Executors.
- `Compute()` must **not block** — if blocking is needed, use `AsyncOpKernel`.
- Outputs must be allocated via `ctx->allocate_output()` — not from external allocators.
- Input tensors are valid only during `Compute()` — do not hold references across calls.
- `done()` callback in AsyncOpKernel must be called exactly once.

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Executor calls OpKernel::Compute() for each node | executor.cc dispatch loop | `tensorflow/core/common_runtime/executor.cc` |
| Op registration: REGISTER_KERNEL_BUILDER maps to OpKernel subclass | kernel_def_builder.h | `tensorflow/core/framework/kernel_def_builder.h` |
| All 1000+ kernels are OpKernel subclasses | kernels/ directory | `tensorflow/core/kernels/` |
| Device::Compute() wraps OpKernel execution | device.h | `tensorflow/core/framework/device.h` |

## Cross-Repo References

No relevant cross-repo evidence found.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Creating custom ops in C++ | — | https://www.tensorflow.org/guide/create_op |

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap
