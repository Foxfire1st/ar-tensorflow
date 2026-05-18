# __init__.py

| Field | Value |
|---|---|
| repository | tensorflow |
| doc_type | `file-level-onboarding` |
| path | `tensorflow/python/__init__.py` |
| governingOverview | [`tensorflow/python/overview.md`](../overview.md) |
| lastUpdated | 2026-05-17T16:00 |
| lastVerifiedCommitHash | 2020b5919c5b66b8672438bed85d0ca88d434438 |
| lastVerifiedCommitDate | 2026-05-16 |

## Governing Overview

Parent route-local overview: [`tensorflow/python/overview.md`](../overview.md)

## Purpose

The top-level `tensorflow` package init file. This is the entry point that runs whenever `import tensorflow` is executed. It deliberately contains **minimal code** — no heavy imports — to keep import time fast.

Its primary responsibility is exposing package-level dunders (`__version__`, `__git_version__`, `__compiler_version__`, `__cxx11_abi_flag__`, `__monolithic_build__`) while hiding all other private (`_`-prefixed) symbols.

## Code Commentary

### Logic

The file follows a simple pattern:
1. Define `_exported_dunders` — a set of special `__dunder__` names that should be public.
2. Compute `__all__` as all names in the module's namespace that either are in `_exported_dunders` or do not start with `_`.

The comment at the top warns: "Do not add code to this file. This file is imported whenever TensorFlow is imported. Additional imports could cause import time to increase by multiple seconds."

### Conventions

- **Absolute minimal init** — no imports beyond what's inherently available in the package namespace.
- **Dunders-only export** — `__all__` gates what `from tensorflow import *` exposes.

### Invariants And Boundaries

- This is a re-export shim — no new ops, classes, or logic are defined here.
- Circular import avoidance: imports are ordered to prevent cycle errors. Changing import order can break startup.
- __all__ controls rom tensorflow import *. Missing entries mean APIs are silently inaccessible.

- This file must remain lightweight. Any new import or logic here multiplies across every `import tensorflow` call in every TF program.
- Package version/build metadata is set elsewhere (C++ extension modules) and exposed as top-level dunders here.
- **Lazy imports break IDE autocompletion** — Many 	ensorflow.X APIs are imported lazily via LazyLoader (or similar). Static analysis tools see ModuleType('tensorflow.X') instead of the actual API surface.
- **Import side effects** — Importing 	ensorflow triggers eager context initialization, GPU enumeration, and logging backend selection. This takes 1-5 seconds and cannot be avoided.
- **Circular import sensitive** — Reordering top-level imports can cause circular import errors that only reproduce on cold starts. Always test import changes on a fresh process.


### Todos

No known TODOs.

## Docs References

| Finding | Citations | Source Path |
|---|---|---|
| TF version and build constants are documented in the install guide | — | https://www.tensorflow.org/install |


## Repo-Internal References

| Finding | Citations | Source Path |
|---|---|---|
| Actual version string is set by the C++ pywrap extension, not in this Python file | version injection | `tensorflow/python/framework/versions.py` |
| The heavy lifting of public API assembly happens in api_template and lazy_loader | API generation | `tensorflow/python/util/lazy_loader.py`, `tensorflow/tools/api/generator/api_gen.bzl` |

## Cross-Repo References

No meaningful cross-repo references found.

## Update History

- 2026-05-17: Initial file-level onboarding from full-bootstrap Wave 2
