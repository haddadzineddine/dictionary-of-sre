---
description: The data a running system emits about its own behaviour, such as logs, metrics, traces and profiles.
---

Telemetry is the data a running system emits about its own behaviour. Engineers use it to understand production without attaching a debugger to live machines. The common signals differ in what they record and what makes them expensive:

| Signal                                       | Records                     | Useful for                       | Cost driver                                   |
| -------------------------------------------- | --------------------------- | -------------------------------- | --------------------------------------------- |
| [Metrics](./Metric.md)                       | Numbers over time           | Trends and [alerts](./Alert.md)  | Label combinations, or [cardinality](./Cardinality.md) |
| [Logs](./Log.md)                             | Individual events           | Detail on what happened          | Volume                                        |
| [Traces](./Distributed%20trace.md)           | The path of one request     | Where time went across services  | [Span](./Span.md) volume, unless [sampled](./Sampling.md) |
| Profiles, from [profiling](./Profiling.md)   | Where code spends resources | Finding expensive functions      | Sample frequency and retention                |

Telemetry passes through a pipeline. Code is instrumented to emit it, an agent or collector gathers it, a processing stage filters and aggregates it, and a backend stores it for queries. Each stage can delay, drop or alter data.

Two problems are common. The first is cost: the [observability](./Observability.md) bill grows faster than traffic because someone added a high-cardinality label or left debug logging on in production. The second is that telemetry degrades when it is needed. During an [incident](./Incident.md), error logs surge, the pipeline backs up, and data from the relevant minutes arrives late or gets dropped. Buffering, rate limits on noisy sources and infrastructure separate from the observed systems all reduce this risk.

Telemetry can also leak sensitive data. Tokens, email addresses and payment details end up in log lines and span attributes when code records whole objects. Remove them at the source and again in the collector, because deleting data from retained stores is harder than never sending it.

Decide what to collect from the questions you need to answer. Data that no dashboard, alert or investigation uses is only a cost. [OpenTelemetry](./OpenTelemetry.md) provides a common format for producing and shipping these signals.

_Usage:_

"Why did the observability bill double this month?"

"Someone left debug logging on in the payments service. That one change accounts for most of the ingested telemetry."
