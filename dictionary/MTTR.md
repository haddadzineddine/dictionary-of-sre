---
description: Mean time to restore, repair, respond or resolve. An ambiguous incident average, so state which version you mean.
---

MTTR is an average [incident](./Incident.md) duration, and the R stands for different things depending on who is speaking. Two teams can report the same acronym while measuring different intervals, so a comparison between them may mean nothing until the definition is agreed.

| Expansion                  | The clock stops when                         |
| -------------------------- | -------------------------------------------- |
| Mean time to respond       | Someone starts working on the problem        |
| Mean time to restore       | Users have working service again             |
| Mean time to repair        | The faulty component is fixed                |
| Mean time to resolve       | The incident is closed, including follow-up  |

Time to restore is usually the useful version for reliability, because it matches the period users were affected. A team that restores service in 10 minutes through a [mitigation](./Mitigation.md) and ships the permanent fix three days later has a restore time of 10 minutes and a resolve time of three days.

Averages describe incident durations poorly, because most incidents are short and a few are very long. Take six hypothetical incidents lasting 10, 15, 20, 25, 30 and 300 minutes. The mean is about 67 minutes and the median is 22.5. Without the 300-minute outage the mean would be 20, so one incident more than tripled it with no change in how the team works. Quarter-on-quarter MTTR can move this way from chance alone. Report the median and the longest incident, and read the individual timelines.

MTTR can also move the wrong way when things improve. If a team eliminates its short, easy incidents, only the long ones remain and the average rises.

To reduce restore time, split it into stages: [MTTD](./MTTD.md), time to diagnose and time to mitigate. Then work on the stage that dominates. Together with [MTBF](./MTBF.md), restore time determines steady-state [availability](./Availability.md).

_Usage:_

"Leadership wants MTTR down 20% by next quarter."

"Which R? And with nine incidents a quarter, one bad night swings the mean. Let's target median time to restore instead."
