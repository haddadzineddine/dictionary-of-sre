---
description: A queueing rule. The average number of items in a system equals the arrival rate times the average time each item spends there.
---

Little's Law states that the average number of items in a stable system equals the average arrival rate multiplied by the average time each item spends in the system. John Little published a proof in 1961.

```text
L = λ × W
```

L is the average number in the system, λ (lambda) is the arrival rate, and W is the average time in the system. The units must match, so convert milliseconds to seconds when the rate is per second. The law holds whatever the arrival pattern or distribution of service times, as long as the system is stable over the period measured.

It answers common sizing questions, shown here with hypothetical numbers:

| Question                                         | Calculation | Answer             |
| ------------------------------------------------ | ----------- | ------------------ |
| Requests in flight at 200 per second and 250 ms? | 200 x 0.25  | 50 concurrent      |
| Maximum rate for 100 connections held 50 ms each? | 100 / 0.05 | 2,000 per second   |

The law explains a confusing kind of outage, where a slow dependency exhausts threads in a service that has no bug. At 200 requests per second and 250 ms, a service holds 50 requests in flight. If a dependency slows and requests now take one second, the same traffic needs 200 in flight. With a pool of 100 threads, requests begin to queue or fail although traffic never changed.

The law describes averages, so it says nothing about [percentiles](./Percentile.md) or short bursts, and sizing needs headroom on top. It does not apply while arrivals exceed completions, because the queue keeps growing and no steady average exists. Related limits appear as [saturation](./Saturation.md) and [queueing delay](./Queueing%20delay.md).

_Usage:_

"How big should the connection pool be for the new service?"

"Little's Law says 500 queries a second at 20 ms is 10 in flight on average. Start around 20 for bursts and watch wait time."
