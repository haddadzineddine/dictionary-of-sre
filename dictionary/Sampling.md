---
description: Keeping only a subset of telemetry, usually traces, to control cost while retaining enough data to investigate problems.
---

Sampling is keeping a subset of [telemetry](./Telemetry.md) and discarding the rest to control cost. It is applied mainly to [distributed traces](./Distributed%20trace.md) and also to high-volume [logs](./Log.md).

The common strategies differ in when they decide:

| Strategy     | Decides                    | Keeps                                  | Trade-off                                       |
| ------------ | -------------------------- | -------------------------------------- | ----------------------------------------------- |
| Head-based   | At the start of a request  | A fixed random share                   | Cheap, but blind to the outcome                 |
| Tail-based   | After the trace completes  | Errors, slow traces, a random baseline | Must buffer all [spans](./Span.md) until it decides |
| Rate-limited | As requests arrive         | Up to a fixed number per second        | Bounded cost, but the kept share varies with traffic |

Head-based sampling loses rare failures. Take a hypothetical service handling 2,000 requests per second with a 0.1% error rate, which is 2 failures per second. Sampling 1% keeps 20 traces per second, of which on average 0.02 are failures, or about one failed trace every 50 seconds. The symptom is searching for a customer's failed request ID and finding that no trace was kept.

Tail-based sampling addresses this at a cost. A collector must hold the spans of in-flight traces in memory, and all spans of a trace must reach the same collector instance.

Decisions need to be consistent across services. The decision is normally made once and passed along by [context propagation](./Context%20propagation.md). If two services each sampled 10% independently, only about 1% of traces would be complete.

Sampled data also needs reweighting when counted. At 1-in-100, each kept trace stands for 100 requests, so raw counts understate traffic. Compute error rates and [SLO](./SLO.md) performance from unsampled [metrics](./Metric.md), and use sampled traces to explain individual behaviour.

_Usage:_

"The customer gave us a request ID, but there's no trace for it."

"We head-sample at 1%, so it probably got dropped. Let's move to tail sampling so errors are always kept."
