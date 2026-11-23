---
description: A rota in which an engineer is responsible for responding to production alerts within an agreed time.
---

On-call is an arrangement in which an engineer is responsible, for a set period, for responding to production [alerts](./Alert.md) within an agreed time. Teams share the duty through a rota, commonly changing weekly or daily.

A typical setup has several layers:

| Role               | Responsibility                                          |
| ------------------ | ------------------------------------------------------- |
| Primary            | Receives the page, acknowledges it and starts the response |
| Secondary          | Takes the page if the primary does not acknowledge in time |
| Escalation contact | A manager or subject expert for decisions and extra help |

Acknowledging a page means "I have this". If nobody acknowledges within a set time, often 5 to 15 minutes, the paging tool escalates to the next layer. Required response times usually depend on the [severity level](./Severity%20level.md).

Rota size determines how sustainable the duty is. With six people on a weekly rota, each engineer is on-call one week in six. Teams spread across time zones can use a follow-the-sun rota, where each region covers its own daytime so nobody is paged overnight.

An unhealthy rota is easy to recognise. People dread their week, sleep badly, and one person gets called regardless of who is on duty because only they understand a particular system. The usual causes are noisy alerts that produce [alert fatigue](./Alert%20fatigue.md), knowledge held by one person, and a missing [runbook](./Runbook.md). Track pages per shift, night-time pages and hours spent, and treat a bad trend as a production problem.

Some practices help. Write handoff notes at the end of each shift covering open issues. Let new engineers shadow a shift before taking one alone. Compensate on-call with pay or time off, including recovery time after a bad night. Quiet shifts can go toward reducing [toil](./Toil.md).

_Usage:_

"Priya got paged about the billing job again and she's not even on-call this week."

"Because she's the only one who knows it. Let's write the runbook and have someone else shadow her next time."
