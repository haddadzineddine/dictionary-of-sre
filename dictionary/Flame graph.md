---
description: A chart of profiled call stacks where each box's width shows its share of samples, used to find expensive code.
---

A flame graph is a visualisation of [profiling](./Profiling.md) data that shows sampled call stacks, with each box's width proportional to the share of samples in which that function appeared. Brendan Gregg created the format.

| Element             | Meaning                                                  |
| ------------------- | -------------------------------------------------------- |
| Box                 | One function in the call stack                           |
| Width               | Share of samples containing this function and its callees |
| Vertical position   | Stack depth, with each callee drawn above its caller     |
| Left-to-right order | Usually alphabetical, and not related to time            |
| Colour              | Usually only separates boxes visually                    |

The common misreading is to treat the horizontal axis as time. Two boxes side by side did not necessarily run one after the other. A view that does plot time horizontally is called a flame chart, as seen in browser developer tools, and it answers a different question.

Look for wide boxes with little or nothing above them. Those functions were themselves on the CPU for that share of samples. A wide box covered by children spent its time in those children. New readers often see the broad `main` or `handleRequest` box at the bottom and conclude it is the problem, when it is only the caller of everything above it.

Width converts to a limit on savings. If `serialize` covers 35% of the graph, making it twice as fast saves at most about 17.5% of CPU.

Differential flame graphs compare two profiles, such as before and after a deployment, and colour the functions that grew or shrank. They are a direct way to investigate a performance regression.

A CPU flame graph shows only on-CPU work, so a service blocked on a database looks idle in it. An off-CPU variant covers blocking.

_Usage:_

"The flame graph shows handleRequest at 95%. So that's our slow function?"

"No, that's just the root of the stack. Look at the wide plateau near the top, it's regex compilation at 30%."
