---
description: A pre-agreed document stating what a team does as its error budget runs down, such as pausing risky releases.
---

An [error budget](./Error%20budget.md) policy is a written agreement stating what a team does as that budget is consumed. Product and engineering leads agree it before it is needed, so a decision to slow releases is not renegotiated during a bad month.

Without a policy, an [SLO](./SLO.md) has no consequence. The familiar pattern is a team that misses its target three months running, notes this in a review, and keeps shipping at the same pace because nobody agreed what a miss should change.

A policy is usually a ladder of responses. The thresholds below are an example, not a standard:

| Budget consumed | Response                                                        |
| --------------- | --------------------------------------------------------------- |
| Under 50%       | Normal releases                                                 |
| 50% to 75%      | Review recent failures and roll out risky changes more slowly   |
| 75% to 100%     | Low-risk changes only, with extra review of each release        |
| Over 100%       | Freeze feature releases and prioritise reliability work until the service is back within SLO |

A useful policy also answers the questions that come up under pressure. It says which changes are exempt from a freeze, typically security and legal fixes and the fix for the outage itself. It says whether failures caused by an external dependency count. It sets a trigger for review, such as requiring a [blameless postmortem](./Blameless%20postmortem.md) for any single [incident](./Incident.md) that consumes more than 20% of the budget. It also names who can override the policy and how that decision is recorded.

A freeze redirects engineering effort toward whatever burned the budget. If freezes keep blocking work for months, the target may be unrealistic for the current system. In that case, renegotiate the SLO using the data, or fund the reliability work, instead of quietly ignoring the policy.

_Usage:_

"Product says the launch can't slip, but we're at 110% of budget."

"The error budget policy they signed says feature freeze. If they want an exception, the VP records it and we accept the risk explicitly."
