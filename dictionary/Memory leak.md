---
description: Memory a program keeps allocated after it stops needing it, so usage climbs until the process slows or is killed.
---

A memory leak is memory that a program keeps allocated after it no longer needs it, so usage climbs until the process slows down or is killed. In manually managed languages such as C, it is memory that was never freed. In garbage-collected languages, the collector cannot release anything still referenced, so a leak is usually an object kept reachable by accident. Common causes are unbounded caches, global maps keyed by request or user, event listeners never removed, and threads blocked forever while holding references.

The usual symptom is that a restart fixes everything for a while. The memory [gauge](./Gauge.md) climbs steadily, the process is killed for exceeding its limit, it restarts, and the cycle repeats at a regular interval.

The shape of the memory graph helps separate a leak from normal behaviour:

| Pattern                         | Likely meaning                                |
| ------------------------------- | --------------------------------------------- |
| Sawtooth with a flat floor      | Normal garbage collection                     |
| Sawtooth with a rising floor    | Leak                                          |
| Rises, then plateaus            | A cache filling to its limit                  |
| Sudden step with a traffic burst | Large allocation, not necessarily a leak     |
| Process memory grows, heap flat | Native memory use or fragmentation            |

Small leaks add up under traffic. If each request retains 2 KB at a hypothetical 100 requests per second, the process leaks 200 KB per second, or 720 MB per hour. A container with a 2 GB limit and a 500 MB baseline has 1.5 GB of headroom and would be killed after about two hours.

To diagnose a leak, take two heap snapshots some time apart under load, compare which object types grew, and find what is holding references to them. Heap [profiling](./Profiling.md) supports this directly. A [soak test](./Soak%20test.md) can reproduce the problem before release, whereas frequent deployments can hide it by restarting the process.

_Usage:_

"The worker gets OOM-killed every six hours, like clockwork."

"Sounds like a memory leak. Grab two heap dumps an hour apart and diff them to see what's growing."
