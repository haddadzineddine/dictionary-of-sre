---
description: Many clients acting at the same moment, such as after a cache expiry or outage, and overwhelming a shared resource.
---

A thundering herd is many clients or processes acting at the same moment and overwhelming a shared resource. Something usually synchronises them: a cache expiry, a service restart, a scheduled job or a reconnect after a failure.

| Trigger                      | What happens                            | Countermeasure                                     |
| ---------------------------- | --------------------------------------- | -------------------------------------------------- |
| Popular cache key expires    | Every request hits the database at once | Request coalescing, where one fetches and others wait |
| Many keys share one lifetime | They all expire together                | Add a random offset to each expiry                 |
| Service recovers from outage | All clients reconnect and retry         | Exponential backoff with jitter                    |
| Midnight scheduled job       | The whole fleet runs it simultaneously  | Randomise start times                              |
| Cold restart                 | Empty caches send all reads downstream  | Warm caches and ramp traffic gradually             |

The recognisable symptom is a service that comes back after an outage and immediately falls over again. Engineers restart it, it survives for seconds, and the cycle repeats. Load at recovery can exceed the normal peak, because clients have accumulated retries and every one of them tries at once.

Randomness breaks the synchronisation. If 1,000 clients each retry after exactly 5 seconds, the server receives 1,000 requests in the same instant, round after round. Spreading those retries randomly over 30 seconds gives about 33 per second. This random spread is called jitter. Exponential backoff, which lengthens the wait after each failure, is not enough alone, because clients that failed together still retry together at 1, 2 and 4 seconds.

During recovery, the usual [mitigation](./Mitigation.md) is to admit traffic in stages at the load balancer and reject the excess until caches are warm. The recovery phase of [stress testing](./Stress%20testing.md) can expose this behaviour in advance.

_Usage:_

"We brought the API back and it died again in thirty seconds."

"That's a thundering herd, every mobile client is retrying at once. Let traffic in 10% at a time and add jitter to the client backoff."
