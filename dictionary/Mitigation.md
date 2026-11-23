---
description: An action that reduces or stops user impact during an incident, even if the underlying fault is not yet fixed.
---

Mitigation is an action that reduces or stops user impact during an [incident](./Incident.md), even while the underlying fault is still present. A rollback that restores service without explaining the bug is a mitigation. The permanent fix comes afterwards.

Responders usually choose from a familiar set of options:

| Mitigation           | Use when                              | Watch for                                 |
| -------------------- | ------------------------------------- | ----------------------------------------- |
| Roll back            | A recent release lines up with onset  | Schema or data changes may not reverse    |
| Disable a feature flag, a switch that turns code off without a deploy | A new feature is at fault | Other code may depend on it |
| Drain or fail over   | One host, zone or region is bad       | The rest must absorb the traffic          |
| Shed load            | Demand exceeds capacity               | Some users get errors by design           |
| Restart              | A process is stuck or leaking         | Destroys evidence, and the fault may recur |

The common failure is spending 90 minutes establishing the root cause while users stay down, when a rollback was available in the first ten. Stop the impact first and explain it later. If an incident began right after a change, reversing that change is usually the first thing to try.

Capture evidence where it is cheap. A heap dump, thread dump or saved [log](./Log.md) excerpt taken before a restart may be the only record of the faulty state. Do not hold up recovery to collect it.

Apply one change at a time where possible, so you can tell what worked, and confirm the effect using a user-level [SLI](./SLI.md) instead of assuming.

A mitigation also needs a follow-up. Temporary measures such as extra replicas or a disabled job tend to become permanent unless someone tracks them. The time taken to reach this point is what the "restore" version of [MTTR](./MTTR.md) measures.

_Usage:_

"I think I've nearly found the bug, give me another twenty minutes."

"Users are still down. Roll back as the mitigation now, then keep digging with the pressure off."
