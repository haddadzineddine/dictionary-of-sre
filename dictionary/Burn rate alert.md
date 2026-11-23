---
description: An alert on how fast an error budget is being spent, so severe failures page quickly and slow ones raise a ticket.
---

A burn rate [alert](./Alert.md) fires based on how fast a service is consuming its [error budget](./Error%20budget.md). Burn rate is the observed error rate divided by the error rate the [SLO](./SLO.md) allows. A burn rate of 1 spends the budget exactly over the SLO window, and a burn rate of 10 spends it in a tenth of the window.

For a 99.9% SLO over 30 days, the allowed error rate is 0.1%. An observed error rate of 1.44% is a burn rate of 14.4, and at that pace the 720-hour budget lasts 50 hours.

Plain thresholds handle this badly. A rule such as "errors above 0.1% for 5 minutes" pages on small blips that cost almost no budget, while a steady leak just under the threshold never fires.

The Google SRE Workbook recommends pairing a long and a short window at several burn rates:

| Severity | Long window | Short window | Burn rate | Budget spent | Error rate at 99.9% |
| -------- | ----------- | ------------ | --------- | ------------ | ------------------- |
| Page     | 1 hour      | 5 minutes    | 14.4      | 2%           | 1.44%               |
| Page     | 6 hours     | 30 minutes   | 6         | 5%           | 0.6%                |
| Ticket   | 3 days      | 6 hours      | 1         | 10%          | 0.1%                |

Budget spent is burn rate x long window / 720 hours, so the first row is 14.4 x 1 / 720 = 2%.

Both windows must exceed the threshold. The long window shows that a meaningful amount of budget has gone. The short window, one twelfth of the long one, confirms the problem is still happening, so the alert clears soon after a fix.

Low-traffic services need adjustment, since one failure in 50 requests is a 2% error rate. Add a minimum request count or synthetic traffic.

_Usage:_

"Error rate is only 0.7%. Is that worth waking anyone?"

"On a 99.9% SLO that's a burn rate of 7, so the budget is gone in about four days. If it holds, the six-hour burn rate alert will page."
