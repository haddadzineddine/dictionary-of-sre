---
description: The person who coordinates an incident response, assigning roles and making decisions instead of debugging.
---

An [incident](./Incident.md) commander (IC) is the person who coordinates the response: assigning work, tracking progress, making decisions and keeping communication flowing. The IC does not debug. The role is adapted from the Incident Command System, which US emergency services developed in the 1970s to coordinate wildfire response.

The role exists because of what happens without it. The most senior engineer starts debugging and becomes absorbed. Nobody updates support, and two people apply conflicting fixes. Someone reading a stack trace cannot also track five workstreams.

Larger incidents split the work into roles:

| Role                | Responsibility                                  |
| ------------------- | ----------------------------------------------- |
| Incident commander  | Owns the overall response and decisions         |
| Operations lead     | Directs technical investigation and changes     |
| Communications lead | Updates stakeholders and customers              |
| Scribe              | Records the timeline, findings and decisions    |

In a small incident, one person may hold several of these roles. If the IC needs to go deep on a technical problem, they should hand over command first.

The IC runs a repeating loop: what do we know, what are we trying, who owns it, and when do we check back? Each action gets a named owner and a time, as in "Sam, roll back payments and report in five minutes". The IC keeps the group focused on [mitigation](./Mitigation.md) over root-cause analysis while users are affected.

The IC does not need to be the most senior person or the deepest expert on the system. The job calls for coordination skills, and authority during the incident comes from the role.

Handovers should be explicit and announced in the channel. In long incidents, rotate the IC every few hours to avoid exhausted decisions. Teams can build this skill by shadowing and during a [game day](./Game%20day.md).

_Usage:_

"Everyone's in the call and three people are suggesting different fixes."

"We need an incident commander. I'll take it. Dana, you own the rollback, and everyone else hold changes until she reports back."
