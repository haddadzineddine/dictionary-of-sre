---
description: Writing log events as named fields, usually JSON, so they can be filtered and aggregated without regex parsing.
---

Structured logging is writing each [log](./Log.md) event as a set of named fields, usually JSON, instead of a free-text sentence. Because fields have names and types, a log system can filter and aggregate them without parsing the text.

Here is the same event written both ways:

```text
2026-03-14 10:02:11 ERROR payment failed for user 4821 after 3 retries (timeout)
```

```json
{"ts":"2026-03-14T10:02:11Z","level":"error","event":"payment_failed","user_id":4821,"retries":3,"reason":"timeout"}
```

The free-text form works until someone needs an answer across millions of lines. During an outage, an engineer writes a regex to count failures, and it silently misses half of them because another developer phrased the message differently. A small wording change can also break a saved query or dashboard with no error. With fields, the same question is a filter on `reason` grouped by `user_id`.

A few conventions keep structured logs useful:

| Practice                           | Reason                                                       |
| ---------------------------------- | ------------------------------------------------------------ |
| Use a stable event name            | Queries survive changes to message wording                   |
| Share field names across services  | `user_id` everywhere, not `uid` in one service               |
| Keep field types consistent        | A field that is sometimes a number and sometimes a string breaks indexing in many backends |
| Include the trace ID               | Connects each line to its [distributed trace](./Distributed%20trace.md) |
| Index only fields you query        | High-[cardinality](./Cardinality.md) indexes are expensive   |

Structured logs are larger per line because field names repeat, though they compress well.

Some teams extend this into wide events: one rich record per request holding everything known about it. That approach supports [observability](./Observability.md) well. Choose fields deliberately in either case, because logging a whole request object is a common way to leak passwords and personal data into [telemetry](./Telemetry.md).

_Usage:_

"My grep found 200 timeouts, but the payment team counted 900."

"Half the services word that message differently. Once they all emit structured logging with a reason field, we'll get one number."
