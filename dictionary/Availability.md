---
description: The share of time or requests in which a service works correctly for its users, usually given as a percentage.
---

Availability is the proportion of time, or of requests, in which a service does its job correctly for users. It is normally written as a percentage and often shortened to a count of [nines](./Nines.md), so 99.9% is "three nines".

There are two common ways to calculate it:

| Method        | Formula                              | Strength                       | Weakness                                     |
| ------------- | ------------------------------------ | ------------------------------ | -------------------------------------------- |
| Time-based    | uptime / (uptime + downtime)         | Easy to explain                | Counts a 3am outage and a peak outage equally |
| Request-based | successful requests / valid requests | Reflects traffic users sent    | Says little when there is no traffic         |

The two methods can disagree about the same month. Take a hypothetical service with 10,000,000 requests in 30 days, of which 5,000 failed during one 30-minute total outage. Request-based availability is 9,995,000 / 10,000,000 = 99.95%. Time-based availability is 43,170 / 43,200 minutes = 99.93%. They differ because traffic during the outage was below the monthly average.

A service can look up while being unavailable to users. The status page is green and the health check returns 200, yet checkout fails for every customer in one region. For this reason, availability is best defined from the user's side through an [SLI](./SLI.md), and a response that is too slow to be useful can count as a failure.

Dependencies limit what a service can reach. If a request needs two components in series, each 99.9% available, and their failures are independent, the combined figure is 0.999 x 0.999, or about 99.8%. Redundant copies raise availability only when they fail independently. Two replicas running the same bad configuration fail together.

Teams set a target through an [SLO](./SLO.md) instead of aiming for 100%, because each extra nine costs more.

_Usage:_

"The uptime check says 100% this month, but support has a queue of failed payments."

"That probe only hits the health endpoint. Measure availability from real checkout requests and the number will drop."
