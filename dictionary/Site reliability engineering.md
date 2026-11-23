---
description: A discipline that keeps production services reliable by applying software engineering to operations work.
---

Site reliability engineering (SRE) is the discipline of keeping production services reliable by applying software engineering to operations work. The term comes from Google, where Ben Treynor Sloss started an SRE team in 2003. An SRE team writes automation, sets reliability targets, responds to failures and feeds what it learns back into the software.

SRE addresses a conflict between shipping and stability. Developers are rewarded for releasing changes, operators for keeping systems steady, and each release can turn into a negotiation. SRE replaces that argument with a number: the team agrees an [SLO](./SLO.md), and the gap between that target and 100% becomes an [error budget](./Error%20budget.md) that can be spent on risky change.

Typical SRE work falls into these areas:

| Practice           | What the team does                                                                    |
| ------------------ | ------------------------------------------------------------------------------------- |
| Set targets        | Defines [SLIs](./SLI.md) and SLOs with product owners                                 |
| Reduce [toil](./Toil.md) | Automates repetitive manual work                                                |
| Build visibility   | Adds [observability](./Observability.md) so production behaviour can be explained     |
| Share [on-call](./On-call.md) | Responds to [alerts](./Alert.md) and [incidents](./Incident.md)            |
| Learn from failure | Writes [blameless postmortems](./Blameless%20postmortem.md)                           |
| Plan ahead         | Does [capacity planning](./Capacity%20planning.md) before demand arrives              |

SRE is different from renaming an operations team. A team that spends all day on tickets, with no time for engineering, is doing operations under a new title. The usual symptom is a pager load that grows in step with the number of services. Google's published guidance caps operational work at 50% of an SRE's time so the rest can go to engineering that reduces future work.

SRE overlaps with DevOps, the broader practice of developers and operators sharing responsibility for delivery. DevOps describes a culture and a set of goals. SRE is one specific way of pursuing them, built around numeric targets.

_Usage:_

"We hired two SREs and now they get every ticket nobody else wants."

"Then we've built an ops team. Give them the SLOs, cap the interrupt work, and let them automate the top sources of pages."
