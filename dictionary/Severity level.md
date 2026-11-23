---
description: A ranking of an incident's impact that sets how urgently people respond and who gets involved.
---

A severity level is a ranking of an [incident](./Incident.md)'s impact that determines how urgently people respond and who gets involved. Most organisations use labels such as SEV1 to SEV4 or P0 to P3, with lower numbers meaning more serious. Exact definitions vary between companies.

Here is an example scale:

| Level | Impact                                        | Response                                  | Example                       |
| ----- | --------------------------------------------- | ----------------------------------------- | ----------------------------- |
| SEV1  | Critical service down, or data at risk        | Page now, [incident commander](./Incident%20commander.md), executive updates | Checkout down for all users |
| SEV2  | Major degradation, or key feature down for some | Page now, incident commander            | Search failing in one region  |
| SEV3  | Minor or partial impact with a workaround     | Handle in working hours                   | Report exports running slow   |
| SEV4  | Cosmetic or no user impact                    | Ticket                                    | Broken chart on an admin page |

Define levels by user impact, and not by the technical cause or by who is complaining. Measurable criteria help: share of users affected, revenue at risk, data loss, security exposure. Without them, the familiar result is an argument in the incident channel about whether this is a SEV1 or a SEV2 while nobody works on the fix. Scales also decay in two directions. If everything is labelled SEV1, the label stops producing urgency. If teams label things low to avoid scrutiny, serious problems miss review.

When unsure, choose the higher level and downgrade later. Severity is expected to change as responders learn more, and each change should be recorded in the timeline.

Severity should drive process automatically. For example, SEV1 and SEV2 might require a [blameless postmortem](./Blameless%20postmortem.md), a status page update and customer notification.

Avoid using severity counts as a performance target for teams. Judging a team on its number of SEV1s encourages under-reporting, and hidden incidents cannot be learned from.

_Usage:_

"Is this a SEV1? Only the EU region is affected."

"EU is 40% of our customers and they can't pay. By our criteria that's a SEV1, and we can downgrade if the impact narrows."
