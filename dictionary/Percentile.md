---
description: The value below which a given share of measurements fall. A p99 of 800 ms means 99% of requests took 800 ms or less.
---

A percentile is the value at or below which a given share of measurements fall. Latency is commonly reported as p50 (the median), p95 and p99. A p99 of 800 ms means 99% of requests took 800 ms or less and 1% took longer.

Percentiles are used because averages hide the shape of a distribution. Take 100 hypothetical requests, where 98 take 100 ms and 2 take 10,000 ms:

| Statistic | Value     |
| --------- | --------- |
| Mean      | 298 ms    |
| p50       | 100 ms    |
| p95       | 100 ms    |
| p99       | 10,000 ms |

The mean of 298 ms describes no real request. This is the situation behind a dashboard showing a healthy average while users report timeouts: the slow requests are real, but the average dilutes them.

Percentiles cannot be averaged. If one instance has a p99 of 100 ms and another has 900 ms, the combined p99 is not 500 ms. It depends on how much traffic each handled and on the full distributions. The same error occurs when per-minute p99s are averaged into an hourly figure. To get a correct result, merge the underlying [histograms](./Histogram.md) and calculate the percentile from the combined counts.

Sample size limits what a percentile can tell you. With 50 requests in a window, p99 is effectively the single slowest request, so the line jumps around at low traffic. Check the request count before reacting to a spike.

A high percentile also affects more users than its number suggests. If a page makes 20 independent requests, the chance that at least one is slower than p99 is 1 - 0.99^20, or about 18%. This effect is a main reason to care about [tail latency](./Tail%20latency.md), and why an [SLI](./SLI.md) for latency is usually defined by a threshold or percentile.

_Usage:_

"Average latency is 120 ms. I don't see what customers are complaining about."

"Look at p99, it's over four seconds. The average is hiding the slowest one in a hundred requests."
