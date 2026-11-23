---
description: The amount of failure an SLO allows during its window, which a team can spend on releases, experiments and other risk.
---

An error budget is the amount of failure an [SLO](./SLO.md) permits during its window. It equals 100% minus the SLO, so a 99.9% SLO has a 0.1% error budget.

A hypothetical service shows how the percentage becomes a usable number:

| Input                            | Value              |
| -------------------------------- | ------------------ |
| SLO                              | 99.9% over 30 days |
| Budget as a share                | 0.1%               |
| Expected requests in the window  | 10,000,000         |
| Budget in failed requests        | 10,000             |
| Budget as time at total outage   | 43.2 minutes       |

If a bad deployment fails 4,000 requests, 40% of the budget is gone and 6,000 failures remain. Partial failures spend budget more slowly. At steady traffic, 5% of requests failing for one hour costs the same as a total outage of three minutes, because 0.05 x 60 = 3.

The budget gives product and engineering one shared number for decisions about risk. Without it, teams argue about whether a release feels too dangerous, and the outcome depends on opinion. With it, budget remaining means the team can keep shipping, and a nearly empty budget means effort moves to stability. An [error budget policy](./Error%20budget%20policy.md) writes those consequences down in advance.

Every user-visible failure spends the budget, whatever the cause: bad deployments, dependency outages, traffic spikes and planned maintenance. Tracking spend by cause shows which fix would return the most budget.

With a rolling window, old failures age out and the budget recovers gradually. A [burn rate alert](./Burn%20rate%20alert.md) watches how quickly the budget is being spent, so a fast failure pages someone long before the budget is empty. A budget that is never touched for several quarters is also information: the SLO may be stricter than users need, or the team may be releasing too cautiously.

_Usage:_

"Can we ship the new auth flow on Thursday?"

"Monday's outage used 85% of the error budget. Let's hold it until the window recovers, or release it behind a flag to 1% of users."
