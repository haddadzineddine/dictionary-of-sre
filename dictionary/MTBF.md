---
description: Mean time between failures. The average working time between one failure and the next in a repairable system.
---

Mean time between failures (MTBF) is the average working time between one failure of a repairable system and the next. It is calculated as total operating time divided by the number of failures. Some sources measure from the start of one failure to the start of the next, which includes repair time, so check the definition before comparing figures.

Take a hypothetical service over a 30-day month, or 720 hours, with three failures that each take 2 hours to restore. Downtime is 6 hours and operating time is 714 hours, so MTBF is 714 / 3 = 238 hours and [MTTR](./MTTR.md) is 2 hours. Steady-state [availability](./Availability.md) is MTBF / (MTBF + MTTR) = 238 / 240, or about 99.17%.

That formula shows the two ways to raise availability:

| Change                   | MTBF  | MTTR   | Availability |
| ------------------------ | ----- | ------ | ------------ |
| Baseline                 | 238 h | 2 h    | 99.17%       |
| Failures half as often   | 476 h | 2 h    | 99.58%       |
| Recovery four times faster | 238 h | 0.5 h | 99.79%      |

In this example, faster recovery gains more than halving the failure rate.

MTBF is commonly misread on hardware spec sheets. A disk rated at 1,000,000 hours will not last 114 years. The figure describes a failure rate across a large population during its service life. A fleet of 1,000 such disks running for a year accumulates 8,760,000 disk-hours, so you should expect about 8.76 failures that year.

The [metric](./Metric.md) assumes a roughly constant failure rate, which suits hardware better than software. Software failures tend to cluster around changes, so a service can run quietly for weeks and then have three [incidents](./Incident.md) in a day after a release. For services, an [SLO](./SLO.md) usually says more about what users experienced.

_Usage:_

"The vendor quotes a million-hour MTBF, so we don't need spare drives, right?"

"With 2,000 drives that's roughly 18 failures a year. Keep the spares and make sure rebuilds are automatic."
