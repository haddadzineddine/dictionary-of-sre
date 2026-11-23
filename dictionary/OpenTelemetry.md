---
description: An open, vendor-neutral standard and toolset for producing, collecting and exporting traces, metrics and logs.
---

OpenTelemetry (OTel) is an open-source, vendor-neutral standard and set of tools for producing, collecting and exporting [telemetry](./Telemetry.md). It covers [distributed traces](./Distributed%20trace.md), [metrics](./Metric.md) and [logs](./Log.md). It is a Cloud Native Computing Foundation project, formed in 2019 by merging OpenTracing and OpenCensus. OTel does not store or display data. You send its output to a backend such as Jaeger, Prometheus or a commercial vendor.

The project is made of several parts:

| Component                 | Role                                                          |
| ------------------------- | ------------------------------------------------------------- |
| API                       | What application and library code calls to record telemetry   |
| SDK                       | Implements the API, including [sampling](./Sampling.md), batching and export |
| Instrumentation libraries | Add telemetry to common frameworks without hand-written code  |
| OTLP                      | The wire protocol for sending telemetry                       |
| Collector                 | A standalone service that receives, filters and forwards data |
| Semantic conventions      | Standard attribute names, such as `http.request.method`       |

OTel addresses vendor lock-in. When code calls one vendor's SDK throughout, changing vendor means re-instrumenting every service. With OTel, the instrumentation stays the same and the export destination changes in configuration. Shared naming and [context propagation](./Context%20propagation.md) also let services written in different languages join the same trace.

A Collector is commonly run in production because it separates applications from backends and gives one place to redact sensitive fields and apply sampling.

Adoption has some practical hazards. Signals and language SDKs mature at different rates, so check the stability status for your language. Automatic instrumentation can emit many [spans](./Span.md), some of them noise, and raise costs, so review what it produces. Semantic conventions have also changed over time. For example, `http.method` became `http.request.method`, so a dashboard built on the old name can go blank after an upgrade.

_Usage:_

"If we switch tracing vendors, do we have to touch every service again?"

"Not if we instrument with OpenTelemetry. We'd change the Collector's exporter and leave the application code alone."
