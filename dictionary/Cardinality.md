---
description: The number of distinct values a field can take. For metrics, the number of time series, which drives cost and query speed.
---

Cardinality is the number of distinct values a field can take. For a [metric](./Metric.md), it usually means the number of distinct time series, because each unique combination of label values is stored and indexed separately.

Labels multiply. Here is how a hypothetical request metric grows as labels are added:

| Label added | Distinct values | Possible series |
| ----------- | --------------- | --------------- |
| method      | 5               | 5               |
| route       | 20              | 100             |
| status      | 6               | 600             |
| instance    | 50              | 30,000          |
| user_id     | 100,000         | 3,000,000,000   |

The last row is an upper bound, since not every user hits every route on every instance. Even a small fraction of it is far more than the first four labels produce.

The symptom is a metrics bill that jumps after a harmless-looking one-line change, or a metrics server that runs out of memory and takes [monitoring](./Monitoring.md) down with it. Common causes are user IDs, request IDs, full URLs containing IDs, raw error messages and timestamps used as label values. Pod names add churn, because each rollout creates new names and therefore new series.

Several fixes apply:

| Fix                | Example                                                  |
| ------------------ | -------------------------------------------------------- |
| Use route templates | `/users/:id` in place of `/users/4821`                  |
| Group values       | `5xx` in place of each status code                       |
| Drop labels        | Remove them in the collection pipeline                   |
| Set limits         | Reject or cap series per service                         |
| Move the detail    | Put per-user data in [logs](./Log.md) or [traces](./Distributed%20trace.md) |

A [histogram](./Histogram.md) multiplies the count again, because each bucket is its own series.

High cardinality is not bad in itself. Asking "which customer is affected?" requires a customer ID somewhere, and that detail supports [observability](./Observability.md). The question is where to store it. Time-series databases handle it poorly, while log and trace stores are built for it.

_Usage:_

"Prometheus got OOM-killed right after the last deploy."

"Someone added the full request path as a label. Every order ID became a new series, so that's a cardinality explosion."
