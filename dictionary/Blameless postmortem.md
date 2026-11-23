---
description: A written review of an incident that examines how the system allowed it, without assigning fault to individuals.
---

A blameless postmortem is a written review of an [incident](./Incident.md) that examines how the system and organisation allowed it to happen, without assigning fault to individuals. It starts from the assumption that people acted reasonably given what they knew at the time.

The reason is practical. If admitting a mistake leads to punishment, people stop reporting mistakes and leave details out, and the organisation loses the information it needs. The recognisable failure is a report concluding "engineer ran the wrong command, reminded to be more careful". Nothing changes, and the same failure returns. Better questions are why one command could do that much damage, why nothing asked for confirmation, and why staging behaved differently.

A typical document contains:

| Section                                             | Contents                                         |
| --------------------------------------------------- | ------------------------------------------------ |
| Summary                                             | What happened, in a few sentences                |
| Impact                                              | Users affected, duration, [error budget](./Error%20budget.md) spent |
| Timeline                                            | Timestamped events and decisions                 |
| [Contributing factors](./Contributing%20factor.md)  | Conditions that allowed or worsened the failure  |
| What went well                                      | Practices worth keeping                          |
| Action items                                        | Specific changes, each with an owner and due date |

Blameless does not mean unaccountable. People are still expected to take part honestly and to complete their actions. Many teams refer to roles in the document, such as "the [on-call](./On-call.md) engineer", instead of names.

Action items are where postmortems most often fail. Vague, unowned items such as "improve [monitoring](./Monitoring.md)" are forgotten once the document is filed. Write specific, checkable items, such as "page when payment errors burn budget at 14.4 times the allowed rate". Prefer changes to systems over training or reminders, because systems do not depend on someone remembering. Track items to completion with the same priority as feature work.

Hold the review within a few days and share the result widely.

_Usage:_

"The postmortem draft says Tom caused the outage by deploying on Friday."

"Rewrite that. Ask why our pipeline let a broken config reach every region, because that's still true whoever deploys next."
