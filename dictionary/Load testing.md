---
description: Sending realistic traffic to a system at expected volumes to check that it meets its performance targets.
---

Load testing is sending realistic traffic to a system at expected volumes to check whether it meets its performance targets. A typical question is whether the service can handle next month's peak with p99 latency under 300 ms.

It belongs to a family of tests that ask different questions:

| Test                                | Question                          | Load shape                 | Typical duration   |
| ----------------------------------- | --------------------------------- | -------------------------- | ------------------ |
| Load test                           | Does it meet targets at expected load? | Ramp to peak and hold | Minutes to an hour |
| [Stress test](./Stress%20testing.md) | Where and how does it break?     | Increase past capacity     | Until failure      |
| [Soak test](./Soak%20test.md)       | Does it degrade over time?        | Steady and moderate        | Hours to days      |

The result is only as good as the realism of the test. The common symptom is a system that passes at 5,000 requests per second and falls over in production at 2,000. Usual reasons are simulated users all requesting the same cached URL, a test database far smaller than production, missing background jobs, and an environment that differs from production.

Derive the traffic mix from production [logs](./Log.md). Use data volumes close to production. Ramp up gradually and hold long enough to get past cold caches. Report [percentiles](./Percentile.md) and error rates, not averages.

Watch the load generator too. If it runs out of CPU, the latencies it reports are wrong. Prefer an open workload, which sends requests at a fixed arrival rate whatever the responses do. Tools that wait for each reply slow down along with the server and hide the delay, an error called [coordinated omission](./Coordinated%20omission.md).

[Little's Law](./Little's%20Law.md) gives a quick sanity check: 50 clients that each wait for a reply at 200 ms average latency cannot exceed 250 requests per second.

Results feed directly into [capacity planning](./Capacity%20planning.md).

_Usage:_

"The load test passed at twice our peak, so Black Friday is safe."

"It also hit one product page with an empty cart table. Replay real traffic against a production-sized database before we trust that."
