---
description: Manual, repetitive operational work that could be automated, leaves no lasting value and grows as the service grows.
---

Toil is operational work that is manual, repetitive, automatable, reactive, without lasting value, and growing in proportion to the service. Restarting a stuck worker every morning, resizing disks by hand and rotating a certificate from a checklist are typical examples. The term is used in [site reliability engineering](./Site%20reliability%20engineering.md) to separate this work from engineering that improves the system.

A task is more likely to be toil the more of these tests it meets:

| Test              | Question                                          |
| ----------------- | ------------------------------------------------- |
| Manual            | Does a person have to run it?                     |
| Repetitive        | Has the team done this many times before?         |
| Automatable       | Could a machine do it equally well?               |
| Tactical          | Is it driven by interrupts instead of a plan?     |
| No lasting value  | Is the service unchanged once it is done?         |
| Scales with growth | Does twice the traffic mean twice the work?      |

Not all unpleasant work is toil. Investigating a new kind of failure is engineering, because it produces knowledge the team did not have.

The symptom of too much toil is a team that is busy all week while nothing improves. Project work keeps slipping, [on-call](./On-call.md) shifts are spent clearing tickets, and each new service means asking for more headcount. Google's SRE guidance caps operational work at 50% of an engineer's time for this reason.

Start by measuring. Record hours by category for a few weeks, then address the largest item. Prefer removing the need over scripting it: fix the bug that makes the worker stick instead of automating the restart. A [runbook](./Runbook.md) followed identically every time is a good candidate for automation. Compare costs before building anything. A 30-minute weekly task takes 26 hours a year, so automation that takes 16 hours to build pays back in 32 weeks.

_Usage:_

"I spent half of on-call manually clearing the stuck export queue again."

"That's toil. Let's find why it sticks this sprint, because a cron job to clear it just hides the bug."
