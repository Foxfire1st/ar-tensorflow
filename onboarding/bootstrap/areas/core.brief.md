# Area Brief — core

| Field | Value |
|---|---|
| repo | tensorflow |
| area | core |
| priority | CRITICAL |
| confidence | [HIGH] |

Core is the C++ runtime foundation: Graph DAG, Executor dispatch loop, Device abstraction, OpKernel contract, Session API, and distributed execution. Everything above it (Python API, bindings, compiler, Lite) either calls into it or reimplements a subset.

**12 load-bearing files** — all onboarded. Key pipeline: `Session::Run()` → `Executor::RunAsync()` → ready ops dispatched to `OpKernel::Compute()` on assigned `Device`.

**Cross-cutting:** Device placement (Placer), tensor passing (Rendezvous), memory management (BFCAllocator), function compilation (FunctionLibraryRuntime).

**Traps:** Thread safety (concurrent Compute), BFCAllocator fragmentation, Placer silent failures, GraphDef version compatibility.
