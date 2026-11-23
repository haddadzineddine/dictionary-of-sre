---
description: Running a system at steady, realistic load for hours or days to expose problems that only appear over time.
---

A soak test runs a system at steady, realistic load for an extended period, typically hours or days, to expose problems that only appear over time. It is also called an endurance test. A short [load test](./Load%20testing.md) can pass while the system is slowly heading toward failure.

Soak tests find a specific class of defect:

| Problem                         | What you see over time                         |
| ------------------------------- | ---------------------------------------------- |
| [Memory leak](./Memory%20leak.md) | Memory floor climbs until the process is killed |
| Connection leak                 | Open connections rise until the pool runs out  |
| Disk growth                     | [Logs](./Log.md) or temp files fill the disk   |
| Data growth                     | Queries slow down as tables grow               |
| Time-based events               | Token expiry, scheduled jobs, log rotation     |

The recognisable symptom is a service that runs well after each deployment and dies after a few days. Teams that deploy often may never see the problem, because each release restarts the process and resets the clock. It then surfaces during a holiday change freeze, when the service runs uninterrupted for longer than it ever has.

Run the test at roughly typical production load. Make it longer than the longest cycle you care about. If a batch job runs daily or an access token lasts 24 hours, a soak of under a day will not exercise them.

Watch the trend instead of the level. Graph each [gauge](./Gauge.md) for memory, connections, threads and disk, and compare the first hour with the last. A flat line is healthy, and a slope gives a time to failure. In a hypothetical test where memory grows 20 MB per hour with 1.6 GB free, the process runs out after 80 hours. An 8-hour soak would not crash, but its slope reveals the problem.

_Usage:_

"It passed the load test, but in production it falls over every four days."

"An hour-long test can't catch that. Run a soak test over the weekend and watch the memory and connection graphs."
