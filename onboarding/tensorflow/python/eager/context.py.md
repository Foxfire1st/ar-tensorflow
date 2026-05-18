# context.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/eager/context.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

Defines the `Context` singleton — the process-global environment for eager execution in TensorFlow. The Context manages devices, execution mode (sync/async), device placement policy, memory growth, thread pools, distributed coordination, and function execution caching. It is the Python-side counterpart to the C++ `EagerContext`.

Every eager operation dispatch, `tf.function` trace, and device configuration goes through this singleton. Key helpers like `context()` (get current context), `executing_eagerly()`, and `num_gpus()` are exported from here.

Also defines supporting types: `PhysicalDevice`, `LogicalDevice`, `LogicalDeviceConfiguration`, `FunctionCallOptions`, and `_EagerDeviceContext`.

## Code Commentary

### Logic

**`Context` (L519-L2420):** The central singleton class (~2000 lines). Key state and methods:

- **Execution mode** — `executing_eagerly()` returns whether ops are evaluated immediately (`SYNC`/`ASYNC`) vs building a graph. TF2 defaults to `EAGER_MODE`.
- **Device management:**
  - `list_physical_devices(device_type)` → `PhysicalDevice` list. Physical devices represent actual hardware.
  - `list_logical_devices()` → `LogicalDevice` list. One physical GPU can host multiple logical devices (virtual GPUs).
  - `set_visible_devices(devices, device_type)` — restricts which physical devices TF can see.
  - `set_logical_device_configuration(device, config_list)` — partitions a physical GPU into logical devices with memory limits.
  - `get_memory_growth(device)` / `set_memory_growth(device, enable)` — GPU memory growth (allocate on demand vs all at once).
- **Device placement policy:**
  - `DEVICE_PLACEMENT_EXPLICIT` — error if op inputs aren't on the right device.
  - `DEVICE_PLACEMENT_WARN` — warn + silently copy.
  - `DEVICE_PLACEMENT_SILENT` — silently copy (default).
  - `DEVICE_PLACEMENT_SILENT_FOR_INT32` — silently copy only int32 tensors.
- **Threading:**
  - `intra_op_parallelism_threads` / `inter_op_parallelism_threads` — controls thread pool sizes.
- **Function execution:**
  - `add_function(fdef)` — registers a `FunctionDef` for eager execution.
  - `has_function(fname)` — checks if a function is registered.
  - `enable_run_metadata()` / `export_run_metadata()` — per-step profiling/tracing.
- **Global flags:**
  - `optimizer_jit` — when True, run computations through XLA JIT.
  - `soft_device_placement` — when True, silently place GPU ops on CPU if GPU is busy.
  - `log_device_placement` — when True, log every op→device assignment.
  - `enable_mlir_bridge` — route computations through MLIR-based compiler.
- **Distributed execution:**
  - `server_def` — `ServerDef` proto for gRPC distributed setup.
  - `update_server_def(server_def)` — reconfigures the distributed environment.
  - `collective_ops` configuration — all-reduce/broadcast settings for multi-worker training.
- **Graph optimization:** `optimizer_jit`, MLIR bridge, auto-mixed-precision, layout optimizer.

**`PhysicalDevice` (L458):** Named tuple of `(name, device_type)`. Represents a real hardware device.

**`LogicalDevice` (L392):** Named tuple of `(name, device_type)`. Represents a virtual device (e.g., one partition of a physical GPU).

**`LogicalDeviceConfiguration` (L413):** Named tuple of `(memory_limit, experimental_priority, experimental_device_ordinal)`. Parameters for partitioning a physical GPU.

**`FunctionCallOptions` (L244):** Configuration for `tf.function` calls — specifies executor type, config proto, and whether to use XLA.

### Conventions

- **Singleton pattern** — `CONTEXT` is the module-level global; `context()` and `context_safe()` provide access.
- **Lazy initialization** — hardware devices are enumerated on first use (`ensure_initialized()`). The `_initialize_lock` prevents races.
- **Module-level getters** — `executing_eagerly()`, `num_gpus()`, `graph_mode()` are thin wrappers around `context()`.

### Invariants And Boundaries

- One Context per process — creating a second Context raises an error.
- ensure_initialized() completes exactly once; devices, thread pools, and function caches are fixed afterward.
- Device placement policy is global — all eager ops share it unless overridden by tf.device().
- Thread-local execution mode: _ContextSwitchStack allows temporary eager↔graph mode switches.
- Memory growth must be set before first GPU allocation; changing it after allocation is ignored.

- **One Context per process** — Creating a new `Context` when one already exists raises an error.
- **Initialization is once** — Once `ensure_initialized()` completes, devices, thread pools, and function caches are fixed for the process lifetime.
- **Device placement policy is global** — All eager ops share the same placement policy. Per-op overrides must use `tf.device()`.
- **Thread-local execution mode** — The `_ContextSwitchStack` allows temporary mode switches (e.g., `tf.function` tracing switches from eager to graph mode).
- **Memory growth must be set before first allocation** — Once a GPU has allocated memory, memory growth cannot be changed for that device.

- **Context is process-global** — Multiple test cases running in the same process share the Context. Tests that modify `set_visible_devices()` or `set_logical_device_configuration()` must reset state between cases.
- **device() context manager is stack-based** — `tf.device('/GPU:0')` pushes onto a stack; forgetting to exit the context affects all subsequent ops in the thread.
- **async execution hides errors** — In ASYNC mode, op errors may surface in a later op's execution, making debugging confusing. Use `DEBUG` mode or `context().sync_executors()` to flush.
- **Memory growth is GPU-only** — Setting memory growth on CPU devices is silently ignored.
- **server_def reconfiguration is global** — Calling `update_server_def()` changes the distributed environment for ALL threads. Existing in-flight ops may fail.
- **add_function races** — If multiple threads call add_function concurrently, function registration may fail non-deterministically.
- **Physical GPU enumeration happens once** — New GPUs attached after initialization are invisible. Restart the process.


### Todos

- Move tensor caches to C++ (L525, `TODO(iga)`).

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| GPU configuration guide (memory growth, virtual devices) | — | https://www.tensorflow.org/guide/gpu |
| Distributed training with TF_CONFIG and collective ops | — | https://www.tensorflow.org/guide/distributed_training |

## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| pywrap_tfe.EagerContext wraps the C++ eager runtime context | C++ eager context | `tensorflow/c/eager/c_api.h` |
| execute.py uses Context for eager op dispatch | execute.quick_execute | `tensorflow/python/eager/execute.py` |
| tf.function switches context mode during tracing | def_function.py | [`tensorflow/python/eager/def_function.py`](../tensorflow/python/eager/def_function.py.md) |
| config.py re-exports most Context methods as public tf.config.* API | set_visible_devices, list_physical_devices, etc. | `tensorflow/python/framework/config.py` |
| LogicalDevice wraps C++ TFE_Device | pywrap_tfe | `tensorflow/python/eager/pywrap_tfe.cc` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
