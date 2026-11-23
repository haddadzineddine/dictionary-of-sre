---
description: Collecting predefined measurements from a system and watching them through dashboards and alerts.
---

Monitoring is the practice of collecting predefined measurements from a system, displaying them and raising an [alert](./Alert.md) when they cross a threshold. It answers questions known in advance: is the service up, what is the error rate, is the disk filling.

Monitoring can look at a system from two positions, and teams generally use both:

| Approach  | Measures from                                                       | Sees                                  | Misses                                    |
| --------- | ------------------------------------------------------------------- | ------------------------------------- | ----------------------------------------- |
| White-box | Inside, through emitted [metrics](./Metric.md)                      | Queue depth, internals, early warning | Failures that stop the service reporting  |
| Black-box | Outside, as a client would, such as [synthetic monitoring](./Synthetic%20monitoring.md) | What a user would get | Why it is failing                         |

Monitoring can fail silently. If the monitoring system goes down along with what it watches, no alert fires and the dashboard shows a gap. A rule such as "error rate above 1%" also stays quiet when a service stops reporting altogether, because missing data does not satisfy the condition. An engineer then reads a flat line as a quiet night.

Three measures reduce that risk. Run monitoring on infrastructure separate from the systems it observes. Alert explicitly on missing data. Add a dead man's switch, which is an alert that fires continuously to an external service, and that service pages someone if the signal ever stops.

Dashboards have their own failure mode. Teams accumulate dozens that nobody opens until an [incident](./Incident.md), when nobody knows which one matters. A small set of service dashboards built around the [golden signals](./Golden%20signals.md) gives [on-call](./On-call.md) engineers a consistent place to start.

Monitoring covers only the failure modes someone thought to measure. [Observability](./Observability.md) extends this to questions nobody anticipated. The two work together: monitoring reports that something is wrong, and observability helps explain why.

_Usage:_

"The checkout graph was flat all night, so I assumed things were quiet."

"Flat at zero means the exporter died. Add an alert on missing data so a silent gap pages someone."
