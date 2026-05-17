# Tools

## Checks

No repo-specific checks configured yet.

Add test, lint, typecheck, build, and smoke-check commands for the tensorflow repo.

## Commands

No repo-specific commands configured yet.

## Runtime Notes

- TensorFlow is built with Bazel. The build system is complex; consult `configure.py` for environment setup.
- Python package lives under `tensorflow/python/`.
- C/C++ core under `tensorflow/core/`, `tensorflow/c/`, `tensorflow/cc/`.
- The compiler stack (XLA) lives under `tensorflow/compiler/`.
- Lite runtime under `tensorflow/lite/`.
