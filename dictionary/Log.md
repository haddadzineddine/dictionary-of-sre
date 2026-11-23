---
description: A timestamped record of a single event in a running system, read later to reconstruct what happened.
---

A log is a timestamped record of a single event in a running system, such as a request handled, a job failed or a connection closed. Logs carry more detail per event than other kinds of [telemetry](./Telemetry.md), and they are often the first place an engineer looks when a specific request went wrong.

Log lines are usually tagged with a level. Names vary by framework:

| Level | Meaning                     | Typical production handling    |
| ----- | --------------------------- | ------------------------------ |
| DEBUG | Detail for developers       | Off, or enabled briefly        |
| INFO  | Normal events               | Kept, sometimes reduced        |
| WARN  | Unexpected but handled      | Reviewed for trends            |
| ERROR | An operation failed         | Investigated                   |
| FATAL | The process cannot continue | Investigated at once           |

Logs are priced by volume, so small changes add up. One 500-byte line per request at 1,000 requests per second is 500 KB per second, or 43.2 GB per day. Adding a log line to a frequently called code path can raise the bill noticeably, and it can slow the service if writes happen synchronously.

Logs also fail in specific ways during an [incident](./Incident.md). A broken dependency can produce one error line per request and bury everything else. A process that crashes may lose its buffered lines, so the entry explaining the crash was never written. Files on a container's local disk disappear when the container is replaced, so ship logs off the host.

Free-text lines are hard to query across many machines, which [structured logging](./Structured%20logging.md) addresses. Including a [distributed trace](./Distributed%20trace.md) ID connects each line to its request. Never write secrets, tokens or personal data to logs.

Use logs to explain individual events. To count how often something happens, use a [metric](./Metric.md), which stays cheap at any traffic level.

_Usage:_

"The pod crashed and the last log line is from two minutes earlier."

"Output is buffered, and it was killed for exceeding its memory limit before flushing. Check the container's exit reason instead."
