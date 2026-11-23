---
description: One named, timed operation inside a distributed trace, such as an HTTP call or a database query.
---

A span is one named, timed operation within a [distributed trace](./Distributed%20trace.md), such as handling an HTTP request, calling another service or running a database query. Spans point to their parent, and those references form the tree of the trace.

A span typically holds these fields:

| Field          | Purpose                       | Example                         |
| -------------- | ----------------------------- | ------------------------------- |
| Trace ID       | Groups spans into one trace   | `4bf92f35...`                   |
| Span ID        | Identifies this span          | `00f067aa...`                   |
| Parent span ID | Places it in the tree         | The caller's span ID            |
| Name           | Describes the operation       | `GET /users/:id`                |
| Start and end  | Give the duration             | 120 ms                          |
| Attributes     | Key-value context             | `http.response.status_code=500` |
| Status         | Records the outcome           | Error                           |
| Events         | Timestamped notes in the span | An exception was recorded       |

A remote call usually produces two spans: a client span in the caller and a server span in the callee. Comparing them is a useful diagnostic. If the client span takes 120 ms and the server span takes 40 ms, the other 80 ms went somewhere outside the handler, such as the network, waiting for a connection, or a queue in front of the server. This explains the common disagreement where the called team's dashboard shows fast responses while the calling team sees slow ones.

Keep span names low in [cardinality](./Cardinality.md). Backends group by name, so use `GET /users/:id` and place the actual ID in an attribute.

A span per network call plus any significant internal step is a common default.

Spans must be ended explicitly in most SDKs, and a span is generally exported only once it ends. Code that forgets to end a span on an error path loses it, so the failing operation is missing from the trace. Close spans in a `finally` block or its equivalent.

_Usage:_

"Our server span says 30 ms, but the frontend team swears the call takes 400."

"Compare it with their client span. A gap that size is usually connection pool wait or a queue in front of us."
