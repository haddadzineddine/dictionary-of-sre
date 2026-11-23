---
description: Collecting performance and error data from actual users' browsers or apps as they use the product.
---

Real user [monitoring](./Monitoring.md) (RUM) is collecting performance and error data from real users' browsers or apps during real sessions. A small script or SDK measures page loads, interactions and client-side errors, then sends the results to a backend.

It is usually paired with [synthetic monitoring](./Synthetic%20monitoring.md), and the two have opposite strengths:

| Aspect                | Real user monitoring                 | Synthetic monitoring              |
| --------------------- | ------------------------------------ | --------------------------------- |
| Traffic               | Real users                           | Scripted checks                   |
| Coverage              | The devices and networks users have  | Chosen paths and locations        |
| Works with no traffic | No                                   | Yes                               |
| Stable over time      | Noisy, as the user mix changes       | Stable baseline                   |
| Good at finding       | Problems affecting one segment       | Outages on known paths            |

RUM shows what server-side [metrics](./Metric.md) cannot. A server can report an 80 ms response while the user waits six seconds, because the time went to the network, a large script bundle or page rendering. The familiar symptom is a backend dashboard that looks fine while users in one country on mobile complain. Segmenting by device, browser, country, connection type and release version usually exposes the group affected. Use [percentiles](./Percentile.md), because client timings have a long tail.

RUM has a blind spot during severe failures. If a page fails to load at all, its script may never run, so the outage appears as a drop in sessions and not as a rise in errors. Ad blockers and privacy settings also block some reporting. Watch session volume alongside error rate.

Because RUM runs on users' devices, privacy needs attention. URLs and form fields can contain personal data, and session recording can capture more. Collect the minimum, mask sensitive fields and follow consent rules. The script itself adds page weight, so keep it small.

_Usage:_

"Our API p95 is 90 ms, so I don't buy that the app feels slow."

"Real user monitoring says p75 load on Android in Brazil is nine seconds. The API isn't where that time is going."
