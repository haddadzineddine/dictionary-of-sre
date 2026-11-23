---
description: Measures of software delivery performance from the DORA research programme, covering throughput and instability.
---

DORA metrics are measures of software delivery performance from the DevOps Research and Assessment (DORA) programme. The research originally identified four key metrics. DORA renamed and narrowed one of them in 2023 and added a fifth in 2024.

| Metric                          | Group       | What it measures                                                  |
| ------------------------------- | ----------- | ----------------------------------------------------------------- |
| Change lead time                | Throughput  | Time from commit to running in production                         |
| Deployment frequency            | Throughput  | How often deployments happen                                      |
| Failed deployment recovery time | Throughput  | Time to recover from a deployment needing immediate intervention  |
| [Change fail rate](./Change%20failure%20rate.md) | Instability | Share of deployments needing immediate intervention |
| Deployment rework rate          | Instability | Share of deployments that are unplanned and caused by a production [incident](./Incident.md) |

Failed deployment recovery time replaced "time to restore service". It counts only failures caused by a change, so it excludes events such as a data centre outage, and it is narrower than a general [MTTR](./MTTR.md).

DORA's research has found that throughput and stability tend to go together: teams that deploy frequently in small batches generally also have fewer and shorter failures.

The usual misuse is turning these measures into targets or league tables. Once deployment frequency is a goal, teams split releases into trivial deployments. Once failure rate is a goal, failures get reclassified as something else. Comparisons between teams also mislead, since a mobile app that waits on store review and a web service have different constraints. Use the metrics to follow one team's trend and to find its bottleneck.

Agree what counts as a deployment and a failure, and pull the data from the deployment pipeline and incident tracker.

These metrics describe delivery and say nothing directly about user experience. Pair them with an [SLO](./SLO.md), since a team can score well on DORA while its service misses its reliability target.

_Usage:_

"The VP wants every team ranked on DORA metrics this quarter."

"Then we'll get lots of one-line deploys. Let's show each team's own trend and use it to find where changes get stuck."
