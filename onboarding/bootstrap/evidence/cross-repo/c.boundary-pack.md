# Cross-Repo Boundary Pack — c

| Field | Value |
|---|---|
| repo | tensorflow |
| areaOrRoute | `tensorflow/c/` |
| generated | 2026-05-17T17:00 |
| topology | external |
| status | no-boundary-found |

## Allowed Adjacent Repos

| Adjacent Repo | Expected Branch | Actual Branch | Status | Use |
|---|---|---|---|---|
| (none configured) | — | — | — | — |

## Boundary Summary

No cross-repo boundaries configured. The C API wraps core TF internally (`c_api.cc` → `core/common_runtime/session.cc`). It is consumed externally by Go, Java, JS, and Swift bindings, but these are in the same repo.

## Same-Repo Facts That Must Not Be Classified As Cross-Repo

| Fact | Correct Bucket |
|---|---|
| c_api.cc calling core::Session::Run() | Repo-Internal |
| Go bindings calling c_api.h | Repo-Internal (go/ in same checkout) |
| Java bindings calling c_api.h | Repo-Internal (java/ in same checkout) |
| Swift bindings calling c_api.h | Repo-Internal (swift/ in same checkout) |

## Boundary Risks

- None at the cross-repo level for the C API itself. The C API's external consumers (other language ecosystems, third-party bindings) rely on ABI stability but are not in-scope repos.

## Needs Developer Confirmation

- Should any language binding consumers be tracked as adjacent repos?
