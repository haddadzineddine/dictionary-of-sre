---
description: Shorthand for availability targets, such as three nines for 99.9%. Each added nine cuts allowed downtime by a factor of ten.
---

Nines are shorthand for [availability](./Availability.md) percentages, counted by the number of 9 digits. 99.9% is three nines and 99.999% is five nines. Each added nine divides the allowed downtime by ten.

| Availability | Name                 | Downtime per 30-day month | Downtime per 365-day year |
| ------------ | -------------------- | ------------------------- | ------------------------- |
| 99%          | Two nines            | 7.2 hours                 | 3.65 days                 |
| 99.9%        | Three nines          | 43.2 minutes              | 8.76 hours                |
| 99.95%       | Three and a half     | 21.6 minutes              | 4.38 hours                |
| 99.99%       | Four nines           | 4.32 minutes              | 52.56 minutes             |
| 99.999%      | Five nines           | 25.92 seconds             | 5.26 minutes              |

The calculation is (1 - availability) x period. For three nines in a 30-day month, that is 0.001 x 43,200 minutes = 43.2 minutes.

The table explains why a target that sounds modest can be out of reach. Four nines allows 4.32 minutes of full outage per month. If paging a person, opening a laptop and finding the problem takes 15 minutes, a single [incident](./Incident.md) breaks the target before anyone has started the fix. Targets at that level need automatic detection and automatic recovery, such as moving traffic to a healthy copy without human involvement.

Serial dependencies multiply. A request that passes through five components, each at 99.99%, gets at most 0.9999^5, or about 99.95%, assuming independent failures. A service cannot promise more nines than its critical dependencies allow unless it keeps working when they fail.

Nines also hide the shape of downtime. Ten 4-minute outages and one 40-minute outage produce the same monthly total but affect users differently, and the percentage does not say which users were hit. An [SLO](./SLO.md) written in nines needs a defined measurement and window to mean anything.

_Usage:_

"Sales wants to promise four nines on the API."

"That's about four minutes a month, and our database failover alone takes three. We can't sign that without automated recovery."
