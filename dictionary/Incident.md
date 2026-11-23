---
description: An unplanned disruption or degradation of a service that needs a coordinated response to restore normal operation.
---

An incident is an unplanned disruption or degradation of a service that requires a coordinated response. A server crash absorbed by redundancy is a normal event. The same crash causing failed user requests is an incident. Many teams also declare incidents for security events and serious near misses.

An incident generally moves through these stages:

| Step    | Who                       | What happens                                                        |
| ------- | ------------------------- | ------------------------------------------------------------------- |
| Detect  | [Alert](./Alert.md) or customer | The problem is noticed                                        |
| Triage  | [On-call](./On-call.md) engineer | Assesses impact and sets a [severity level](./Severity%20level.md) |
| Declare | On-call engineer          | Opens a channel and names an [incident commander](./Incident%20commander.md) |
| Mitigate | Responders               | Applies a [mitigation](./Mitigation.md) to restore service          |
| Resolve | Responders                | Confirms recovery and closes the incident                           |
| Learn   | Team                      | Writes a [blameless postmortem](./Blameless%20postmortem.md)        |

The cost of having no process is easy to recognise. Five engineers debug separately in direct messages, two of them push different changes at the same moment, support has no idea what to tell customers, and nobody knows who is in charge. Declaring an incident creates one place to talk and clear roles.

Declare early. A false alarm costs a few minutes to close, while a late declaration leaves people working without coordination during the period that matters. Written criteria remove the need for judgement under stress, for example any customer-visible impact lasting over 10 minutes, any risk to data, or any problem needing more than one team.

During the response, keep a timestamped record of observations and actions, since memory of the sequence fades quickly and the review depends on it. Send stakeholders updates at a regular interval, even when the update is that nothing has changed.

Each incident spends [error budget](./Error%20budget.md), and its timings feed measures such as [MTTD](./MTTD.md) and [MTTR](./MTTR.md).

_Usage:_

"Should I open an incident? It might just be a blip on the payments graph."

"Open it. If it's nothing, we close it in five minutes, and if it's real we already have a channel and a commander."
