# op_kernel.h

| Field | Value |
|---|---|
| repository | tensorflow |
| path | `tensorflow/core/framework/op_kernel.h` |
| doc_type | `file-level-onboarding` |
| lastUpdated | 2026-05-18T08:21 |
| lastVerifiedCommitHash | `2020b5919c5b66b8672438bed85d0ca88d434438` |
| lastVerifiedCommitDate | 2026-05-16 |
| governingOverview | [`tensorflow/core/overview.md`](../overview.md) |

## Governing Overview

Parent route-local overview: [`tensorflow/core/overview.md`](../overview.md)

## Purpose

Defines `OpKernel` (L140-L400) — the abstract base class for all operation implementations in TensorFlow. Every TF op (MatMul, Conv2D, Add, etc.) has one or more `OpKernel` subclasses registered per (op_type, device_type). The framework invokes `Compute()` (synchronous) or `ComputeAsync()` (asynchronous) with an `OpKernelContext` that provides inputs, allocates outputs, and manages resources. Also defines `OpKernelConstruction` (L70-L130) for kernel initialization, `OpKernelContext` (L410-L900) for per-invocation state, and `AsyncOpKernel` (L920-L960) for ops with deferred completion.

## Code Commentary

### Logic

**OpKernelConstruction (L70-L130):** Passed to the kernel constructor. Provides:
- `def()` — Returns the `OpDef` with input/output specs.
- `input_type(i)` / `output_type(i)` — Returns `DataType` for each input/output.
- `GetAttr(name, &value)` / `HasAttr(name)` — Attribute access for Attr() parameters.
- `device()` — The device this kernel is registered on.
- `env()` — `tsl::Env` for filesystem access during construction.

**OpKernel (L140-L400):** Abstract kernel base:
- Constructor takes `OpKernelConstruction*` for one-time setup.
- `Compute(OpKernelContext* context)` (L220) — Synchronous compute. Pure virtual — every kernel overrides this.
- `ComputeAsync(OpKernelContext* context, DoneCallback done)` (L230) — Async variant for GPU ops. Default falls back to Compute() and calls done. Override for stream-ordered execution.
- `IsExpensive()` — Hints to the executor whether this op is compute-heavy (affects scheduling).
- `def()` — Returns the OpDef this kernel was built from.
- `input_memory_types()` / `output_memory_types()` — Whether tensors are in HOST_MEMORY or DEVICE_MEMORY.
- `TraceString(op_ctx, verbose)` — Human-readable trace for logging, profiling, and debugging.

**OpKernelContext (L410-L900):** The per-invocation execution context:
- `input(i)` / `input_ref(i)` — Returns input tensors (or mutable ref). CHECK-fails on out-of-bounds.
- `num_inputs()` / `num_outputs()` — Tensor counts.
- `allocate_output(i, shape, &tensor)` — Allocates an output tensor. Returns Status. The framework owns the allocation.
- `allocate_temp(dtype, shape, &tensor)` — Allocates a temporary tensor (scratch memory).
- `SetStatus(status)` — Signals an error; the executor aborts the step.
- `eigen_cpu_device()` / `eigen_gpu_device()` — Returns Eigen thread-pool devices for Eigen-based ops.
- `op_device_context()` — Returns DeviceContext for stream-ordered GPU ops.
- `step_id()` — Process-wide unique step identifier.
- `cancellation_manager()` — Allows the kernel to abort on external cancellation.
- `ParallelFor(start, end, graint_size, fn)` — Parallelizes a loop body across the intra-op thread pool.
- `session_state()` / `tensor_store()` / `resource_manager()` — Shared per-step/per-session state.

**AsyncOpKernel (L920-L960):** Base for kernels that defer completion. `ComputeAsync()` stores state and calls the done callback later, typically when GPU work completes or data arrives over the network.

### Conventions

- **One kernel per (op_type, device_type)** — Multiple kernel instances exist for different devices; each is constructed once and Compute() is called many times.
- **Status-return, not exceptions** — Errors go through SetStatus() and Status returns. The framework propagates.
- **Output ownership** — Allocated outputs are owned by the framework. Kernels must not free or cache output pointers across invocations.
- **Input immutability** — Input tensors are read-only in Compute(). Mutating ref inputs must go through input_ref().

### Invariants And Boundaries

- OpKernelConstruction is valid only during the constructor. Accessing it afterward is undefined behavior.
- OpKernelContext is valid only during a single Compute() or ComputeAsync() call. Don't cache it.
- allocate_output() must be called in output-index order (0, 1, 2, ...). Out-of-order allocation fails.
- Each kernel instance handles exactly one (op_type, device_type) pair — kernel selection is done by the framework, not by the kernel itself.- **OpKernelContext::allocate_output() ownership** — Allocated output tensors are owned by the framework, not the kernel. Freeing or modifying them after Compute() returns is undefined behavior.
- **status() return is advisory** — Setting context->SetStatus(error) after a successful allocate_output() may leave partially populated outputs. Always set status BEFORE allocating outputs.
- **OpKernelConstruction::GetAttr fails silently** — Missing or wrong-type attributes cause errors during kernel instantiation, not compile time. Always check the returned Status.
- **resource_manager() is per-step** — OpKernelContext::resource_manager() is step-scoped. Resources stored there are freed after the step. Use OpKernelConstruction for persistent registration.
- **ParallelFor is cooperative** — OpKernelContext::ParallelFor spawns worker threads but the kernel caller owns cancellation. Not checking context->status() in each parallel iteration causes wasted compute.
- **DeviceContext lifecycle** — `context->eigen_gpu_device()` borrows the underlying CUDA stream. Don't cache it across Compute() calls; the stream may change between invocations.

### Todos

No known TODOs specific to this file.

## Docs References

No relevant domain documentation found. OpKernel is an internal C++ implementation detail not covered by public TF documentation.

## Repo-Internal References

OpKernel is the kernel-level API that every TF op implementation (C++ and Python custom ops) must conform to.

| Finding | Citations | Source Path |
|---|---|---|
| OpDef determines the input/output signature for each kernel | OpDef | [`op_def.proto`](op_def.proto) |
| Tensor input/output type for OpKernelContext | Tensor class | [`tensor.h`](tensor.h.md) |
| Device::Compute dispatches kernels with this interface | Device | [`device.h`](device.h.md) |
| Executor::RunAsync invokes OpKernel::Compute for each node | Executor | [`executor.h`](../common_runtime/executor.h.md) |
| OpRegistry maps op names to kernels registered on devices | OpRegistry | [`op.h`](op.h.md) |
| AsyncOpKernel for GPU/network ops with deferred completion | AsyncOpKernel | This file (L920-L960) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-18T08:21: Restructured to comply with C-05 file-level-onboarding template.
- 2026-05-17T17:30: Deep-improvement pass — added traps section for OpKernelContext ownership, Status ordering, resource manager scope, ParallelFor cancellation.
- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 1.
