---
description: A checklist from Brendan Gregg that examines utilisation, saturation and errors for every resource in a system.
---

The USE method is a performance-analysis checklist created by Brendan Gregg: for every resource, check Utilisation, [Saturation](./Saturation.md) and Errors. A resource is anything with finite capacity, including CPU, memory, disks and network interfaces, as well as software limits such as connection pools.

| Letter      | Meaning                        | CPU example         | Disk example           |
| ----------- | ------------------------------ | ------------------- | ---------------------- |
| Utilisation | Share of time the resource is busy | 85% busy        | Device busy 70% of the time |
| Saturation  | Extra work waiting             | Run queue length    | I/O queue length       |
| Errors      | Count of error events          | Hardware CPU faults | Read and write errors  |

The method replaces guessing with a fixed walk through the system. When a service is slow, teams often restart it, or look only at whichever graphs already exist. Working through a list of resources covers the parts nobody thought to check.

Utilisation is easy to misread, and two cases come up repeatedly. A single-threaded process can max out one of 16 CPU cores while the host reports about 6% usage, so per-core figures matter. Averages also hide bursts: a CPU at 60% over a minute may have spent several seconds at 100% with work queued behind it.

Thread pools, file descriptors, connection pools and locks are often the actual limit. A pool of 20 connections with all 20 in use and callers waiting is saturated even when the machine looks idle.

USE complements the [RED method](./RED%20method.md). RED looks at a service from the outside and shows whether it is healthy. USE looks at the resources underneath and shows what is limiting it. A common sequence is to use RED to find the unhealthy service, USE to find the constrained resource, and then [profiling](./Profiling.md) if the constraint is CPU.

_Usage:_

"Host CPU is only at 8%, so it can't be a compute problem."

"Run through USE per core. That worker is single-threaded, and I'd expect one core pinned at 100% with a queue behind it."
