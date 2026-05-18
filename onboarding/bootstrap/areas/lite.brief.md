# Area Brief — lite

| Field | Value |
|---|---|
| repo | tensorflow |
| area | lite |
| priority | HIGH |
| confidence | [HIGH] |

Mobile/embedded ML runtime. FlatBuffer models, static memory arena, op resolution via OpResolver, hardware acceleration via delegates.

**8 load-bearing files** — all onboarded. Key flow: FlatBufferModel → InterpreterBuilder + OpResolver → Interpreter::AllocateTensors() → Interpreter::Invoke().

**Traps:** Fixed arena size, op resolver misses, delegate compatibility, micro/ separate codebase.
