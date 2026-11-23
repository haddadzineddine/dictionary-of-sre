---
description: Pushing a system past its expected capacity to find where it breaks, how it fails and whether it recovers.
---

Stress testing is pushing a system beyond its expected capacity to find where it breaks, how it fails and whether it recovers. [Load testing](./Load%20testing.md) asks whether a system meets its targets at expected demand.

A stress test usually runs in phases:

| Phase    | Action                               | Record                                   |
| -------- | ------------------------------------ | ---------------------------------------- |
| Baseline | Run at normal load                   | Latency and error rate                   |
| Ramp     | Raise load in steps, holding each    | The point where the [SLO](./SLO.md) fails |
| Break    | Continue until errors dominate       | First resource to reach [saturation](./Saturation.md) |
| Recover  | Drop back to normal load             | Time to recover, or failure to recover   |

The first useful output is the knee: the load at which latency starts rising sharply because [queueing delay](./Queueing%20delay.md) takes over. Plan capacity below the knee, not at the point of collapse.

How the system fails matters as much as where. In a graceful failure, it rejects excess requests quickly and keeps serving what it can. In a collapse, it accepts everything, queues grow, timeouts trigger retries, and it spends its time on requests whose clients have already given up. Throughput then falls as load rises.

The recovery phase catches a difficult case known as a metastable failure, where the system stays broken after the original trigger has gone. Retries or cold caches keep load above what it can handle. In production this appears as a traffic spike that has passed while the service stays down until someone restarts it or blocks traffic. A [thundering herd](./Thundering%20herd.md) is a frequent cause.

Findings usually lead to load shedding (rejecting excess work on purpose), bounded queues, timeouts, and retry budgets that cap retries as a share of requests.

Run stress tests in an isolated environment, and confirm that each [alert](./Alert.md) fires at the expected point.

_Usage:_

"We know it handles 8,000 requests a second. What happens at 12,000?"

"Nobody knows. Let's stress test it in staging, then drop the load and see if it recovers without a restart."
