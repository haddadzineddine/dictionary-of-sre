---
description: A metric that counts observations into value buckets, so percentiles can be estimated and combined across instances.
---

A histogram is a [metric](./Metric.md) that counts observations into buckets by value. Request duration is the usual example: each request increments the bucket covering how long it took. Keeping counts instead of every value lets you estimate a [percentile](./Percentile.md) at a fixed storage cost.

Here is a hypothetical histogram of 1,000 requests, using cumulative buckets as Prometheus does:

| Bucket (at or under) | Cumulative count |
| -------------------- | ---------------- |
| 100 ms               | 900              |
| 250 ms               | 980              |
| 500 ms               | 995              |
| 1,000 ms             | 1,000            |

To estimate p99, find request number 990, which falls in the 250 to 500 ms bucket. Linear interpolation gives 250 + ((990 - 980) / (995 - 980)) x 250, or about 417 ms. The true value could be anywhere from 250 to 500 ms, so bucket boundaries set the accuracy.

Poorly chosen buckets produce a recognisable symptom. If more than 1% of requests exceed the highest finite boundary, the reported p99 flatlines at that boundary, even as real latency keeps climbing. Choose boundaries that cover both normal and failing behaviour, and place one at your [SLO](./SLO.md) threshold so the count of good requests is exact.

Histograms can be combined across instances by adding counts bucket by bucket, provided the boundaries match. Percentiles already calculated on each instance cannot be merged in this way.

Each bucket is its own time series, multiplied by every label combination, which makes histograms a common source of [cardinality](./Cardinality.md) growth. In the classic Prometheus format, 12 buckets across 1,000 label combinations produce 12,000 series, plus 2,000 more for each combination's sum and count. Native Prometheus histograms and [OpenTelemetry](./OpenTelemetry.md) exponential histograms choose their buckets automatically, which reduces this cost.

_Usage:_

"p99 has sat at exactly 10 seconds since the [incident](./Incident.md) started. Suspicious, right?"

"10 is our top histogram bucket. Everything slower lands in the overflow bucket, so real latency could be much higher."
