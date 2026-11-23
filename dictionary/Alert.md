---
description: An automated notification that a monitored condition has been met and may need someone's attention.
---

An alert is an automated notification that a monitored condition has been met. It is defined by a rule such as "error rate above 2% for 5 minutes", a severity, a destination and ideally a link to a [runbook](./Runbook.md). The duration in the rule filters out brief blips, at the price of a slower response.

Alerts should be routed according to how quickly someone needs to act:

| Level     | Delivery                  | Expected response           | Example                    |
| --------- | ------------------------- | --------------------------- | -------------------------- |
| Page      | Phone call or push        | Act within minutes          | Checkout is failing        |
| Ticket    | Issue queue               | Act within days             | Disk fills in two weeks    |
| Record    | Dashboard, no notification | Read during investigation  | Brief spike in retries     |

The test for a page is whether a person needs to do something now. A page that wakes an engineer for a condition that clears itself before they have opened a laptop teaches them to ignore the next one, which leads to [alert fatigue](./Alert%20fatigue.md). If no action is needed, make it a ticket or remove it.

Flapping is a common defect, where a value hovers around the threshold and the alert fires and clears repeatedly. Use a longer window, or separate thresholds for firing and clearing.

A useful page says what is broken and who is affected. "HighCPU on host-17" leaves the responder to work out whether anything matters. "Checkout error rate 4%, EU users affected" with links to a dashboard and runbook gives a starting point.

Grouping related alerts stops one outage from sending 200 notifications. Silences for planned maintenance should have an expiry.

Pages are generally tied to user impact through [symptom-based alerting](./Symptom-based%20alerting.md), often as a [burn rate alert](./Burn%20rate%20alert.md) on an [SLO](./SLO.md). They go to whoever is [on-call](./On-call.md).

_Usage:_

"I got paged for high memory at 4am and it was gone by the time I logged in."

"Then that alert shouldn't page. Downgrade it to a ticket and page on user-facing errors instead."
