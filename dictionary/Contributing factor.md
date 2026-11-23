---
description: A condition that helped an incident happen or made it worse, used in place of naming one single root cause.
---

A contributing factor is a condition that helped an [incident](./Incident.md) happen or made it worse. Incident reviews use the term because failures in complex systems rarely have one cause. Several ordinary conditions, each harmless alone, combine.

Consider a hypothetical incident in which a configuration change sets a timeout to zero and takes an API down for 50 minutes:

| Contributing factor                       | Effect                            | Action                                              |
| ----------------------------------------- | --------------------------------- | --------------------------------------------------- |
| Config values were not validated          | The bad value was accepted        | Add schema validation                               |
| Change applied to all regions at once     | Local mistake became global       | Roll out region by region                           |
| [Alerts](./Alert.md) watched only CPU     | Detected after 20 minutes, by support | Add [symptom-based alerting](./Symptom-based%20alerting.md) |
| [Runbook](./Runbook.md) had an old command | Recovery took 15 minutes longer  | Update and test it                                  |

A trigger is different from a factor. The trigger is the event that set off the failure, here the config change. The factors explain why that event did so much damage, and most of them existed long before the day of the incident.

Asking for a single root cause tends to stop the investigation at the first plausible answer, which is often the last human action: "an engineer pushed bad config". Fixing only that leaves every other condition in place, ready to combine with the next mistake. The symptom is a team that closes each incident with a fix and still sees the same kind of failure return with a different trigger.

"Human error" is not a useful factor on its own. Treat it as the start of the question: what made this action easy to get wrong and hard to notice?

Not every factor deserves work. In a [blameless postmortem](./Blameless%20postmortem.md), record a decision for each one: fix it, monitor it or accept the risk explicitly.

_Usage:_

"Can we just put 'bad deploy' as the root cause and close this?"

"The deploy was the trigger. No canary and a silent alert were contributing factors, and those will bite us again."
