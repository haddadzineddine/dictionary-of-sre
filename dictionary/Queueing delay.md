---
description: Time a request spends waiting for a busy resource before work on it starts. It grows sharply near full utilisation.
---

Queueing delay is the time a request spends waiting for a busy resource before any work on it begins. Total latency is queueing delay plus service time, the time spent doing the work.

A simple queueing model, known as M/M/1, assumes one server, random arrivals and exponentially distributed service times. In this model the average total time is S / (1 - U), where S is service time and U is utilisation. With a 10 ms service time:

| Utilisation | Average wait | Average total |
| ----------- | ------------ | ------------- |
| 50%         | 10 ms        | 20 ms         |
| 80%         | 40 ms        | 50 ms         |
| 90%         | 90 ms        | 100 ms        |
| 95%         | 190 ms       | 200 ms        |
| 99%         | 990 ms       | 1,000 ms      |

Moving from 90% to 99% utilisation is 10% more traffic and ten times the latency. Real systems have different numbers, since multiple workers soften the curve and bursty traffic worsens it, but the shape holds.

The symptom is a service that is fine all day until a small rise at peak makes latency jump. Engineers search for a bad query or a recent deployment that does not exist. Each request takes no longer to process once it starts, so [profiling](./Profiling.md) shows nothing new. In a [distributed trace](./Distributed%20trace.md), the time appears as a gap before a [span](./Span.md) begins or between client and server spans.

To respond, keep utilisation below the steep part of the curve through [capacity planning](./Capacity%20planning.md), reduce service time or add workers. Bound queues and reject work when they fill, because a request served after its client has given up is wasted. Track queue depth or wait time as a [saturation](./Saturation.md) signal. [Little's Law](./Little's%20Law.md) connects the two: 30 requests waiting at 200 per second means a 150 ms average wait.

_Usage:_

"Traffic is up 8% and p99 tripled. Did someone ship something?"

"Nothing shipped. We're at 92% utilisation, so that's queueing delay. Add two instances and it should drop."
