---
description: The response time of the slowest small share of requests, read at high percentiles such as p99. Fan-out makes it more visible.
---

Tail latency is the response time of the slowest small fraction of requests, usually reported as p99 or p99.9. The name comes from the shape of a latency distribution: most requests form a hump near the typical value, and a long thin tail stretches to the right. A [percentile](./Percentile.md) is the standard way to measure it.

Fan-out makes the tail matter more than its size suggests. When one user request calls many backends in parallel and waits for all of them, it is as slow as the slowest. If each backend has a 1% chance of being slower than its p99, independently:

| Backends called | Chance at least one is slower than p99 |
| --------------- | -------------------------------------- |
| 1               | 1%                                     |
| 5               | 4.9%                                   |
| 20              | 18.2%                                  |
| 100             | 63.4%                                  |

The formula is 1 - 0.99^n.

The symptom is a product that is intermittently slow while every service's median looks healthy. The problem is hard to reproduce, because any single test request will probably be fast.

Common causes include garbage collection pauses, lock contention, [queueing delay](./Queueing%20delay.md), cache misses, retries, background work such as database compaction, and other workloads competing on shared hardware. A [distributed trace](./Distributed%20trace.md) of a slow request shows where the time went, and [profiling](./Profiling.md) can explain why.

Several techniques reduce the effect. Lower the fan-out, or return partial results when a deadline passes. Set timeouts based on observed latency. Use hedged requests, which send a second copy to another replica if the first has not replied after a short wait, and take whichever answers first. Hedging adds load, so limit how often it triggers.

Check the measurement too. Load tools affected by [coordinated omission](./Coordinated%20omission.md) can understate the tail by a wide margin.

_Usage:_

"Each service is under 50 ms at p50. Why does the page take two seconds for some users?"

"It fans out to 40 services. That's a tail latency problem, since one slow call holds up the whole response."
