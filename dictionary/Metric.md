---
description: A numeric measurement recorded over time, cheap to store and query, used for dashboards, trends and alerts.
---

A metric is a numeric measurement of a system recorded over time. Each metric has a name and a set of labels, and every distinct combination of label values forms its own time series of timestamped numbers:

```text
http_requests_total{method="GET", route="/checkout", status="500"} 1027
```

Three metric types cover most needs:

| Type                         | Behaviour                                | Example           |
| ---------------------------- | ---------------------------------------- | ----------------- |
| [Counter](./Counter.md)      | Only goes up, resets on restart          | Requests served   |
| [Gauge](./Gauge.md)          | Goes up and down                         | Memory in use     |
| [Histogram](./Histogram.md)  | Counts values into buckets               | Request duration  |

Metrics are cheap because storage grows with the number of time series and not with traffic. A counter takes the same space at 10 requests per second as at 10,000. That makes metrics suitable for [alerts](./Alert.md) and for trends over months.

The saving comes from discarding detail. A metric can report that 2% of requests failed, but not which requests. A breakdown is only possible if the label already exists. A common symptom is an [incident](./Incident.md) affecting one large customer that no graph can show, because customer is not a label. Adding customer ID as a label is usually the wrong fix because of [cardinality](./Cardinality.md). Per-request detail belongs in [logs](./Log.md) or [distributed traces](./Distributed%20trace.md).

Metrics reach storage by being pulled or pushed. Prometheus, for example, scrapes an endpoint on an interval, while other systems receive pushed data. Either way, resolution limits what you can see: a value sampled every 60 seconds cannot show a 5-second spike.

Aggregation needs care. Averages hide outliers, so latency is better read through [percentiles](./Percentile.md) calculated from histograms. Put the unit in the metric name, as in `request_duration_seconds`, so nobody compares milliseconds with seconds on one graph.

_Usage:_

"Can you show me error rate for just the Acme account?"

"Not from this metric, there's no customer label and we can't add one at that cardinality. Filter the traces by account instead."
