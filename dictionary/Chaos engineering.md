---
description: Deliberately injecting failures into a system in a controlled way to test whether it behaves as expected.
---

Chaos engineering is the practice of deliberately injecting failures into a system, as a controlled experiment, to test whether it behaves as expected. It grew out of work at Netflix, whose Chaos Monkey tool randomly terminated production instances.

The practice is an experiment with a defined method:

| Step | Action                 | Example                                       |
| ---- | ---------------------- | --------------------------------------------- |
| 1    | Define steady state    | Checkout success stays at or above 99.9%      |
| 2    | Form a hypothesis      | Losing one cache node will not affect checkout |
| 3    | Limit the blast radius | 1% of traffic in one zone                     |
| 4    | Set abort conditions   | Stop if the [SLI](./SLI.md) drops below target |
| 5    | Inject the fault       | Terminate a cache node                        |
| 6    | Observe and compare    | Did steady state hold?                        |
| 7    | Fix and repeat         | Address what was learned                      |

The problem it addresses is redundancy that has never been exercised. A team has a standby database and a failover procedure that has never run. On the day the primary fails, the standby turns out to have stale configuration or too little capacity.

Typical injected faults include instance termination, added network latency, packet loss, dependency errors, DNS failures, full disks and clock skew. Added latency is often more revealing than a crash, because a slow dependency ties up threads and connections while a dead one fails quickly.

Some preconditions apply. You need enough [observability](./Observability.md) to see the effect, an [SLO](./SLO.md) to define normal, and a fast way to stop. If you already expect the system to fail the experiment, fix it first. Start in staging and move to production as confidence grows, since only production has real traffic and configuration.

Experiments that affect users spend [error budget](./Error%20budget.md), so run them when budget is available. A [game day](./Game%20day.md) complements this work by exercising the people as well as the software.

_Usage:_

"We've got three replicas, so losing one is fine."

"Have we ever tried it? Let's run a chaos engineering experiment on 1% of traffic and see whether failover works."
