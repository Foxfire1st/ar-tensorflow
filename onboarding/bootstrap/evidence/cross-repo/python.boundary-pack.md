# Cross-Repo Boundary Pack — python

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/python/` |
| generated | 2026-05-17T17:00 |
| topology | external |
| status | no-boundary-found |

## Allowed Adjacent Repos

| Adjacent Repo | Expected Branch | Actual Branch | Status | Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Boundary Summary

No cross-repo boundaries configured. The Python API communicates with the C++ runtime via the pywrap bridge (same repo, Python → C++ boundary). Keras within TF is a fork of the standalone `keras-team/keras` repository.

## Vendored Dependencies

| Dependency | Location | Role | Interface |
|---|---|---|---|
| Pywrap bridge | `tensorflow/python/pywrap_*.cc` | Python↔C++ tensor/op marshalling via pybind11 | `pywrap_tfe`, `pywrap_tensorflow` modules |
| Keras (integrated) | `tensorflow/python/keras/` | High-level modeling (forked from keras-team/keras) | TF-specific: `tf.keras` namespace |

## Same-Repo Facts That Must Not Be Classified As Cross-Repo

| Fact | Correct Bucket |
|---|---|
| Python calling into C++ via pywrap | Repo-Internal (same repo, same build) |
| Keras as integrated in TF | Repo-Internal (TF fork, not cross-repo) |
| numpy/tf.experimental.numpy interop | Repo-Internal (TF's numpy implementation) |

## Boundary Risks

- None at the cross-repo level.

## Needs Developer Confirmation

- Should `keras-team/keras` be configured as an adjacent repo for tracking divergence?
