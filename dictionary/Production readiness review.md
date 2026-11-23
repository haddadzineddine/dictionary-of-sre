---
description: A structured check that a service can be run safely in production before it launches or changes owners.
---

A production readiness review (PRR) is a structured check that a service can be run safely in production, done before it launches or before another team takes responsibility for it. Google's [site reliability engineering](./Site%20reliability%20engineering.md) teams helped popularise the practice.

A review typically covers these areas:

| Area             | Questions                                                                  |
| ---------------- | -------------------------------------------------------------------------- |
| Targets          | Are [SLIs](./SLI.md) and [SLOs](./SLO.md) defined?                         |
| Visibility       | Is there enough [observability](./Observability.md) to debug it?           |
| Alerting         | Does each [alert](./Alert.md) reflect user impact and link a [runbook](./Runbook.md)? |
| Capacity         | Has [load testing](./Load%20testing.md) established its limits?            |
| Failure handling | What happens when each dependency fails?                                   |
| Change safety    | Can a release be rolled back quickly?                                      |
| Data             | Are backups taken, and has a restore been tested?                          |
| Security         | How are secrets and access managed?                                        |
| Ownership        | Who is [on-call](./On-call.md), and do they know the service?              |

The problem it prevents is a service that ships on a Friday and pages someone who has never seen it. There are no dashboards, no runbook and no list of dependencies, so the team discovers each gap during its first [incident](./Incident.md). A review moves that discovery to a point where fixes are cheaper.

Timing affects the outcome. A review held a week before launch produces a list nobody has time to address, so start the conversation at the design stage. The output should separate blocking items, which must be fixed before launch, from non-blocking items tracked with owners and dates. When a team launches with a known gap, a named person should accept that risk explicitly.

A review can also decay into bureaucracy, where a long generic checklist becomes box-ticking and teams route around it. Scale the rigour to how critical the service is, so an internal tool gets a lighter review than a payment path.

_Usage:_

"We're launching the new billing service next Tuesday. Can SRE take the pager?"

"Not before a production readiness review. Right now it has no runbook, no SLO and an untested restore."
