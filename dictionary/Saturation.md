---
description: How close a resource is to its limit, including work that must wait because the resource is already busy.
---

Saturation is how close a resource is to its limit and, past that point, how much work is waiting because the resource is busy. The [golden signals](./Golden%20signals.md) describe it as how full a service is. The [USE method](./USE%20method.md) defines it more narrowly as queued work the resource cannot yet serve.

Saturation differs from utilisation. A CPU that is 100% busy with nothing waiting is fully utilised but not saturated. The same CPU with 30 runnable threads waiting for a turn is saturated.

| Resource        | Utilisation        | Saturation signal                     |
| --------------- | ------------------ | ------------------------------------- |
| CPU             | Percent busy       | Run queue length, throttled time      |
| Memory          | Percent used       | Swapping, reclaim, out-of-memory kills |
| Connection pool | Connections in use | Callers waiting, wait time            |
| Message queue   | Not applicable     | Consumer lag, age of oldest message   |

The symptom is latency that jumps sharply while traffic has risen only a little, with no errors in the [logs](./Log.md) and CPU looking moderate. This happens because [queueing delay](./Queueing%20delay.md) grows non-linearly near capacity. A database call that normally takes 5 ms can take 500 ms because it waited for a free connection, and the database's own slow-query record shows nothing, since the query itself was fast.

A container CPU limit is enforced over short periods, by default 100 ms on Linux, so a multi-threaded service can use its quota early in a period and then be paused for the rest. Average CPU looks comfortable while latency suffers. The throttled-time [metric](./Metric.md) shows it.

Measure waiting as well as usage. To respond, add capacity, reduce the work per request, or bound queues so excess requests are rejected quickly instead of waiting indefinitely. [Capacity planning](./Capacity%20planning.md) aims to add resources before this point is reached.

_Usage:_

"DB CPU is at 30% and queries are fast, but the API is timing out."

"Check saturation on the connection pool. If all 50 are in use, requests are queueing before they ever reach the database."
