# xla_device.h

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/compiler/jit/xla_device.h` |
| governingOverview | [`tensorflow/compiler/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/compiler/overview.md`](../overview.md)

## Purpose

Defines `XlaDevice` — a `LocalDevice` subclass that executes TF graph ops via XLA compilation rather than the normal TF executor. Ops assigned to an `XlaDevice` are compiled into XLA computations; tensors on the device are thin wrappers around `XLA ScopedShapedBuffers`.

Each XLA backend (CPU, GPU, TPU) instantiates its own `XlaDevice` under a distinct name (e.g., `XLA_CPU`, `XLA_GPU`). Used both for implicit JIT compilation and explicit `tf.device("XLA_CPU")` placement.

## Code Commentary

### Logic

**`XlaDevice`** inherits from `LocalDevice`:
- **`PaddedShapeFn`** — function type: given a `Tensor`, returns the fully-padded `xla::Shape` of its representation on device.
- **`Metadata`** — nested class storing device metadata (platform, device ordinal, `PaddedShapeFn`, `XlaOpRegistry::DeviceRegistration`). Retrievable via `GetMetadata()`.
- **`Compute(OpKernel*, OpKernelContext*)`** — overrides `Device::Compute()`. Dispatches ops through XLA-compiled paths rather than the default `op_kernel->Compute()`.
- **Compilation integration** — uses `XlaCompiler`, `DeviceCompiler`, and `XlaOpRegistry` to compile clusters on-the-fly.
- **Tensor representation** — `XlaTensor` wraps `ScopedShapedBuffer`; tensors are stored in XLA-internal format on the device.

### Conventions

- **Thin device abstraction** — `XlaDevice` looks like any other `Device` to the rest of TF, but internally routes computation through XLA.
- **Per-backend instantiation** — separate `XlaDevice` instances for CPU, GPU, TPU.

### Invariants And Boundaries

- XlaDevice is not a real compute device — it's a placeholder that triggers XLA compilation.
- Only ops marked for compilation execute on XlaDevice; non-marked ops run on the backing device.
- XlaDevice owns its DeviceCompiler; destroying the device clears the compilation cache.

- **All ops on XlaDevice go through XLA** — there is no fallback to the TF executor within this device.
- **XLA tensors are not standard TF tensors** — `XlaTensor` carries `ScopedShapedBuffer`, not raw memory. Cross-device copies between XLA and non-XLA devices require format conversion.
- **Shape must be known** — XLA requires static shapes for compilation.
- **XlaDevice is invisible to tf.config** — It doesn't appear in list_physical_devices(). Users can't manually place ops on it; placement is handled by the mark-for-compilation pass.
- **Backing device determines memory** — XlaDevice borrows memory from its backing device (CPU/GPU). Running out of backing device memory while XLA is compiling crashes the process.
- **Metadata propagation through XLA** — After compilation, node metadata (debug names, shapes) may be lost because HLO doesn't preserve all TF metadata. Debugging errors in XLA-compiled code is harder.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| Using XLA devices explicitly with tf.device | — | https://www.tensorflow.org/xla |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| LocalDevice is the base class (adds local execution semantics) | LocalDevice class | `tensorflow/core/common_runtime/local_device.h` |
| XlaTensor wraps ScopedShapedBuffer for XLA device tensors | XlaTensor class | `tensorflow/compiler/jit/xla_tensor.h` |
| XlaCompiler compiles ops assigned to XlaDevice | XlaCompiler | [`tensorflow/compiler/tf2xla/xla_compiler.h`](../tensorflow/compiler/tf2xla/xla_compiler.h.md) |
| DeviceCompiler caches compiled results per cluster | DeviceCompiler | [`tensorflow/compiler/jit/device_compiler.h`](../tensorflow/compiler/jit/device_compiler.h.md) |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 3
