# Area Brief — python

| Field | Value |
|---|---|
| repo | tensorflow |
| area | python |
| priority | CRITICAL |
| confidence | [HIGH] |

Primary user-facing API layer. Eager execution by default, tf.function for graph compilation, GradientTape for autodiff, Keras for high-level modeling, tf.data for input pipelines.

**10 load-bearing files** — all onboarded. Key flow: `tf.Variable` + `tf.GradientTape` → forward pass → `tape.gradient()` → optimizer. Graph path: `@tf.function` traces to `ConcreteFunction` → `Graph.executing_eagerly() == False`.

**Traps:** Tracing overhead on first call, eager/graph semantic gaps, shape polymorphism causing retraces, Keras→C++ round-trip opacity.
