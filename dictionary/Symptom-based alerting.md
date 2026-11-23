---
description: Paging on what users experience, such as errors and slowness, instead of on internal causes such as high CPU.
---

Symptom-based alerting is paging on what users experience, such as failed or slow requests, instead of on internal conditions that may or may not matter. A symptom is what is wrong from the user's side. A cause is why it is happening.

| Aspect                     | Cause-based [alert](./Alert.md) | Symptom-based alert              |
| -------------------------- | ------------------------------- | -------------------------------- |
| Example                    | CPU above 90%                   | Checkout errors threatening the [SLO](./SLO.md) |
| Fires with no user harm    | Often                           | Rarely                           |
| Catches unforeseen failures | Only the listed causes         | Any cause with visible impact    |
| Explains why               | Sometimes                       | No, this needs investigation     |

Cause-based alerts fail in two directions. One replica dies behind a load balancer, traffic moves to the others, no user notices, and someone is still woken at 3am. On another night a bad configuration fails every payment while CPU, memory and disk stay normal, and nothing pages. Teams recognise this as being woken for harmless events and missing the serious ones.

To implement it, measure an [SLI](./SLI.md) close to the user and alert on it, commonly with a [burn rate alert](./Burn%20rate%20alert.md). Keep cause data on dashboards, such as the [golden signals](./Golden%20signals.md), where it helps with diagnosis once a page has fired.

Some causes still warrant an alert, because by the time the symptom appears it is too late to act cheaply. A disk filling up, a certificate about to expire, a quota nearly reached and a failing backup job all belong here. These alerts give lead time, and many can be tickets.

Low-traffic services need extra care, because one failure in ten requests reads as a 10% error rate. Add a minimum request count to the rule, or use [synthetic monitoring](./Synthetic%20monitoring.md) to provide steady traffic. For data pipelines, the symptom is usually stale data, so alert on freshness.

_Usage:_

"Should we page when a Kafka broker goes down?"

"Not by itself. With symptom-based alerting we page if producers start failing, and a lost broker opens a ticket for the morning."
