---
description: Measuring where a running program spends CPU, memory or waiting time, down to individual functions.
---

Profiling is measuring where a running program spends resources, such as CPU time, memory or waiting time, down to the level of individual functions. It replaces guesses about performance with data. The familiar failure it prevents is spending two days optimising a function that turns out to account for 1% of runtime, where even a perfect fix could save at most 1%.

Different profile types answer different questions:

| Type       | Question                      | Useful when                           |
| ---------- | ----------------------------- | ------------------------------------- |
| CPU        | Which code is on the CPU?     | High CPU or slow computation          |
| Heap       | What is holding memory now?   | Investigating a [memory leak](./Memory%20leak.md) |
| Allocation | What allocates the most?      | Heavy garbage collection              |
| Off-CPU    | Where are threads blocked?    | Slow service with low CPU             |

Profilers work in two main ways. A [sampling](./Sampling.md) profiler interrupts the program at a fixed frequency, such as 99 times per second, and records the call stack. Its overhead is low, and functions that use a lot of time appear in many samples. An instrumenting profiler records every function entry and exit, which is exact but slower and can distort the behaviour being measured. Sampling is the usual choice in production.

A CPU profile does not show waiting. A request that uses 10 ms of CPU and blocks for 900 ms on a database barely appears in it, which is why a slow service can produce a CPU profile with nothing interesting in it. Use an off-CPU profile or a [distributed trace](./Distributed%20trace.md) for that case.

Continuous profiling runs low-overhead sampling across production all the time. It lets you compare before and after a deployment, or inspect the moment of an [incident](./Incident.md) without reproducing it. Results are commonly displayed as a [flame graph](./Flame%20graph.md).

_Usage:_

"I'm going to rewrite the JSON parser, it has to be what's slow."

"Profile it first. Last time we guessed, the parser was 2% of CPU and the real cost was in logging."
