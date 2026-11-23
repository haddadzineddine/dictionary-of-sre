---
description: Service level indicator. A measured ratio of good events to valid events that describes service quality as users see it.
---

A service level indicator (SLI) is a measurement of one aspect of service quality as users experience it. The recommended form is a ratio: good events divided by valid events, expressed as a percentage. Because every SLI then sits on the same 0 to 100% scale, it can feed an [SLO](./SLO.md) and an [error budget](./Error%20budget.md) in the same way.

Common SLIs by type of service:

| Service type | SLI                        | A good event is                        |
| ------------ | -------------------------- | -------------------------------------- |
| Request API  | [Availability](./Availability.md) | A response that is not a server error |
| Request API  | Latency                    | A response in under 300 ms             |
| Data pipeline | Freshness                 | A record processed within 10 minutes   |
| Storage      | Durability                 | A stored object read back intact       |
| Batch job    | Completion                 | A scheduled run finishing on time      |

Defining an SLI takes three decisions: which events count, what makes one good, and where it is measured.

The measurement point changes the answer. A [metric](./Metric.md) recorded inside the application misses requests that never reached it, such as those dropped by a load balancer or lost to a DNS failure. Measuring at the load balancer, or with [real user monitoring](./Real%20user%20monitoring.md), sits closer to what users saw but adds noise.

Exclusions need the same care. Client errors are usually left out of valid events, but a burst of 401s from a broken login flow is a real failure, and excluding it makes a broken feature look healthy.

A weak SLI shows up as a green dashboard while users complain. CPU and memory are poor SLIs because users do not experience them directly. An average latency hides slow requests, whereas "share of requests under 300 ms" counts them. A [histogram](./Histogram.md) with a bucket boundary at the threshold makes that count exact.

_Usage:_

"Our availability SLI says 99.99, but customers keep reporting failed uploads."

"It only counts requests that reach the app. Uploads rejected at the proxy never show up. Move the measurement to the load balancer."
