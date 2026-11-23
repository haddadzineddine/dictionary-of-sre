---
description: A written procedure for diagnosing and handling a specific alert or operational task.
---

A runbook is a written procedure for diagnosing and handling a specific [alert](./Alert.md) or operational task. Its reader is an [on-call](./On-call.md) engineer who may not know the service and may have just been woken up.

A typical runbook follows this order:

| Section         | Contents                                                     |
| --------------- | ------------------------------------------------------------ |
| What this means | The alert in plain language, and the likely user impact      |
| Check           | Dashboards, queries and commands that confirm the problem    |
| Mitigate        | Ordered, safe actions with exact commands                    |
| Verify          | What recovery looks like                                     |
| Escalate        | Who to contact and when                                      |
| Context         | Links to architecture notes and past [incidents](./Incident.md) |

The problem a runbook solves shows up at 3am. A page says "queue depth high" with no link. The wiki page someone finds is two years old and refers to a host that no longer exists, so the responder ends up phoning a colleague who is asleep. Linking the runbook directly from the alert removes the search.

Write for someone under pressure. Give exact commands with placeholders, the expected output of each step, and warnings on anything dangerous. Put [mitigation](./Mitigation.md) ahead of root-cause analysis, because the first goal is to restore service.

A stale runbook can do harm, since a responder will follow its instructions against a system that has since changed. Keep runbooks near the code, review them when the service changes, and update them after each use. A [game day](./Game%20day.md) is a good way to find out whether one still works.

If a runbook is a fixed sequence that needs no judgement, it is [toil](./Toil.md) and a candidate for automation, either as a script a person triggers or as a fix that removes the page altogether.

_Usage:_

"The alert fired but I have no idea what this consumer even does."

"Follow the runbook link in the page. Step two tells you how to drain the queue safely, and it lists who to call if that fails."
