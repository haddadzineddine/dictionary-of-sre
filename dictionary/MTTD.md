---
description: Mean time to detect. The average gap between an incident starting and a person or system noticing it.
---

Mean time to detect (MTTD) is the average time between the start of an [incident](./Incident.md) and its detection. Detection is usually an [alert](./Alert.md) firing. It can also be a customer ticket, which means the team's [monitoring](./Monitoring.md) missed the problem.

The mean is easily distorted by one case. If four incidents were detected after 2, 3, 5 and 30 minutes, the mean is 10 minutes and the median is 4. Report the median and the longest case alongside the mean, particularly when the sample is small.

Several things set a lower limit on detection time:

| Source of delay    | Example                                                        |
| ------------------ | -------------------------------------------------------------- |
| Collection         | A [metric](./Metric.md) scraped every 60 seconds               |
| Pipeline lag       | [Logs](./Log.md) that take minutes to become searchable        |
| Alert window       | A rule that requires a condition to hold for 10 minutes        |
| Missing coverage   | A failure no check measures, found only when users complain    |

The recognisable symptom of poor detection is learning about an outage from support, a sales colleague or social media while every dashboard still looks normal. [Symptom-based alerting](./Symptom-based%20alerting.md) closes much of this gap by watching what users experience, and [synthetic monitoring](./Synthetic%20monitoring.md) can catch failures on paths that have little traffic at that hour.

Lowering detection time has a cost. Tighter thresholds and shorter windows fire on more harmless blips, which feeds [alert fatigue](./Alert%20fatigue.md). A [burn rate alert](./Burn%20rate%20alert.md) handles this trade-off by detecting severe failures quickly and small ones more slowly.

MTTD also has two measurement hazards. The start time is often established afterwards from graphs, so teams need a consistent rule for recording it. And the metric covers only incidents that were noticed. A failure nobody detected does not appear in the average, so a low MTTD can coexist with blind spots.

_Usage:_

"Our MTTD looks great this quarter, under three minutes."

"Only for the incidents that paged. The two that customers reported first aren't timed at all, so fix how we record those."
