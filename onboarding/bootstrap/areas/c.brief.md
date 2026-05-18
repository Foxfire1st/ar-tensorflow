# Area Brief — c

| Field | Value |
|---|---|
| repo | tensorflow |
| area | c |
| priority | HIGH |
| confidence | [HIGH] |

Stable C ABI wrapping core TF. Foundation for all non-Python language bindings.

**3 files onboarded.** Key pattern: thin C wrappers around C++ Session/Graph/Tensor, error via TF_Status, opaque handle types.

**Traps:** ABI versioning, memory ownership (caller-controlled deallocator), opaque handles hide internal changes.
