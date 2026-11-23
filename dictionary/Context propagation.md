---
description: Passing trace identifiers and related metadata across process boundaries so separate spans join into one trace.
---

Context propagation is the passing of trace identifiers and related metadata from one operation to the next, so that work done in separate services joins the same [distributed trace](./Distributed%20trace.md). Without it, each service records its own disconnected fragment.

Over HTTP, the W3C Trace Context standard carries this in a `traceparent` header:

```text
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

| Field     | Value in the example | Meaning                                  |
| --------- | -------------------- | ---------------------------------------- |
| Version   | `00`                 | Format version                           |
| Trace ID  | `4bf92f35...`        | 32 hex characters, shared by the trace   |
| Parent ID | `00f067aa...`        | 16 hex characters, the caller's [span](./Span.md) |
| Flags     | `01`                 | The sampled bit is set                   |

Each service reads the incoming header, starts a child span and writes that span's ID into its outgoing calls. A companion `tracestate` header carries vendor-specific data.

The symptom of broken propagation is a trace that stops after two services, or a downstream service appearing as its own unrelated trace. Common causes are a proxy that strips unknown headers, a hand-written HTTP client that does not forward them, and message queues where the producer does not write context into message headers. Context can also be lost inside one process, because libraries hold the current span in thread-local or async-local storage, and work handed to a thread pool may not carry it along.

Propagation also carries the [sampling](./Sampling.md) decision, so every service keeps or drops a trace together.

Baggage is a related mechanism for application-defined key-value data that travels with the context. It is sent to every downstream service, possibly including third parties, so keep secrets and personal data out of it. Services at a public edge should generally start a new trace instead of trusting incoming headers.

_Usage:_

"The trace ends at the order service, then nothing, even though the worker clearly ran."

"Context propagation breaks at the queue. The producer isn't writing traceparent into the message headers."
