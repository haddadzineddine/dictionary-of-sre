---
description: The share of deployments that need immediate intervention afterwards, such as a rollback or hotfix.
---

Change failure rate is the share of deployments to production that need immediate intervention afterwards, such as a rollback or hotfix. It is one of the [DORA metrics](./DORA%20metrics.md), where it is named "change fail rate". The formula is failed deployments divided by total deployments. If a team ships 120 deployments in a quarter and 6 need a rollback or hotfix, the rate is 5%.

The number depends on what the team counts as a failure:

| Event                                                  | Usually counts?                |
| ------------------------------------------------------ | ------------------------------ |
| Deployment rolled back because of errors               | Yes                            |
| Hotfix shipped to correct a release                    | Yes                            |
| Caught by a canary, a release sent to a small share of traffic first | Depends, so record the policy |
| Bug found weeks later and fixed in normal work         | No                             |
| Outage caused by a cloud provider                      | No                             |
| Planned rollback for business reasons                  | No                             |

Without agreed definitions, the figure can drift away from reality. The telltale sign is a rate of 1% on a team whose engineers remember constant firefighting. Often failures are being fixed forward in later releases and never labelled as failures. Linking deployments to [incidents](./Incident.md) and to hotfix commits gives a more honest count.

The rate says nothing about severity. A change that takes down checkout counts the same as one with a minor effect, so read it alongside [error budget](./Error%20budget.md) spend and recovery time.

Deploying less often to reduce failures tends to backfire. Larger batches carry more change per release and are harder to diagnose when they go wrong. To lower the rate, make changes smaller, strengthen automated tests, validate configuration, release to a growing share of traffic, and act on the [contributing factors](./Contributing%20factor.md) found in past failures.

_Usage:_

"Our change failure rate is 2%, so why does [on-call](./On-call.md) feel so bad?"

"We only count rollbacks. Half our bad releases get patched forward, so let's count those hotfixes too."
