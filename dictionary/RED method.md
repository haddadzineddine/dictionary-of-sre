---
description: A monitoring recipe for request-driven services that tracks rate, errors and duration for every service.
---

The RED method is a [monitoring](./Monitoring.md) recipe for request-driven services: for each service, track Rate, Errors and Duration. Tom Wilkie proposed it as a simpler form of the [golden signals](./Golden%20signals.md), leaving out [saturation](./Saturation.md).

| Letter   | Measures                   | Typical [metric](./Metric.md)                                 |
| -------- | -------------------------- | ------------------------------------------------------------- |
| Rate     | Requests per second        | Rate of a [counter](./Counter.md)                             |
| Errors   | Failed requests per second | The same counter, filtered by an error label                  |
| Duration | How long requests take     | [Percentiles](./Percentile.md) from a [histogram](./Histogram.md) |

The benefit is consistency across services. Without a shared recipe, each team builds its own dashboards, and during an [incident](./Incident.md) nobody can read another team's graphs quickly. With RED, every service shows the same three panels, so an engineer can judge an unfamiliar service in seconds. A shared library or a service mesh, which is a network layer handling traffic between services, can emit these metrics automatically.

The three panels are most useful read together:

| Pattern                                | Likely meaning                          |
| -------------------------------------- | --------------------------------------- |
| Rate drops, errors flat                | Upstream failure or a routing problem   |
| Errors rise, duration falls            | Requests are failing fast               |
| Duration rises, rate steady            | Slow dependency or a resource limit     |
| Rate rises, then duration, then errors | Overload                                |

Read errors against rate. Five failures per second is a serious problem at 50 requests per second and a small one at 5,000. [Alerts](./Alert.md) generally use the ratio.

RED has limits. It says which service is unhealthy but not which resource is responsible, which is the job of the [USE method](./USE%20method.md). It also fits poorly with batch jobs and stream processors, which have no request to measure and are better judged by throughput and how far behind they are.

_Usage:_

"I've never touched the inventory service. Where do I start?"

"Open its RED dashboard. If duration is up with a steady rate, look at what it calls before you look at its code."
