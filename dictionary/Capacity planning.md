---
description: Forecasting future demand and arranging enough resources to serve it while still meeting reliability targets.
---

Capacity planning is forecasting future demand and arranging enough resources to serve it while still meeting [SLOs](./SLO.md). It covers compute, storage, network, database connections and third-party quotas.

The process follows a repeating sequence, shown here with hypothetical numbers:

| Step | Action                    | Example                                      |
| ---- | ------------------------- | -------------------------------------------- |
| 1    | Measure current peak      | 6,000 requests per second                    |
| 2    | Forecast growth           | 10% a month for 6 months gives about 10,630  |
| 3    | Measure unit capacity     | [Load testing](./Load%20testing.md) shows 500 per instance within SLO |
| 4    | Calculate need            | 10,630 / 500 = 21.26, so 22 instances        |
| 5    | Add redundancy            | To survive losing one of 3 zones, 11 per zone, 33 total |
| 6    | Review                    | Compare forecast with actual each quarter    |

Step 5 is often skipped. With 22 instances spread over three zones, losing a zone leaves 14 or 15, which cannot carry the forecast peak. With 33, the remaining two zones hold 22.

Forecasts need two inputs. Organic growth is the steady trend visible in historical data. Inorganic growth comes from launches, marketing campaigns and large new customers, and it appears in no graph until it happens. The familiar failure is a launch that takes the site down because nobody told the infrastructure team about it.

Lead times shape the plan. Quota increases, hardware orders and splitting a database can take weeks. Autoscaling, which adds instances automatically as load rises, does not remove the need to plan. It reacts with a delay, stops at quota limits and does not help components that cannot scale out, such as a single primary database.

Plan around [saturation](./Saturation.md) instead of CPU alone, and keep headroom, because [queueing delay](./Queueing%20delay.md) rises sharply near capacity. Re-measure unit capacity after major releases, since a performance regression changes every number above.

_Usage:_

"Marketing is running a TV ad in March. Do we need to do anything?"

"Yes, get the expected traffic from them now. Capacity planning needs a number, and our database quota takes weeks to raise."
