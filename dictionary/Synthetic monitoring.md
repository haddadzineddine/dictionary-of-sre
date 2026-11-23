---
description: Running scripted requests against a service on a schedule to check it works, whether or not real users are active.
---

Synthetic [monitoring](./Monitoring.md) is running scripted checks against a service on a schedule, from outside it, to verify that it works. A check can be a single HTTP request or a browser script that signs in, adds an item to a basket and reaches checkout. Because it does not depend on user traffic, it can detect failures when nobody is using the service.

| Check type   | What it does                                    | Catches                                  |
| ------------ | ----------------------------------------------- | ---------------------------------------- |
| Uptime       | Requests a URL and checks the status            | Full outages, DNS faults, expired certificates |
| API          | Calls endpoints in sequence, validates replies  | Broken contracts and authentication      |
| Browser      | Drives a real browser through a user journey    | Broken pages and script errors           |
| Multi-region | Runs the same check from several locations      | Regional routing failures                |

The main use is covering quiet periods. A bad deployment at 3am breaks login, no customers are awake to notice, and the first report arrives at 8am. A scripted login every minute would have raised an [alert](./Alert.md) within minutes, which directly lowers [MTTD](./MTTD.md).

The limits come from the fact that the users are not real. Only the scripted paths are covered. A clean browser on a fast datacenter connection does not represent an old phone on a slow network, so pair it with [real user monitoring](./Real%20user%20monitoring.md).

A few practices keep checks trustworthy. Require failures from more than one location before paging, so that a network problem at one probe does not wake anyone. Use dedicated test accounts and tag the traffic so it stays out of business analytics and any [SLI](./SLI.md) based on real users. Avoid real side effects such as live payments. Maintain the scripts, because a redesigned button will fail a browser check, and a check that fails routinely contributes to [alert fatigue](./Alert%20fatigue.md).

_Usage:_

"Nobody noticed the password reset was broken until Monday."

"It gets almost no weekend traffic. Add a synthetic monitoring check for that flow so we don't rely on users reporting it."
