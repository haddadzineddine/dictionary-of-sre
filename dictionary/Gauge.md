---
description: A metric that can rise or fall, reporting a current value such as memory in use, queue depth or open connections.
---

A gauge is a [metric](./Metric.md) that reports a current value, which can rise or fall. Memory in use, queue depth, open connections and temperature are typical gauges. Reading one tells you the level at that moment.

It differs from a [counter](./Counter.md) in several ways:

| Aspect        | Gauge                         | Counter                             |
| ------------- | ----------------------------- | ----------------------------------- |
| Direction     | Up and down                   | Up only, except on reset            |
| Read as       | Current value                 | Rate of change                      |
| Example       | 37 messages queued            | 1,204 messages processed            |
| Missed sample | That moment's value is lost   | Next sample still has the increments |

The weakness of a gauge is that it is only seen when sampled. A scraper recording every 15 seconds knows nothing about the values in between. A memory spike lasting three seconds may never appear on the graph even if it got the process killed, which produces a familiar confusion: an out-of-memory crash on a chart that shows plenty of headroom. Where available, record a peak value since the last scrape, and check the container's exit reason.

Aggregating gauges takes thought. Averaging memory across instances hides the one about to crash, so graph the maximum as well.

Gauges are the usual way to measure [saturation](./Saturation.md): connections in use against pool size, disk used, queue length. For resources that fill up, [alert](./Alert.md) on the trend. A prediction that a disk will fill within four hours is more useful than a fixed 90% threshold, which leaves 1 TB free on a 10 TB disk and 10 GB on a 100 GB disk.

Do not apply a counter function such as Prometheus `rate()` to a gauge. It reads every dip as a reset and returns wrong values. Use `deriv()` or `delta()` instead.

_Usage:_

"Memory never went above 70% on the dashboard, but the pod got OOM-killed."

"That gauge is scraped every 30 seconds, so it can miss a short spike. Check the peak from the container stats."
