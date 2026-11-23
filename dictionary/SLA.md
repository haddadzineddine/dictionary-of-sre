---
description: Service level agreement. A contract promising a level of service, with a remedy such as credits if the provider misses it.
---

A service level agreement (SLA) is a contract between a provider and a customer that promises a level of service and states what happens if the provider misses it. The usual remedy is a service credit, which is a partial refund on the bill. That contractual consequence separates an SLA from an [SLO](./SLO.md), which is an internal target.

The three related terms differ as follows:

| Term | What it is        | Example                  | If missed                                                      |
| ---- | ----------------- | ------------------------ | -------------------------------------------------------------- |
| [SLI](./SLI.md) | A measurement | 99.93% of requests succeeded | Nothing, it is a reading                                 |
| SLO  | Internal target   | 99.9% over 30 days       | Team follows its [error budget policy](./Error%20budget%20policy.md) |
| SLA  | External contract | 99.5% per calendar month | Customer may receive credits                                   |

The SLO is normally set tighter than the SLA so the team gets a warning before money is at stake. With the numbers above, in a 30-day month, the SLO allows 43.2 minutes of full outage and the SLA allows 3.6 hours. A team that has just exhausted its SLO still has about 2.9 hours of further downtime before the contract is breached.

The definitions in the contract decide what counts. Many SLAs exclude scheduled maintenance, customer-caused failures and events outside the provider's control, and some measure [availability](./Availability.md) in their own way, such as counting a minute as down only above an error threshold. Customers often have to file a claim with evidence before a deadline. A common surprise follows a long provider outage: the credit covers a small part of one bill and none of the revenue lost.

Your own SLA is limited by those of your dependencies. If you promise 99.95% while relying on a provider that guarantees 99.9%, your business carries the difference. Check dependency terms before signing.

_Usage:_

"The customer wants 99.99% in the contract. We hit that last quarter."

"One good quarter isn't an SLA. Our SLO is 99.9%, so offer 99.5% and keep a margin before credits start."
