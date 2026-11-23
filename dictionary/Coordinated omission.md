---
description: A load-testing error where the tool sends fewer requests during a stall, so the slowest latencies go unrecorded.
---

Coordinated omission is a measurement error in which a load generator slows down when the system under test slows down, and so never records the latency of requests it should have sent. Gil Tene gave the problem its name. It makes [tail latency](./Tail%20latency.md) look far better than it is.

The mechanism is a closed loop. Many tools wait for each response before sending the next request. During a stall, the tool sends nothing, so the tool and the server in effect cooperate to leave out samples at the moment the measurements would be worst.

Take a hypothetical test intended to send 100 requests per second for 60 seconds over one connection. Responses normally take 5 ms, and the server stalls once for 10 seconds.

| Measure                   | Recorded by closed-loop tool | Measured from intended send times |
| ------------------------- | ---------------------------- | --------------------------------- |
| Requests counted          | 5,001                        | 6,000                             |
| Requests delayed by stall | 1                            | 1,000                             |
| [p99](./Percentile.md)    | 5 ms                         | About 9.4 s                       |
| Maximum                   | 10 s                         | 10 s                              |

The 1,000 requests scheduled during the stall would have waited between nearly 0 and 10 seconds. The tool saw only one of them.

The symptom is a [load test](./Load%20testing.md) reporting a p99 of a few milliseconds while users see multi-second pauses in production. A high maximum next to a low p99 is a warning sign.

To avoid it, use an open-loop generator that launches requests on a fixed schedule regardless of outstanding responses, and measure latency from the intended start time.

Production measurement can have the same blind spot. A service that starts its timer when a handler begins does not count [queueing delay](./Queueing%20delay.md) before that point. Measure at the client or load balancer as well.

_Usage:_

"The benchmark says p99 is 4 ms, but we saw a ten-second GC pause during the run."

"Then it's coordinated omission. The tool stopped sending during the pause, so rerun it with a constant arrival rate."
