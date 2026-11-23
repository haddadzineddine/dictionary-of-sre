---
description: Service level objective. A target for an SLI over a time window, such as 99.9% of requests succeeding over 30 days.
---

A service level objective (SLO) is a target value for an [SLI](./SLI.md) over a defined time window. It states how reliable a service needs to be, and so also how much unreliability is acceptable.

An SLO has three parts:

| Part   | Example                                                  |
| ------ | -------------------------------------------------------- |
| SLI    | Share of checkout requests that succeed in under 500 ms  |
| Target | 99.9%                                                    |
| Window | Rolling 30 days                                          |

The gap between the target and 100% is the [error budget](./Error%20budget.md). In this example, 0.1% of checkout requests may fail or run slow before the objective is missed.

The window can be rolling or calendar-based. A rolling window always covers the last 30 days, so a bad week keeps counting until it ages out. A calendar window resets on the first of the month, which suits reporting but lets a team start fresh the day after a major outage.

Choose a target from what users tolerate and what the service has achieved recently. A target set too high means every small blip blocks releases, and teams soon start ignoring it. A target set too low reports success while users are unhappy, which appears as a healthy SLO dashboard next to a growing pile of support tickets. Review the target periodically against complaints and [incident](./Incident.md) history.

An SLO only has an effect if missing it changes a decision. An [error budget policy](./Error%20budget%20policy.md) records what the team will do when the budget runs low. Avoid 100% as a target: it cannot be met, and it leaves no room for releases or maintenance.

Keep internal SLOs tighter than any external [SLA](./SLA.md). The difference gives the team a warning and time to react before a contract is breached.

_Usage:_

"Latency spiked for twenty minutes yesterday. Do we need to drop everything?"

"Check the SLO first. We've used 12% of the budget this window, so file a follow-up and keep the release on track."
