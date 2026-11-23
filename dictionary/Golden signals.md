---
description: Four measurements for a user-facing service, from Google's SRE book. They are latency, traffic, errors and saturation.
---

The golden signals are four measurements that Google's SRE book recommends for [monitoring](./Monitoring.md) a user-facing service: latency, traffic, errors and [saturation](./Saturation.md). If a team can measure only four things about a service, these give a reasonable picture of its health.

| Signal                         | Question it answers             | Example                                |
| ------------------------------ | ------------------------------- | -------------------------------------- |
| Latency                        | How long do requests take?      | p99 checkout duration                  |
| Traffic                        | How much demand is there?       | Requests per second                    |
| Errors                         | What share of requests fail?    | Rate of 5xx responses                  |
| Saturation                     | How full is the service?        | Connection pool in use, queue depth    |

Latency should be split by outcome. A request that fails immediately is fast, so during an outage the latency graph can improve while users get nothing but errors. Track successful and failed requests separately, and read them as [percentiles](./Percentile.md) instead of averages.

Errors include failures that do not look like errors. A 200 response with the wrong content is a failure, and so is a response slower than the service has committed to.

Traffic provides context for the others. A sudden drop often means a failure upstream, and the error rate can stay at zero throughout because no requests are arriving to fail. An error-rate graph alone will show that as a healthy service.

Saturation tends to lead the other three. It shows trouble building before users feel it, while latency and errors show what is affecting them now.

A common use is one dashboard per service with these four panels, which gives an [on-call](./On-call.md) engineer the same first view everywhere. [Symptom-based alerting](./Symptom-based%20alerting.md) usually pages on latency and errors. The [RED method](./RED%20method.md) is a narrower recipe for request-driven services, and the [USE method](./USE%20method.md) applies similar thinking to individual resources.

_Usage:_

"Latency just got better across the board. Did someone ship a performance fix?"

"Check the other golden signals first. Errors tripled, so we're failing fast and those requests are dragging the numbers down."
