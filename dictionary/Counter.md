---
description: A metric that only increases, such as total requests served, and is read as a rate of change over time.
---

A counter is a [metric](./Metric.md) whose value only increases, apart from resetting to zero when the process restarts. Requests served, errors returned and bytes sent are typical counters.

The raw value is rarely useful, because "1,204,331 requests since start" says little about the present. Counters are read as a rate: take the change over a window and divide by the window's length. A counter that moves from 1,000 to 1,600 over 60 seconds is running at 10 per second.

Restarts need handling, as this hypothetical series shows:

| Time | Raw value | What happened                 |
| ---- | --------- | ----------------------------- |
| 0 s  | 1,000     | Normal                        |
| 15 s | 1,150     | 150 new events                |
| 30 s | 40        | Process restarted, counted 40 |
| 45 s | 190       | 150 new events                |

Plain subtraction between 15 s and 30 s gives -1,110, which appears on a graph as a sharp negative spike. Query functions built for counters, such as `rate()` in Prometheus, treat any drop as a reset and count the 40 as new events. Increments made between the last scrape and the crash are still lost.

A counter tolerates missed scrapes better than a [gauge](./Gauge.md) reporting a current rate. If one sample is lost, the next still includes every increment since, so the total over the window stays correct. This is why event counts are stored as counters.

Counters have a less obvious failure with new label values. If a series for `status="500"` does not exist until the first error, there is no earlier sample to compare it with, and `rate()` misses that first increment. An [alert](./Alert.md) on rare errors may then stay silent. Initialise known label combinations to zero at startup.

_Usage:_

"The error graph dropped to minus a thousand at 14:05."

"Someone graphed a raw delta on a counter and the pod restarted. Use rate so resets are handled."
