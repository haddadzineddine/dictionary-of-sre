---
description: A record of one request's path through multiple services, shown as a tree of timed spans.
---

A distributed trace is a record of one request's path through the services that handled it. It is made of [spans](./Span.md), each representing a timed operation, joined by a shared trace ID into a tree.

A trace view for a hypothetical slow checkout looks like this:

```text
POST /checkout       [==============================] 820 ms
  auth.verify        [==]                              40 ms
  cart.load             [===]                          60 ms
  payment.charge            [======================]  640 ms
    fraud.score             [==================]      520 ms
```

The tree answers a question that per-service dashboards struggle with. When a request is slow and it crosses twelve services, each team can look at its own graphs, see nothing unusual and conclude the problem is elsewhere. Here the trace shows that checkout is slow because the payment step spends most of its time waiting on fraud scoring.

Traces also expose patterns that are hard to see in aggregate. Calls that could run in parallel appear one after another. Retries show up as repeated spans. The N+1 query problem, where code runs one query per item in a list, appears as a long staircase of identical database spans.

Tracing depends on every service in the path. Each must be instrumented and must forward the trace ID through [context propagation](./Context%20propagation.md). If one service in the middle does not, the trace splits into disconnected fragments. Unexplained gaps in a timeline usually mean uninstrumented work, such as time spent waiting in a queue.

Storing every trace is expensive, so most systems use [sampling](./Sampling.md). The consequence is that the trace for the specific failed request you are investigating may not have been kept. A trace explains one request, so use a [metric](./Metric.md) to find out how common the behaviour is. [OpenTelemetry](./OpenTelemetry.md) is the common standard for producing traces.

_Usage:_

"Search is slow but the search service's own latency looks flat."

"Pull a distributed trace for a slow request. I'd bet the time is going to the ranking call before search even starts."
