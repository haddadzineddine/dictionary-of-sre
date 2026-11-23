---
description: A scheduled exercise in which a team simulates a failure and practises detecting, responding to and recovering from it.
---

A game day is a scheduled exercise in which a team simulates a failure and practises detecting it, responding to it and recovering from it. [Chaos engineering](./Chaos%20engineering.md) tests how software behaves. A game day also tests people, tools and procedures.

An exercise usually runs through these phases:

| Phase   | What happens                                                 |
| ------- | ------------------------------------------------------------ |
| Plan    | Choose the scenario, scope, participants and abort criteria  |
| Brief   | Tell stakeholders the time and the possible impact           |
| Inject  | A facilitator triggers or simulates the failure              |
| Respond | [On-call](./On-call.md) engineers work it as a real [incident](./Incident.md) |
| Recover | Restore normal operation and confirm the system is healthy   |
| Debrief | Record the gaps and assign actions                           |

The problem it addresses is discovering a broken procedure during a real outage. The first attempt to restore a database from backup should not be the one that reveals the backup is corrupt or that the restore takes nine hours. Game days also expose missing credentials, an out-of-date [runbook](./Runbook.md), an [alert](./Alert.md) that never fires, dashboards some responders cannot open, and uncertainty over who may make a decision.

The format can match the team's maturity. A tabletop exercise talks through the scenario without touching any system and costs little. A staging exercise runs real actions safely. A production exercise is the closest to reality and needs strong safeguards.

Keep the exercise realistic. Use the normal tools, practise the [incident commander](./Incident%20commander.md) role, and have an observer note where people got stuck. Label all communication as an exercise so nobody mistakes it for a real outage, and stop at once if a real incident begins.

Track the resulting actions as you would those from a [blameless postmortem](./Blameless%20postmortem.md). Repeat the exercise periodically, because systems and teams change.

_Usage:_

"Our DR plan says we can restore in an hour. Has anyone timed it?"

"No. Let's schedule a game day next month, restore last night's backup and see how long it takes."
