---
description: How well a system's internal state can be worked out from its outputs, including during failures nobody predicted.
---

Observability is the degree to which you can work out what a system is doing internally from the data it emits. The word comes from control theory. In software, an observable system lets an engineer answer a new question about production without shipping new code to collect the answer.

It is often contrasted with [monitoring](./Monitoring.md):

| Aspect         | Monitoring                       | Observability                                         |
| -------------- | -------------------------------- | ----------------------------------------------------- |
| Questions      | Known in advance                 | Arise during an investigation                         |
| Typical output | Dashboards and [alerts](./Alert.md) | Exploratory queries                                |
| Example        | "Is the error rate above 1%?"    | "Which customers on which version see these errors?"  |
| Needs          | Predefined [metrics](./Metric.md) | Detailed, high-[cardinality](./Cardinality.md) [telemetry](./Telemetry.md) |

The symptom of low observability is a dashboard that shows errors rising while nobody can say why. The team adds a [log](./Log.md) line, deploys it, waits for the failure to recur, and repeats. Each question costs a deployment, and the [incident](./Incident.md) continues in the meantime. In an observable system, each request already carries context such as customer, version, region and enabled features, so the same question becomes a query.

The usual data sources are metrics, logs and [distributed traces](./Distributed%20trace.md), often called the three pillars. Collecting all three does not produce observability by itself. If they cannot be correlated, an engineer ends up copying timestamps between tools. A shared trace ID lets you move from a slow trace to the log lines for that exact request.

Detailed telemetry costs money to store and process. [Sampling](./Sampling.md) and retention limits keep that cost under control. Instrumentation is also ongoing work, and it is easier to add alongside a feature than during an outage. [OpenTelemetry](./OpenTelemetry.md) is a common vendor-neutral way to do it.

_Usage:_

"Errors are up on checkout, but I can't tell who's affected. Can we add a log and redeploy?"

"That's the third redeploy tonight. After this, let's put customer and version on every [span](./Span.md) so the next one is a query."
