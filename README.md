# SRE and Observability Dictionary

A plain-English dictionary of keeping production healthy: reliability targets, telemetry, alerting, incidents, capacity and learning from failure.

Why did nobody get paged? Why is the average fine while users complain? Why did the metrics bill explode?

Each of these has a short explanation once you know the right words. Entries define the term, describe the failure it explains where there is one, and end with a short example of engineers using it.

---

## Table of contents

- **[Section 1 — Reliability Targets](#section-1--reliability-targets)**
  - [Site reliability engineering](#site-reliability-engineering)
  - [Availability](#availability)
  - [Nines](#nines)
  - [SLI](#sli)
  - [SLO](#slo)
  - [SLA](#sla)
  - [Error budget](#error-budget)
  - [Error budget policy](#error-budget-policy)
  - [Toil](#toil)
  - [MTTD](#mttd)
  - [MTTR](#mttr)
  - [MTBF](#mtbf)
- **[Section 2 — Observability](#section-2--observability)**
  - [Observability](#observability)
  - [Monitoring](#monitoring)
  - [Telemetry](#telemetry)
  - [Log](#log)
  - [Structured logging](#structured-logging)
  - [Metric](#metric)
  - [Counter](#counter)
  - [Gauge](#gauge)
  - [Histogram](#histogram)
  - [Percentile](#percentile)
  - [Cardinality](#cardinality)
  - [Distributed trace](#distributed-trace)
  - [Span](#span)
  - [Context propagation](#context-propagation)
  - [Sampling](#sampling)
  - [OpenTelemetry](#opentelemetry)
- **[Section 3 — What to Measure](#section-3--what-to-measure)**
  - [Golden signals](#golden-signals)
  - [RED method](#red-method)
  - [USE method](#use-method)
  - [Saturation](#saturation)
  - [Tail latency](#tail-latency)
  - [Synthetic monitoring](#synthetic-monitoring)
  - [Real user monitoring](#real-user-monitoring)
- **[Section 4 — Alerting and On-Call](#section-4--alerting-and-on-call)**
  - [Alert](#alert)
  - [Alert fatigue](#alert-fatigue)
  - [Symptom-based alerting](#symptom-based-alerting)
  - [Burn rate alert](#burn-rate-alert)
  - [On-call](#on-call)
  - [Runbook](#runbook)
- **[Section 5 — Incidents](#section-5--incidents)**
  - [Incident](#incident)
  - [Severity level](#severity-level)
  - [Incident commander](#incident-commander)
  - [Mitigation](#mitigation)
  - [Blameless postmortem](#blameless-postmortem)
  - [Contributing factor](#contributing-factor)
- **[Section 6 — Capacity and Performance](#section-6--capacity-and-performance)**
  - [Capacity planning](#capacity-planning)
  - [Load testing](#load-testing)
  - [Stress testing](#stress-testing)
  - [Soak test](#soak-test)
  - [Little's Law](#littles-law)
  - [Queueing delay](#queueing-delay)
  - [Coordinated omission](#coordinated-omission)
  - [Thundering herd](#thundering-herd)
  - [Memory leak](#memory-leak)
  - [Profiling](#profiling)
  - [Flame graph](#flame-graph)
- **[Section 7 — Learning from Change](#section-7--learning-from-change)**
  - [Chaos engineering](#chaos-engineering)
  - [Game day](#game-day)
  - [DORA metrics](#dora-metrics)
  - [Change failure rate](#change-failure-rate)
  - [Production readiness review](#production-readiness-review)

## Section 1 — Reliability Targets

### Site reliability engineering

Site reliability engineering (SRE) is the discipline of keeping production services reliable by applying software engineering to operations work. The term comes from Google, where Ben Treynor Sloss started an SRE team in 2003. An SRE team writes automation, sets reliability targets, responds to failures and feeds what it learns back into the software.

SRE addresses a conflict between shipping and stability. Developers are rewarded for releasing changes, operators for keeping systems steady, and each release can turn into a negotiation. SRE replaces that argument with a number: the team agrees an [SLO](#slo), and the gap between that target and 100% becomes an [error budget](#error-budget) that can be spent on risky change.

Typical SRE work falls into these areas:

| Practice           | What the team does                                                                    |
| ------------------ | ------------------------------------------------------------------------------------- |
| Set targets        | Defines [SLIs](#sli) and SLOs with product owners                                 |
| Reduce [toil](#toil) | Automates repetitive manual work                                                |
| Build visibility   | Adds [observability](#observability) so production behaviour can be explained     |
| Share [on-call](#on-call) | Responds to [alerts](#alert) and [incidents](#incident)            |
| Learn from failure | Writes [blameless postmortems](#blameless-postmortem)                           |
| Plan ahead         | Does [capacity planning](#capacity-planning) before demand arrives              |

SRE is different from renaming an operations team. A team that spends all day on tickets, with no time for engineering, is doing operations under a new title. The usual symptom is a pager load that grows in step with the number of services. Google's published guidance caps operational work at 50% of an SRE's time so the rest can go to engineering that reduces future work.

SRE overlaps with DevOps, the broader practice of developers and operators sharing responsibility for delivery. DevOps describes a culture and a set of goals. SRE is one specific way of pursuing them, built around numeric targets.

_Usage:_

"We hired two SREs and now they get every ticket nobody else wants."

"Then we've built an ops team. Give them the SLOs, cap the interrupt work, and let them automate the top sources of pages."

### Availability

Availability is the proportion of time, or of requests, in which a service does its job correctly for users. It is normally written as a percentage and often shortened to a count of [nines](#nines), so 99.9% is "three nines".

There are two common ways to calculate it:

| Method        | Formula                              | Strength                       | Weakness                                     |
| ------------- | ------------------------------------ | ------------------------------ | -------------------------------------------- |
| Time-based    | uptime / (uptime + downtime)         | Easy to explain                | Counts a 3am outage and a peak outage equally |
| Request-based | successful requests / valid requests | Reflects traffic users sent    | Says little when there is no traffic         |

The two methods can disagree about the same month. Take a hypothetical service with 10,000,000 requests in 30 days, of which 5,000 failed during one 30-minute total outage. Request-based availability is 9,995,000 / 10,000,000 = 99.95%. Time-based availability is 43,170 / 43,200 minutes = 99.93%. They differ because traffic during the outage was below the monthly average.

A service can look up while being unavailable to users. The status page is green and the health check returns 200, yet checkout fails for every customer in one region. For this reason, availability is best defined from the user's side through an [SLI](#sli), and a response that is too slow to be useful can count as a failure.

Dependencies limit what a service can reach. If a request needs two components in series, each 99.9% available, and their failures are independent, the combined figure is 0.999 x 0.999, or about 99.8%. Redundant copies raise availability only when they fail independently. Two replicas running the same bad configuration fail together.

Teams set a target through an [SLO](#slo) instead of aiming for 100%, because each extra nine costs more.

_Usage:_

"The uptime check says 100% this month, but support has a queue of failed payments."

"That probe only hits the health endpoint. Measure availability from real checkout requests and the number will drop."

### Nines

Nines are shorthand for [availability](#availability) percentages, counted by the number of 9 digits. 99.9% is three nines and 99.999% is five nines. Each added nine divides the allowed downtime by ten.

| Availability | Name                 | Downtime per 30-day month | Downtime per 365-day year |
| ------------ | -------------------- | ------------------------- | ------------------------- |
| 99%          | Two nines            | 7.2 hours                 | 3.65 days                 |
| 99.9%        | Three nines          | 43.2 minutes              | 8.76 hours                |
| 99.95%       | Three and a half     | 21.6 minutes              | 4.38 hours                |
| 99.99%       | Four nines           | 4.32 minutes              | 52.56 minutes             |
| 99.999%      | Five nines           | 25.92 seconds             | 5.26 minutes              |

The calculation is (1 - availability) x period. For three nines in a 30-day month, that is 0.001 x 43,200 minutes = 43.2 minutes.

The table explains why a target that sounds modest can be out of reach. Four nines allows 4.32 minutes of full outage per month. If paging a person, opening a laptop and finding the problem takes 15 minutes, a single [incident](#incident) breaks the target before anyone has started the fix. Targets at that level need automatic detection and automatic recovery, such as moving traffic to a healthy copy without human involvement.

Serial dependencies multiply. A request that passes through five components, each at 99.99%, gets at most 0.9999^5, or about 99.95%, assuming independent failures. A service cannot promise more nines than its critical dependencies allow unless it keeps working when they fail.

Nines also hide the shape of downtime. Ten 4-minute outages and one 40-minute outage produce the same monthly total but affect users differently, and the percentage does not say which users were hit. An [SLO](#slo) written in nines needs a defined measurement and window to mean anything.

_Usage:_

"Sales wants to promise four nines on the API."

"That's about four minutes a month, and our database failover alone takes three. We can't sign that without automated recovery."

### SLI

A service level indicator (SLI) is a measurement of one aspect of service quality as users experience it. The recommended form is a ratio: good events divided by valid events, expressed as a percentage. Because every SLI then sits on the same 0 to 100% scale, it can feed an [SLO](#slo) and an [error budget](#error-budget) in the same way.

Common SLIs by type of service:

| Service type | SLI                        | A good event is                        |
| ------------ | -------------------------- | -------------------------------------- |
| Request API  | [Availability](#availability) | A response that is not a server error |
| Request API  | Latency                    | A response in under 300 ms             |
| Data pipeline | Freshness                 | A record processed within 10 minutes   |
| Storage      | Durability                 | A stored object read back intact       |
| Batch job    | Completion                 | A scheduled run finishing on time      |

Defining an SLI takes three decisions: which events count, what makes one good, and where it is measured.

The measurement point changes the answer. A [metric](#metric) recorded inside the application misses requests that never reached it, such as those dropped by a load balancer or lost to a DNS failure. Measuring at the load balancer, or with [real user monitoring](#real-user-monitoring), sits closer to what users saw but adds noise.

Exclusions need the same care. Client errors are usually left out of valid events, but a burst of 401s from a broken login flow is a real failure, and excluding it makes a broken feature look healthy.

A weak SLI shows up as a green dashboard while users complain. CPU and memory are poor SLIs because users do not experience them directly. An average latency hides slow requests, whereas "share of requests under 300 ms" counts them. A [histogram](#histogram) with a bucket boundary at the threshold makes that count exact.

_Usage:_

"Our availability SLI says 99.99, but customers keep reporting failed uploads."

"It only counts requests that reach the app. Uploads rejected at the proxy never show up. Move the measurement to the load balancer."

### SLO

A service level objective (SLO) is a target value for an [SLI](#sli) over a defined time window. It states how reliable a service needs to be, and so also how much unreliability is acceptable.

An SLO has three parts:

| Part   | Example                                                  |
| ------ | -------------------------------------------------------- |
| SLI    | Share of checkout requests that succeed in under 500 ms  |
| Target | 99.9%                                                    |
| Window | Rolling 30 days                                          |

The gap between the target and 100% is the [error budget](#error-budget). In this example, 0.1% of checkout requests may fail or run slow before the objective is missed.

The window can be rolling or calendar-based. A rolling window always covers the last 30 days, so a bad week keeps counting until it ages out. A calendar window resets on the first of the month, which suits reporting but lets a team start fresh the day after a major outage.

Choose a target from what users tolerate and what the service has achieved recently. A target set too high means every small blip blocks releases, and teams soon start ignoring it. A target set too low reports success while users are unhappy, which appears as a healthy SLO dashboard next to a growing pile of support tickets. Review the target periodically against complaints and [incident](#incident) history.

An SLO only has an effect if missing it changes a decision. An [error budget policy](#error-budget-policy) records what the team will do when the budget runs low. Avoid 100% as a target: it cannot be met, and it leaves no room for releases or maintenance.

Keep internal SLOs tighter than any external [SLA](#sla). The difference gives the team a warning and time to react before a contract is breached.

_Usage:_

"Latency spiked for twenty minutes yesterday. Do we need to drop everything?"

"Check the SLO first. We've used 12% of the budget this window, so file a follow-up and keep the release on track."

### SLA

A service level agreement (SLA) is a contract between a provider and a customer that promises a level of service and states what happens if the provider misses it. The usual remedy is a service credit, which is a partial refund on the bill. That contractual consequence separates an SLA from an [SLO](#slo), which is an internal target.

The three related terms differ as follows:

| Term | What it is        | Example                  | If missed                                                      |
| ---- | ----------------- | ------------------------ | -------------------------------------------------------------- |
| [SLI](#sli) | A measurement | 99.93% of requests succeeded | Nothing, it is a reading                                 |
| SLO  | Internal target   | 99.9% over 30 days       | Team follows its [error budget policy](#error-budget-policy) |
| SLA  | External contract | 99.5% per calendar month | Customer may receive credits                                   |

The SLO is normally set tighter than the SLA so the team gets a warning before money is at stake. With the numbers above, in a 30-day month, the SLO allows 43.2 minutes of full outage and the SLA allows 3.6 hours. A team that has just exhausted its SLO still has about 2.9 hours of further downtime before the contract is breached.

The definitions in the contract decide what counts. Many SLAs exclude scheduled maintenance, customer-caused failures and events outside the provider's control, and some measure [availability](#availability) in their own way, such as counting a minute as down only above an error threshold. Customers often have to file a claim with evidence before a deadline. A common surprise follows a long provider outage: the credit covers a small part of one bill and none of the revenue lost.

Your own SLA is limited by those of your dependencies. If you promise 99.95% while relying on a provider that guarantees 99.9%, your business carries the difference. Check dependency terms before signing.

_Usage:_

"The customer wants 99.99% in the contract. We hit that last quarter."

"One good quarter isn't an SLA. Our SLO is 99.9%, so offer 99.5% and keep a margin before credits start."

### Error budget

An error budget is the amount of failure an [SLO](#slo) permits during its window. It equals 100% minus the SLO, so a 99.9% SLO has a 0.1% error budget.

A hypothetical service shows how the percentage becomes a usable number:

| Input                            | Value              |
| -------------------------------- | ------------------ |
| SLO                              | 99.9% over 30 days |
| Budget as a share                | 0.1%               |
| Expected requests in the window  | 10,000,000         |
| Budget in failed requests        | 10,000             |
| Budget as time at total outage   | 43.2 minutes       |

If a bad deployment fails 4,000 requests, 40% of the budget is gone and 6,000 failures remain. Partial failures spend budget more slowly. At steady traffic, 5% of requests failing for one hour costs the same as a total outage of three minutes, because 0.05 x 60 = 3.

The budget gives product and engineering one shared number for decisions about risk. Without it, teams argue about whether a release feels too dangerous, and the outcome depends on opinion. With it, budget remaining means the team can keep shipping, and a nearly empty budget means effort moves to stability. An [error budget policy](#error-budget-policy) writes those consequences down in advance.

Every user-visible failure spends the budget, whatever the cause: bad deployments, dependency outages, traffic spikes and planned maintenance. Tracking spend by cause shows which fix would return the most budget.

With a rolling window, old failures age out and the budget recovers gradually. A [burn rate alert](#burn-rate-alert) watches how quickly the budget is being spent, so a fast failure pages someone long before the budget is empty. A budget that is never touched for several quarters is also information: the SLO may be stricter than users need, or the team may be releasing too cautiously.

_Usage:_

"Can we ship the new auth flow on Thursday?"

"Monday's outage used 85% of the error budget. Let's hold it until the window recovers, or release it behind a flag to 1% of users."

### Error budget policy

An [error budget](#error-budget) policy is a written agreement stating what a team does as that budget is consumed. Product and engineering leads agree it before it is needed, so a decision to slow releases is not renegotiated during a bad month.

Without a policy, an [SLO](#slo) has no consequence. The familiar pattern is a team that misses its target three months running, notes this in a review, and keeps shipping at the same pace because nobody agreed what a miss should change.

A policy is usually a ladder of responses. The thresholds below are an example, not a standard:

| Budget consumed | Response                                                        |
| --------------- | --------------------------------------------------------------- |
| Under 50%       | Normal releases                                                 |
| 50% to 75%      | Review recent failures and roll out risky changes more slowly   |
| 75% to 100%     | Low-risk changes only, with extra review of each release        |
| Over 100%       | Freeze feature releases and prioritise reliability work until the service is back within SLO |

A useful policy also answers the questions that come up under pressure. It says which changes are exempt from a freeze, typically security and legal fixes and the fix for the outage itself. It says whether failures caused by an external dependency count. It sets a trigger for review, such as requiring a [blameless postmortem](#blameless-postmortem) for any single [incident](#incident) that consumes more than 20% of the budget. It also names who can override the policy and how that decision is recorded.

A freeze redirects engineering effort toward whatever burned the budget. If freezes keep blocking work for months, the target may be unrealistic for the current system. In that case, renegotiate the SLO using the data, or fund the reliability work, instead of quietly ignoring the policy.

_Usage:_

"Product says the launch can't slip, but we're at 110% of budget."

"The error budget policy they signed says feature freeze. If they want an exception, the VP records it and we accept the risk explicitly."

### Toil

Toil is operational work that is manual, repetitive, automatable, reactive, without lasting value, and growing in proportion to the service. Restarting a stuck worker every morning, resizing disks by hand and rotating a certificate from a checklist are typical examples. The term is used in [site reliability engineering](#site-reliability-engineering) to separate this work from engineering that improves the system.

A task is more likely to be toil the more of these tests it meets:

| Test              | Question                                          |
| ----------------- | ------------------------------------------------- |
| Manual            | Does a person have to run it?                     |
| Repetitive        | Has the team done this many times before?         |
| Automatable       | Could a machine do it equally well?               |
| Tactical          | Is it driven by interrupts instead of a plan?     |
| No lasting value  | Is the service unchanged once it is done?         |
| Scales with growth | Does twice the traffic mean twice the work?      |

Not all unpleasant work is toil. Investigating a new kind of failure is engineering, because it produces knowledge the team did not have.

The symptom of too much toil is a team that is busy all week while nothing improves. Project work keeps slipping, [on-call](#on-call) shifts are spent clearing tickets, and each new service means asking for more headcount. Google's SRE guidance caps operational work at 50% of an engineer's time for this reason.

Start by measuring. Record hours by category for a few weeks, then address the largest item. Prefer removing the need over scripting it: fix the bug that makes the worker stick instead of automating the restart. A [runbook](#runbook) followed identically every time is a good candidate for automation. Compare costs before building anything. A 30-minute weekly task takes 26 hours a year, so automation that takes 16 hours to build pays back in 32 weeks.

_Usage:_

"I spent half of on-call manually clearing the stuck export queue again."

"That's toil. Let's find why it sticks this sprint, because a cron job to clear it just hides the bug."

### MTTD

Mean time to detect (MTTD) is the average time between the start of an [incident](#incident) and its detection. Detection is usually an [alert](#alert) firing. It can also be a customer ticket, which means the team's [monitoring](#monitoring) missed the problem.

The mean is easily distorted by one case. If four incidents were detected after 2, 3, 5 and 30 minutes, the mean is 10 minutes and the median is 4. Report the median and the longest case alongside the mean, particularly when the sample is small.

Several things set a lower limit on detection time:

| Source of delay    | Example                                                        |
| ------------------ | -------------------------------------------------------------- |
| Collection         | A [metric](#metric) scraped every 60 seconds               |
| Pipeline lag       | [Logs](#log) that take minutes to become searchable        |
| Alert window       | A rule that requires a condition to hold for 10 minutes        |
| Missing coverage   | A failure no check measures, found only when users complain    |

The recognisable symptom of poor detection is learning about an outage from support, a sales colleague or social media while every dashboard still looks normal. [Symptom-based alerting](#symptom-based-alerting) closes much of this gap by watching what users experience, and [synthetic monitoring](#synthetic-monitoring) can catch failures on paths that have little traffic at that hour.

Lowering detection time has a cost. Tighter thresholds and shorter windows fire on more harmless blips, which feeds [alert fatigue](#alert-fatigue). A [burn rate alert](#burn-rate-alert) handles this trade-off by detecting severe failures quickly and small ones more slowly.

MTTD also has two measurement hazards. The start time is often established afterwards from graphs, so teams need a consistent rule for recording it. And the metric covers only incidents that were noticed. A failure nobody detected does not appear in the average, so a low MTTD can coexist with blind spots.

_Usage:_

"Our MTTD looks great this quarter, under three minutes."

"Only for the incidents that paged. The two that customers reported first aren't timed at all, so fix how we record those."

### MTTR

MTTR is an average [incident](#incident) duration, and the R stands for different things depending on who is speaking. Two teams can report the same acronym while measuring different intervals, so a comparison between them may mean nothing until the definition is agreed.

| Expansion                  | The clock stops when                         |
| -------------------------- | -------------------------------------------- |
| Mean time to respond       | Someone starts working on the problem        |
| Mean time to restore       | Users have working service again             |
| Mean time to repair        | The faulty component is fixed                |
| Mean time to resolve       | The incident is closed, including follow-up  |

Time to restore is usually the useful version for reliability, because it matches the period users were affected. A team that restores service in 10 minutes through a [mitigation](#mitigation) and ships the permanent fix three days later has a restore time of 10 minutes and a resolve time of three days.

Averages describe incident durations poorly, because most incidents are short and a few are very long. Take six hypothetical incidents lasting 10, 15, 20, 25, 30 and 300 minutes. The mean is about 67 minutes and the median is 22.5. Without the 300-minute outage the mean would be 20, so one incident more than tripled it with no change in how the team works. Quarter-on-quarter MTTR can move this way from chance alone. Report the median and the longest incident, and read the individual timelines.

MTTR can also move the wrong way when things improve. If a team eliminates its short, easy incidents, only the long ones remain and the average rises.

To reduce restore time, split it into stages: [MTTD](#mttd), time to diagnose and time to mitigate. Then work on the stage that dominates. Together with [MTBF](#mtbf), restore time determines steady-state [availability](#availability).

_Usage:_

"Leadership wants MTTR down 20% by next quarter."

"Which R? And with nine incidents a quarter, one bad night swings the mean. Let's target median time to restore instead."

### MTBF

Mean time between failures (MTBF) is the average working time between one failure of a repairable system and the next. It is calculated as total operating time divided by the number of failures. Some sources measure from the start of one failure to the start of the next, which includes repair time, so check the definition before comparing figures.

Take a hypothetical service over a 30-day month, or 720 hours, with three failures that each take 2 hours to restore. Downtime is 6 hours and operating time is 714 hours, so MTBF is 714 / 3 = 238 hours and [MTTR](#mttr) is 2 hours. Steady-state [availability](#availability) is MTBF / (MTBF + MTTR) = 238 / 240, or about 99.17%.

That formula shows the two ways to raise availability:

| Change                   | MTBF  | MTTR   | Availability |
| ------------------------ | ----- | ------ | ------------ |
| Baseline                 | 238 h | 2 h    | 99.17%       |
| Failures half as often   | 476 h | 2 h    | 99.58%       |
| Recovery four times faster | 238 h | 0.5 h | 99.79%      |

In this example, faster recovery gains more than halving the failure rate.

MTBF is commonly misread on hardware spec sheets. A disk rated at 1,000,000 hours will not last 114 years. The figure describes a failure rate across a large population during its service life. A fleet of 1,000 such disks running for a year accumulates 8,760,000 disk-hours, so you should expect about 8.76 failures that year.

The [metric](#metric) assumes a roughly constant failure rate, which suits hardware better than software. Software failures tend to cluster around changes, so a service can run quietly for weeks and then have three [incidents](#incident) in a day after a release. For services, an [SLO](#slo) usually says more about what users experienced.

_Usage:_

"The vendor quotes a million-hour MTBF, so we don't need spare drives, right?"

"With 2,000 drives that's roughly 18 failures a year. Keep the spares and make sure rebuilds are automatic."

## Section 2 — Observability

### Observability

Observability is the degree to which you can work out what a system is doing internally from the data it emits. The word comes from control theory. In software, an observable system lets an engineer answer a new question about production without shipping new code to collect the answer.

It is often contrasted with [monitoring](#monitoring):

| Aspect         | Monitoring                       | Observability                                         |
| -------------- | -------------------------------- | ----------------------------------------------------- |
| Questions      | Known in advance                 | Arise during an investigation                         |
| Typical output | Dashboards and [alerts](#alert) | Exploratory queries                                |
| Example        | "Is the error rate above 1%?"    | "Which customers on which version see these errors?"  |
| Needs          | Predefined [metrics](#metric) | Detailed, high-[cardinality](#cardinality) [telemetry](#telemetry) |

The symptom of low observability is a dashboard that shows errors rising while nobody can say why. The team adds a [log](#log) line, deploys it, waits for the failure to recur, and repeats. Each question costs a deployment, and the [incident](#incident) continues in the meantime. In an observable system, each request already carries context such as customer, version, region and enabled features, so the same question becomes a query.

The usual data sources are metrics, logs and [distributed traces](#distributed-trace), often called the three pillars. Collecting all three does not produce observability by itself. If they cannot be correlated, an engineer ends up copying timestamps between tools. A shared trace ID lets you move from a slow trace to the log lines for that exact request.

Detailed telemetry costs money to store and process. [Sampling](#sampling) and retention limits keep that cost under control. Instrumentation is also ongoing work, and it is easier to add alongside a feature than during an outage. [OpenTelemetry](#opentelemetry) is a common vendor-neutral way to do it.

_Usage:_

"Errors are up on checkout, but I can't tell who's affected. Can we add a log and redeploy?"

"That's the third redeploy tonight. After this, let's put customer and version on every [span](#span) so the next one is a query."

### Monitoring

Monitoring is the practice of collecting predefined measurements from a system, displaying them and raising an [alert](#alert) when they cross a threshold. It answers questions known in advance: is the service up, what is the error rate, is the disk filling.

Monitoring can look at a system from two positions, and teams generally use both:

| Approach  | Measures from                                                       | Sees                                  | Misses                                    |
| --------- | ------------------------------------------------------------------- | ------------------------------------- | ----------------------------------------- |
| White-box | Inside, through emitted [metrics](#metric)                      | Queue depth, internals, early warning | Failures that stop the service reporting  |
| Black-box | Outside, as a client would, such as [synthetic monitoring](#synthetic-monitoring) | What a user would get | Why it is failing                         |

Monitoring can fail silently. If the monitoring system goes down along with what it watches, no alert fires and the dashboard shows a gap. A rule such as "error rate above 1%" also stays quiet when a service stops reporting altogether, because missing data does not satisfy the condition. An engineer then reads a flat line as a quiet night.

Three measures reduce that risk. Run monitoring on infrastructure separate from the systems it observes. Alert explicitly on missing data. Add a dead man's switch, which is an alert that fires continuously to an external service, and that service pages someone if the signal ever stops.

Dashboards have their own failure mode. Teams accumulate dozens that nobody opens until an [incident](#incident), when nobody knows which one matters. A small set of service dashboards built around the [golden signals](#golden-signals) gives [on-call](#on-call) engineers a consistent place to start.

Monitoring covers only the failure modes someone thought to measure. [Observability](#observability) extends this to questions nobody anticipated. The two work together: monitoring reports that something is wrong, and observability helps explain why.

_Usage:_

"The checkout graph was flat all night, so I assumed things were quiet."

"Flat at zero means the exporter died. Add an alert on missing data so a silent gap pages someone."

### Telemetry

Telemetry is the data a running system emits about its own behaviour. Engineers use it to understand production without attaching a debugger to live machines. The common signals differ in what they record and what makes them expensive:

| Signal                                       | Records                     | Useful for                       | Cost driver                                   |
| -------------------------------------------- | --------------------------- | -------------------------------- | --------------------------------------------- |
| [Metrics](#metric)                       | Numbers over time           | Trends and [alerts](#alert)  | Label combinations, or [cardinality](#cardinality) |
| [Logs](#log)                             | Individual events           | Detail on what happened          | Volume                                        |
| [Traces](#distributed-trace)           | The path of one request     | Where time went across services  | [Span](#span) volume, unless [sampled](#sampling) |
| Profiles, from [profiling](#profiling)   | Where code spends resources | Finding expensive functions      | Sample frequency and retention                |

Telemetry passes through a pipeline. Code is instrumented to emit it, an agent or collector gathers it, a processing stage filters and aggregates it, and a backend stores it for queries. Each stage can delay, drop or alter data.

Two problems are common. The first is cost: the [observability](#observability) bill grows faster than traffic because someone added a high-cardinality label or left debug logging on in production. The second is that telemetry degrades when it is needed. During an [incident](#incident), error logs surge, the pipeline backs up, and data from the relevant minutes arrives late or gets dropped. Buffering, rate limits on noisy sources and infrastructure separate from the observed systems all reduce this risk.

Telemetry can also leak sensitive data. Tokens, email addresses and payment details end up in log lines and span attributes when code records whole objects. Remove them at the source and again in the collector, because deleting data from retained stores is harder than never sending it.

Decide what to collect from the questions you need to answer. Data that no dashboard, alert or investigation uses is only a cost. [OpenTelemetry](#opentelemetry) provides a common format for producing and shipping these signals.

_Usage:_

"Why did the observability bill double this month?"

"Someone left debug logging on in the payments service. That one change accounts for most of the ingested telemetry."

### Log

A log is a timestamped record of a single event in a running system, such as a request handled, a job failed or a connection closed. Logs carry more detail per event than other kinds of [telemetry](#telemetry), and they are often the first place an engineer looks when a specific request went wrong.

Log lines are usually tagged with a level. Names vary by framework:

| Level | Meaning                     | Typical production handling    |
| ----- | --------------------------- | ------------------------------ |
| DEBUG | Detail for developers       | Off, or enabled briefly        |
| INFO  | Normal events               | Kept, sometimes reduced        |
| WARN  | Unexpected but handled      | Reviewed for trends            |
| ERROR | An operation failed         | Investigated                   |
| FATAL | The process cannot continue | Investigated at once           |

Logs are priced by volume, so small changes add up. One 500-byte line per request at 1,000 requests per second is 500 KB per second, or 43.2 GB per day. Adding a log line to a frequently called code path can raise the bill noticeably, and it can slow the service if writes happen synchronously.

Logs also fail in specific ways during an [incident](#incident). A broken dependency can produce one error line per request and bury everything else. A process that crashes may lose its buffered lines, so the entry explaining the crash was never written. Files on a container's local disk disappear when the container is replaced, so ship logs off the host.

Free-text lines are hard to query across many machines, which [structured logging](#structured-logging) addresses. Including a [distributed trace](#distributed-trace) ID connects each line to its request. Never write secrets, tokens or personal data to logs.

Use logs to explain individual events. To count how often something happens, use a [metric](#metric), which stays cheap at any traffic level.

_Usage:_

"The pod crashed and the last log line is from two minutes earlier."

"Output is buffered, and it was killed for exceeding its memory limit before flushing. Check the container's exit reason instead."

### Structured logging

Structured logging is writing each [log](#log) event as a set of named fields, usually JSON, instead of a free-text sentence. Because fields have names and types, a log system can filter and aggregate them without parsing the text.

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
| Include the trace ID               | Connects each line to its [distributed trace](#distributed-trace) |
| Index only fields you query        | High-[cardinality](#cardinality) indexes are expensive   |

Structured logs are larger per line because field names repeat, though they compress well.

Some teams extend this into wide events: one rich record per request holding everything known about it. That approach supports [observability](#observability) well. Choose fields deliberately in either case, because logging a whole request object is a common way to leak passwords and personal data into [telemetry](#telemetry).

_Usage:_

"My grep found 200 timeouts, but the payment team counted 900."

"Half the services word that message differently. Once they all emit structured logging with a reason field, we'll get one number."

### Metric

A metric is a numeric measurement of a system recorded over time. Each metric has a name and a set of labels, and every distinct combination of label values forms its own time series of timestamped numbers:

```text
http_requests_total{method="GET", route="/checkout", status="500"} 1027
```

Three metric types cover most needs:

| Type                         | Behaviour                                | Example           |
| ---------------------------- | ---------------------------------------- | ----------------- |
| [Counter](#counter)      | Only goes up, resets on restart          | Requests served   |
| [Gauge](#gauge)          | Goes up and down                         | Memory in use     |
| [Histogram](#histogram)  | Counts values into buckets               | Request duration  |

Metrics are cheap because storage grows with the number of time series and not with traffic. A counter takes the same space at 10 requests per second as at 10,000. That makes metrics suitable for [alerts](#alert) and for trends over months.

The saving comes from discarding detail. A metric can report that 2% of requests failed, but not which requests. A breakdown is only possible if the label already exists. A common symptom is an [incident](#incident) affecting one large customer that no graph can show, because customer is not a label. Adding customer ID as a label is usually the wrong fix because of [cardinality](#cardinality). Per-request detail belongs in [logs](#log) or [distributed traces](#distributed-trace).

Metrics reach storage by being pulled or pushed. Prometheus, for example, scrapes an endpoint on an interval, while other systems receive pushed data. Either way, resolution limits what you can see: a value sampled every 60 seconds cannot show a 5-second spike.

Aggregation needs care. Averages hide outliers, so latency is better read through [percentiles](#percentile) calculated from histograms. Put the unit in the metric name, as in `request_duration_seconds`, so nobody compares milliseconds with seconds on one graph.

_Usage:_

"Can you show me error rate for just the Acme account?"

"Not from this metric, there's no customer label and we can't add one at that cardinality. Filter the traces by account instead."

### Counter

A counter is a [metric](#metric) whose value only increases, apart from resetting to zero when the process restarts. Requests served, errors returned and bytes sent are typical counters.

The raw value is rarely useful, because "1,204,331 requests since start" says little about the present. Counters are read as a rate: take the change over a window and divide by the window's length. A counter that moves from 1,000 to 1,600 over 60 seconds is running at 10 per second.

Restarts need handling, as this hypothetical series shows:

| Time | Raw value | What happened                 |
| ---- | --------- | ----------------------------- |
| 0 s  | 1,000     | Normal                        |
| 15 s | 1,150     | 150 new events                |
| 30 s | 40        | Process restarted, counted 40 |
| 45 s | 190       | 150 new events                |

Plain subtraction between 15 s and 30 s gives -1,110, which appears on a graph as a sharp negative spike. Query functions built for counters, such as `rate()` in Prometheus, treat any drop as a reset and count the 40 as new events. Increments made between the last scrape and the crash are still lost.

A counter tolerates missed scrapes better than a [gauge](#gauge) reporting a current rate. If one sample is lost, the next still includes every increment since, so the total over the window stays correct. This is why event counts are stored as counters.

Counters have a less obvious failure with new label values. If a series for `status="500"` does not exist until the first error, there is no earlier sample to compare it with, and `rate()` misses that first increment. An [alert](#alert) on rare errors may then stay silent. Initialise known label combinations to zero at startup.

_Usage:_

"The error graph dropped to minus a thousand at 14:05."

"Someone graphed a raw delta on a counter and the pod restarted. Use rate so resets are handled."

### Gauge

A gauge is a [metric](#metric) that reports a current value, which can rise or fall. Memory in use, queue depth, open connections and temperature are typical gauges. Reading one tells you the level at that moment.

It differs from a [counter](#counter) in several ways:

| Aspect        | Gauge                         | Counter                             |
| ------------- | ----------------------------- | ----------------------------------- |
| Direction     | Up and down                   | Up only, except on reset            |
| Read as       | Current value                 | Rate of change                      |
| Example       | 37 messages queued            | 1,204 messages processed            |
| Missed sample | That moment's value is lost   | Next sample still has the increments |

The weakness of a gauge is that it is only seen when sampled. A scraper recording every 15 seconds knows nothing about the values in between. A memory spike lasting three seconds may never appear on the graph even if it got the process killed, which produces a familiar confusion: an out-of-memory crash on a chart that shows plenty of headroom. Where available, record a peak value since the last scrape, and check the container's exit reason.

Aggregating gauges takes thought. Averaging memory across instances hides the one about to crash, so graph the maximum as well.

Gauges are the usual way to measure [saturation](#saturation): connections in use against pool size, disk used, queue length. For resources that fill up, [alert](#alert) on the trend. A prediction that a disk will fill within four hours is more useful than a fixed 90% threshold, which leaves 1 TB free on a 10 TB disk and 10 GB on a 100 GB disk.

Do not apply a counter function such as Prometheus `rate()` to a gauge. It reads every dip as a reset and returns wrong values. Use `deriv()` or `delta()` instead.

_Usage:_

"Memory never went above 70% on the dashboard, but the pod got OOM-killed."

"That gauge is scraped every 30 seconds, so it can miss a short spike. Check the peak from the container stats."

### Histogram

A histogram is a [metric](#metric) that counts observations into buckets by value. Request duration is the usual example: each request increments the bucket covering how long it took. Keeping counts instead of every value lets you estimate a [percentile](#percentile) at a fixed storage cost.

Here is a hypothetical histogram of 1,000 requests, using cumulative buckets as Prometheus does:

| Bucket (at or under) | Cumulative count |
| -------------------- | ---------------- |
| 100 ms               | 900              |
| 250 ms               | 980              |
| 500 ms               | 995              |
| 1,000 ms             | 1,000            |

To estimate p99, find request number 990, which falls in the 250 to 500 ms bucket. Linear interpolation gives 250 + ((990 - 980) / (995 - 980)) x 250, or about 417 ms. The true value could be anywhere from 250 to 500 ms, so bucket boundaries set the accuracy.

Poorly chosen buckets produce a recognisable symptom. If more than 1% of requests exceed the highest finite boundary, the reported p99 flatlines at that boundary, even as real latency keeps climbing. Choose boundaries that cover both normal and failing behaviour, and place one at your [SLO](#slo) threshold so the count of good requests is exact.

Histograms can be combined across instances by adding counts bucket by bucket, provided the boundaries match. Percentiles already calculated on each instance cannot be merged in this way.

Each bucket is its own time series, multiplied by every label combination, which makes histograms a common source of [cardinality](#cardinality) growth. In the classic Prometheus format, 12 buckets across 1,000 label combinations produce 12,000 series, plus 2,000 more for each combination's sum and count. Native Prometheus histograms and [OpenTelemetry](#opentelemetry) exponential histograms choose their buckets automatically, which reduces this cost.

_Usage:_

"p99 has sat at exactly 10 seconds since the [incident](#incident) started. Suspicious, right?"

"10 is our top histogram bucket. Everything slower lands in the overflow bucket, so real latency could be much higher."

### Percentile

A percentile is the value at or below which a given share of measurements fall. Latency is commonly reported as p50 (the median), p95 and p99. A p99 of 800 ms means 99% of requests took 800 ms or less and 1% took longer.

Percentiles are used because averages hide the shape of a distribution. Take 100 hypothetical requests, where 98 take 100 ms and 2 take 10,000 ms:

| Statistic | Value     |
| --------- | --------- |
| Mean      | 298 ms    |
| p50       | 100 ms    |
| p95       | 100 ms    |
| p99       | 10,000 ms |

The mean of 298 ms describes no real request. This is the situation behind a dashboard showing a healthy average while users report timeouts: the slow requests are real, but the average dilutes them.

Percentiles cannot be averaged. If one instance has a p99 of 100 ms and another has 900 ms, the combined p99 is not 500 ms. It depends on how much traffic each handled and on the full distributions. The same error occurs when per-minute p99s are averaged into an hourly figure. To get a correct result, merge the underlying [histograms](#histogram) and calculate the percentile from the combined counts.

Sample size limits what a percentile can tell you. With 50 requests in a window, p99 is effectively the single slowest request, so the line jumps around at low traffic. Check the request count before reacting to a spike.

A high percentile also affects more users than its number suggests. If a page makes 20 independent requests, the chance that at least one is slower than p99 is 1 - 0.99^20, or about 18%. This effect is a main reason to care about [tail latency](#tail-latency), and why an [SLI](#sli) for latency is usually defined by a threshold or percentile.

_Usage:_

"Average latency is 120 ms. I don't see what customers are complaining about."

"Look at p99, it's over four seconds. The average is hiding the slowest one in a hundred requests."

### Cardinality

Cardinality is the number of distinct values a field can take. For a [metric](#metric), it usually means the number of distinct time series, because each unique combination of label values is stored and indexed separately.

Labels multiply. Here is how a hypothetical request metric grows as labels are added:

| Label added | Distinct values | Possible series |
| ----------- | --------------- | --------------- |
| method      | 5               | 5               |
| route       | 20              | 100             |
| status      | 6               | 600             |
| instance    | 50              | 30,000          |
| user_id     | 100,000         | 3,000,000,000   |

The last row is an upper bound, since not every user hits every route on every instance. Even a small fraction of it is far more than the first four labels produce.

The symptom is a metrics bill that jumps after a harmless-looking one-line change, or a metrics server that runs out of memory and takes [monitoring](#monitoring) down with it. Common causes are user IDs, request IDs, full URLs containing IDs, raw error messages and timestamps used as label values. Pod names add churn, because each rollout creates new names and therefore new series.

Several fixes apply:

| Fix                | Example                                                  |
| ------------------ | -------------------------------------------------------- |
| Use route templates | `/users/:id` in place of `/users/4821`                  |
| Group values       | `5xx` in place of each status code                       |
| Drop labels        | Remove them in the collection pipeline                   |
| Set limits         | Reject or cap series per service                         |
| Move the detail    | Put per-user data in [logs](#log) or [traces](#distributed-trace) |

A [histogram](#histogram) multiplies the count again, because each bucket is its own series.

High cardinality is not bad in itself. Asking "which customer is affected?" requires a customer ID somewhere, and that detail supports [observability](#observability). The question is where to store it. Time-series databases handle it poorly, while log and trace stores are built for it.

_Usage:_

"Prometheus got OOM-killed right after the last deploy."

"Someone added the full request path as a label. Every order ID became a new series, so that's a cardinality explosion."

### Distributed trace

A distributed trace is a record of one request's path through the services that handled it. It is made of [spans](#span), each representing a timed operation, joined by a shared trace ID into a tree.

A trace view for a hypothetical slow checkout looks like this:

```text
POST /checkout       [==============================] 820 ms
  auth.verify        [==]                              40 ms
  cart.load             [===]                          60 ms
  payment.charge            [======================]  640 ms
    fraud.score             [==================]      520 ms
```

The tree answers a question that per-service dashboards struggle with. When a request is slow and it crosses twelve services, each team can look at its own graphs, see nothing unusual and conclude the problem is elsewhere. Here the trace shows that checkout is slow because the payment step spends most of its time waiting on fraud scoring.

Traces also expose patterns that are hard to see in aggregate. Calls that could run in parallel appear one after another. Retries show up as repeated spans. The N+1 query problem, where code runs one query per item in a list, appears as a long staircase of identical database spans.

Tracing depends on every service in the path. Each must be instrumented and must forward the trace ID through [context propagation](#context-propagation). If one service in the middle does not, the trace splits into disconnected fragments. Unexplained gaps in a timeline usually mean uninstrumented work, such as time spent waiting in a queue.

Storing every trace is expensive, so most systems use [sampling](#sampling). The consequence is that the trace for the specific failed request you are investigating may not have been kept. A trace explains one request, so use a [metric](#metric) to find out how common the behaviour is. [OpenTelemetry](#opentelemetry) is the common standard for producing traces.

_Usage:_

"Search is slow but the search service's own latency looks flat."

"Pull a distributed trace for a slow request. I'd bet the time is going to the ranking call before search even starts."

### Span

A span is one named, timed operation within a [distributed trace](#distributed-trace), such as handling an HTTP request, calling another service or running a database query. Spans point to their parent, and those references form the tree of the trace.

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

Keep span names low in [cardinality](#cardinality). Backends group by name, so use `GET /users/:id` and place the actual ID in an attribute.

A span per network call plus any significant internal step is a common default.

Spans must be ended explicitly in most SDKs, and a span is generally exported only once it ends. Code that forgets to end a span on an error path loses it, so the failing operation is missing from the trace. Close spans in a `finally` block or its equivalent.

_Usage:_

"Our server span says 30 ms, but the frontend team swears the call takes 400."

"Compare it with their client span. A gap that size is usually connection pool wait or a queue in front of us."

### Context propagation

Context propagation is the passing of trace identifiers and related metadata from one operation to the next, so that work done in separate services joins the same [distributed trace](#distributed-trace). Without it, each service records its own disconnected fragment.

Over HTTP, the W3C Trace Context standard carries this in a `traceparent` header:

```text
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

| Field     | Value in the example | Meaning                                  |
| --------- | -------------------- | ---------------------------------------- |
| Version   | `00`                 | Format version                           |
| Trace ID  | `4bf92f35...`        | 32 hex characters, shared by the trace   |
| Parent ID | `00f067aa...`        | 16 hex characters, the caller's [span](#span) |
| Flags     | `01`                 | The sampled bit is set                   |

Each service reads the incoming header, starts a child span and writes that span's ID into its outgoing calls. A companion `tracestate` header carries vendor-specific data.

The symptom of broken propagation is a trace that stops after two services, or a downstream service appearing as its own unrelated trace. Common causes are a proxy that strips unknown headers, a hand-written HTTP client that does not forward them, and message queues where the producer does not write context into message headers. Context can also be lost inside one process, because libraries hold the current span in thread-local or async-local storage, and work handed to a thread pool may not carry it along.

Propagation also carries the [sampling](#sampling) decision, so every service keeps or drops a trace together.

Baggage is a related mechanism for application-defined key-value data that travels with the context. It is sent to every downstream service, possibly including third parties, so keep secrets and personal data out of it. Services at a public edge should generally start a new trace instead of trusting incoming headers.

_Usage:_

"The trace ends at the order service, then nothing, even though the worker clearly ran."

"Context propagation breaks at the queue. The producer isn't writing traceparent into the message headers."

### Sampling

Sampling is keeping a subset of [telemetry](#telemetry) and discarding the rest to control cost. It is applied mainly to [distributed traces](#distributed-trace) and also to high-volume [logs](#log).

The common strategies differ in when they decide:

| Strategy     | Decides                    | Keeps                                  | Trade-off                                       |
| ------------ | -------------------------- | -------------------------------------- | ----------------------------------------------- |
| Head-based   | At the start of a request  | A fixed random share                   | Cheap, but blind to the outcome                 |
| Tail-based   | After the trace completes  | Errors, slow traces, a random baseline | Must buffer all [spans](#span) until it decides |
| Rate-limited | As requests arrive         | Up to a fixed number per second        | Bounded cost, but the kept share varies with traffic |

Head-based sampling loses rare failures. Take a hypothetical service handling 2,000 requests per second with a 0.1% error rate, which is 2 failures per second. Sampling 1% keeps 20 traces per second, of which on average 0.02 are failures, or about one failed trace every 50 seconds. The symptom is searching for a customer's failed request ID and finding that no trace was kept.

Tail-based sampling addresses this at a cost. A collector must hold the spans of in-flight traces in memory, and all spans of a trace must reach the same collector instance.

Decisions need to be consistent across services. The decision is normally made once and passed along by [context propagation](#context-propagation). If two services each sampled 10% independently, only about 1% of traces would be complete.

Sampled data also needs reweighting when counted. At 1-in-100, each kept trace stands for 100 requests, so raw counts understate traffic. Compute error rates and [SLO](#slo) performance from unsampled [metrics](#metric), and use sampled traces to explain individual behaviour.

_Usage:_

"The customer gave us a request ID, but there's no trace for it."

"We head-sample at 1%, so it probably got dropped. Let's move to tail sampling so errors are always kept."

### OpenTelemetry

OpenTelemetry (OTel) is an open-source, vendor-neutral standard and set of tools for producing, collecting and exporting [telemetry](#telemetry). It covers [distributed traces](#distributed-trace), [metrics](#metric) and [logs](#log). It is a Cloud Native Computing Foundation project, formed in 2019 by merging OpenTracing and OpenCensus. OTel does not store or display data. You send its output to a backend such as Jaeger, Prometheus or a commercial vendor.

The project is made of several parts:

| Component                 | Role                                                          |
| ------------------------- | ------------------------------------------------------------- |
| API                       | What application and library code calls to record telemetry   |
| SDK                       | Implements the API, including [sampling](#sampling), batching and export |
| Instrumentation libraries | Add telemetry to common frameworks without hand-written code  |
| OTLP                      | The wire protocol for sending telemetry                       |
| Collector                 | A standalone service that receives, filters and forwards data |
| Semantic conventions      | Standard attribute names, such as `http.request.method`       |

OTel addresses vendor lock-in. When code calls one vendor's SDK throughout, changing vendor means re-instrumenting every service. With OTel, the instrumentation stays the same and the export destination changes in configuration. Shared naming and [context propagation](#context-propagation) also let services written in different languages join the same trace.

A Collector is commonly run in production because it separates applications from backends and gives one place to redact sensitive fields and apply sampling.

Adoption has some practical hazards. Signals and language SDKs mature at different rates, so check the stability status for your language. Automatic instrumentation can emit many [spans](#span), some of them noise, and raise costs, so review what it produces. Semantic conventions have also changed over time. For example, `http.method` became `http.request.method`, so a dashboard built on the old name can go blank after an upgrade.

_Usage:_

"If we switch tracing vendors, do we have to touch every service again?"

"Not if we instrument with OpenTelemetry. We'd change the Collector's exporter and leave the application code alone."

## Section 3 — What to Measure

### Golden signals

The golden signals are four measurements that Google's SRE book recommends for [monitoring](#monitoring) a user-facing service: latency, traffic, errors and [saturation](#saturation). If a team can measure only four things about a service, these give a reasonable picture of its health.

| Signal                         | Question it answers             | Example                                |
| ------------------------------ | ------------------------------- | -------------------------------------- |
| Latency                        | How long do requests take?      | p99 checkout duration                  |
| Traffic                        | How much demand is there?       | Requests per second                    |
| Errors                         | What share of requests fail?    | Rate of 5xx responses                  |
| Saturation                     | How full is the service?        | Connection pool in use, queue depth    |

Latency should be split by outcome. A request that fails immediately is fast, so during an outage the latency graph can improve while users get nothing but errors. Track successful and failed requests separately, and read them as [percentiles](#percentile) instead of averages.

Errors include failures that do not look like errors. A 200 response with the wrong content is a failure, and so is a response slower than the service has committed to.

Traffic provides context for the others. A sudden drop often means a failure upstream, and the error rate can stay at zero throughout because no requests are arriving to fail. An error-rate graph alone will show that as a healthy service.

Saturation tends to lead the other three. It shows trouble building before users feel it, while latency and errors show what is affecting them now.

A common use is one dashboard per service with these four panels, which gives an [on-call](#on-call) engineer the same first view everywhere. [Symptom-based alerting](#symptom-based-alerting) usually pages on latency and errors. The [RED method](#red-method) is a narrower recipe for request-driven services, and the [USE method](#use-method) applies similar thinking to individual resources.

_Usage:_

"Latency just got better across the board. Did someone ship a performance fix?"

"Check the other golden signals first. Errors tripled, so we're failing fast and those requests are dragging the numbers down."

### RED method

The RED method is a [monitoring](#monitoring) recipe for request-driven services: for each service, track Rate, Errors and Duration. Tom Wilkie proposed it as a simpler form of the [golden signals](#golden-signals), leaving out [saturation](#saturation).

| Letter   | Measures                   | Typical [metric](#metric)                                 |
| -------- | -------------------------- | ------------------------------------------------------------- |
| Rate     | Requests per second        | Rate of a [counter](#counter)                             |
| Errors   | Failed requests per second | The same counter, filtered by an error label                  |
| Duration | How long requests take     | [Percentiles](#percentile) from a [histogram](#histogram) |

The benefit is consistency across services. Without a shared recipe, each team builds its own dashboards, and during an [incident](#incident) nobody can read another team's graphs quickly. With RED, every service shows the same three panels, so an engineer can judge an unfamiliar service in seconds. A shared library or a service mesh, which is a network layer handling traffic between services, can emit these metrics automatically.

The three panels are most useful read together:

| Pattern                                | Likely meaning                          |
| -------------------------------------- | --------------------------------------- |
| Rate drops, errors flat                | Upstream failure or a routing problem   |
| Errors rise, duration falls            | Requests are failing fast               |
| Duration rises, rate steady            | Slow dependency or a resource limit     |
| Rate rises, then duration, then errors | Overload                                |

Read errors against rate. Five failures per second is a serious problem at 50 requests per second and a small one at 5,000. [Alerts](#alert) generally use the ratio.

RED has limits. It says which service is unhealthy but not which resource is responsible, which is the job of the [USE method](#use-method). It also fits poorly with batch jobs and stream processors, which have no request to measure and are better judged by throughput and how far behind they are.

_Usage:_

"I've never touched the inventory service. Where do I start?"

"Open its RED dashboard. If duration is up with a steady rate, look at what it calls before you look at its code."

### USE method

The USE method is a performance-analysis checklist created by Brendan Gregg: for every resource, check Utilisation, [Saturation](#saturation) and Errors. A resource is anything with finite capacity, including CPU, memory, disks and network interfaces, as well as software limits such as connection pools.

| Letter      | Meaning                        | CPU example         | Disk example           |
| ----------- | ------------------------------ | ------------------- | ---------------------- |
| Utilisation | Share of time the resource is busy | 85% busy        | Device busy 70% of the time |
| Saturation  | Extra work waiting             | Run queue length    | I/O queue length       |
| Errors      | Count of error events          | Hardware CPU faults | Read and write errors  |

The method replaces guessing with a fixed walk through the system. When a service is slow, teams often restart it, or look only at whichever graphs already exist. Working through a list of resources covers the parts nobody thought to check.

Utilisation is easy to misread, and two cases come up repeatedly. A single-threaded process can max out one of 16 CPU cores while the host reports about 6% usage, so per-core figures matter. Averages also hide bursts: a CPU at 60% over a minute may have spent several seconds at 100% with work queued behind it.

Thread pools, file descriptors, connection pools and locks are often the actual limit. A pool of 20 connections with all 20 in use and callers waiting is saturated even when the machine looks idle.

USE complements the [RED method](#red-method). RED looks at a service from the outside and shows whether it is healthy. USE looks at the resources underneath and shows what is limiting it. A common sequence is to use RED to find the unhealthy service, USE to find the constrained resource, and then [profiling](#profiling) if the constraint is CPU.

_Usage:_

"Host CPU is only at 8%, so it can't be a compute problem."

"Run through USE per core. That worker is single-threaded, and I'd expect one core pinned at 100% with a queue behind it."

### Saturation

Saturation is how close a resource is to its limit and, past that point, how much work is waiting because the resource is busy. The [golden signals](#golden-signals) describe it as how full a service is. The [USE method](#use-method) defines it more narrowly as queued work the resource cannot yet serve.

Saturation differs from utilisation. A CPU that is 100% busy with nothing waiting is fully utilised but not saturated. The same CPU with 30 runnable threads waiting for a turn is saturated.

| Resource        | Utilisation        | Saturation signal                     |
| --------------- | ------------------ | ------------------------------------- |
| CPU             | Percent busy       | Run queue length, throttled time      |
| Memory          | Percent used       | Swapping, reclaim, out-of-memory kills |
| Connection pool | Connections in use | Callers waiting, wait time            |
| Message queue   | Not applicable     | Consumer lag, age of oldest message   |

The symptom is latency that jumps sharply while traffic has risen only a little, with no errors in the [logs](#log) and CPU looking moderate. This happens because [queueing delay](#queueing-delay) grows non-linearly near capacity. A database call that normally takes 5 ms can take 500 ms because it waited for a free connection, and the database's own slow-query record shows nothing, since the query itself was fast.

A container CPU limit is enforced over short periods, by default 100 ms on Linux, so a multi-threaded service can use its quota early in a period and then be paused for the rest. Average CPU looks comfortable while latency suffers. The throttled-time [metric](#metric) shows it.

Measure waiting as well as usage. To respond, add capacity, reduce the work per request, or bound queues so excess requests are rejected quickly instead of waiting indefinitely. [Capacity planning](#capacity-planning) aims to add resources before this point is reached.

_Usage:_

"DB CPU is at 30% and queries are fast, but the API is timing out."

"Check saturation on the connection pool. If all 50 are in use, requests are queueing before they ever reach the database."

### Tail latency

Tail latency is the response time of the slowest small fraction of requests, usually reported as p99 or p99.9. The name comes from the shape of a latency distribution: most requests form a hump near the typical value, and a long thin tail stretches to the right. A [percentile](#percentile) is the standard way to measure it.

Fan-out makes the tail matter more than its size suggests. When one user request calls many backends in parallel and waits for all of them, it is as slow as the slowest. If each backend has a 1% chance of being slower than its p99, independently:

| Backends called | Chance at least one is slower than p99 |
| --------------- | -------------------------------------- |
| 1               | 1%                                     |
| 5               | 4.9%                                   |
| 20              | 18.2%                                  |
| 100             | 63.4%                                  |

The formula is 1 - 0.99^n.

The symptom is a product that is intermittently slow while every service's median looks healthy. The problem is hard to reproduce, because any single test request will probably be fast.

Common causes include garbage collection pauses, lock contention, [queueing delay](#queueing-delay), cache misses, retries, background work such as database compaction, and other workloads competing on shared hardware. A [distributed trace](#distributed-trace) of a slow request shows where the time went, and [profiling](#profiling) can explain why.

Several techniques reduce the effect. Lower the fan-out, or return partial results when a deadline passes. Set timeouts based on observed latency. Use hedged requests, which send a second copy to another replica if the first has not replied after a short wait, and take whichever answers first. Hedging adds load, so limit how often it triggers.

Check the measurement too. Load tools affected by [coordinated omission](#coordinated-omission) can understate the tail by a wide margin.

_Usage:_

"Each service is under 50 ms at p50. Why does the page take two seconds for some users?"

"It fans out to 40 services. That's a tail latency problem, since one slow call holds up the whole response."

### Synthetic monitoring

Synthetic [monitoring](#monitoring) is running scripted checks against a service on a schedule, from outside it, to verify that it works. A check can be a single HTTP request or a browser script that signs in, adds an item to a basket and reaches checkout. Because it does not depend on user traffic, it can detect failures when nobody is using the service.

| Check type   | What it does                                    | Catches                                  |
| ------------ | ----------------------------------------------- | ---------------------------------------- |
| Uptime       | Requests a URL and checks the status            | Full outages, DNS faults, expired certificates |
| API          | Calls endpoints in sequence, validates replies  | Broken contracts and authentication      |
| Browser      | Drives a real browser through a user journey    | Broken pages and script errors           |
| Multi-region | Runs the same check from several locations      | Regional routing failures                |

The main use is covering quiet periods. A bad deployment at 3am breaks login, no customers are awake to notice, and the first report arrives at 8am. A scripted login every minute would have raised an [alert](#alert) within minutes, which directly lowers [MTTD](#mttd).

The limits come from the fact that the users are not real. Only the scripted paths are covered. A clean browser on a fast datacenter connection does not represent an old phone on a slow network, so pair it with [real user monitoring](#real-user-monitoring).

A few practices keep checks trustworthy. Require failures from more than one location before paging, so that a network problem at one probe does not wake anyone. Use dedicated test accounts and tag the traffic so it stays out of business analytics and any [SLI](#sli) based on real users. Avoid real side effects such as live payments. Maintain the scripts, because a redesigned button will fail a browser check, and a check that fails routinely contributes to [alert fatigue](#alert-fatigue).

_Usage:_

"Nobody noticed the password reset was broken until Monday."

"It gets almost no weekend traffic. Add a synthetic monitoring check for that flow so we don't rely on users reporting it."

### Real user monitoring

Real user [monitoring](#monitoring) (RUM) is collecting performance and error data from real users' browsers or apps during real sessions. A small script or SDK measures page loads, interactions and client-side errors, then sends the results to a backend.

It is usually paired with [synthetic monitoring](#synthetic-monitoring), and the two have opposite strengths:

| Aspect                | Real user monitoring                 | Synthetic monitoring              |
| --------------------- | ------------------------------------ | --------------------------------- |
| Traffic               | Real users                           | Scripted checks                   |
| Coverage              | The devices and networks users have  | Chosen paths and locations        |
| Works with no traffic | No                                   | Yes                               |
| Stable over time      | Noisy, as the user mix changes       | Stable baseline                   |
| Good at finding       | Problems affecting one segment       | Outages on known paths            |

RUM shows what server-side [metrics](#metric) cannot. A server can report an 80 ms response while the user waits six seconds, because the time went to the network, a large script bundle or page rendering. The familiar symptom is a backend dashboard that looks fine while users in one country on mobile complain. Segmenting by device, browser, country, connection type and release version usually exposes the group affected. Use [percentiles](#percentile), because client timings have a long tail.

RUM has a blind spot during severe failures. If a page fails to load at all, its script may never run, so the outage appears as a drop in sessions and not as a rise in errors. Ad blockers and privacy settings also block some reporting. Watch session volume alongside error rate.

Because RUM runs on users' devices, privacy needs attention. URLs and form fields can contain personal data, and session recording can capture more. Collect the minimum, mask sensitive fields and follow consent rules. The script itself adds page weight, so keep it small.

_Usage:_

"Our API p95 is 90 ms, so I don't buy that the app feels slow."

"Real user monitoring says p75 load on Android in Brazil is nine seconds. The API isn't where that time is going."

## Section 4 — Alerting and On-Call

### Alert

An alert is an automated notification that a monitored condition has been met. It is defined by a rule such as "error rate above 2% for 5 minutes", a severity, a destination and ideally a link to a [runbook](#runbook). The duration in the rule filters out brief blips, at the price of a slower response.

Alerts should be routed according to how quickly someone needs to act:

| Level     | Delivery                  | Expected response           | Example                    |
| --------- | ------------------------- | --------------------------- | -------------------------- |
| Page      | Phone call or push        | Act within minutes          | Checkout is failing        |
| Ticket    | Issue queue               | Act within days             | Disk fills in two weeks    |
| Record    | Dashboard, no notification | Read during investigation  | Brief spike in retries     |

The test for a page is whether a person needs to do something now. A page that wakes an engineer for a condition that clears itself before they have opened a laptop teaches them to ignore the next one, which leads to [alert fatigue](#alert-fatigue). If no action is needed, make it a ticket or remove it.

Flapping is a common defect, where a value hovers around the threshold and the alert fires and clears repeatedly. Use a longer window, or separate thresholds for firing and clearing.

A useful page says what is broken and who is affected. "HighCPU on host-17" leaves the responder to work out whether anything matters. "Checkout error rate 4%, EU users affected" with links to a dashboard and runbook gives a starting point.

Grouping related alerts stops one outage from sending 200 notifications. Silences for planned maintenance should have an expiry.

Pages are generally tied to user impact through [symptom-based alerting](#symptom-based-alerting), often as a [burn rate alert](#burn-rate-alert) on an [SLO](#slo). They go to whoever is [on-call](#on-call).

_Usage:_

"I got paged for high memory at 4am and it was gone by the time I logged in."

"Then that alert shouldn't page. Downgrade it to a ticket and page on user-facing errors instead."

### Alert fatigue

[Alert](#alert) fatigue is the loss of attention that happens when responders receive so many low-value alerts that they stop treating each one as meaningful. In practice, the phone goes off, the [on-call](#on-call) engineer acknowledges it without reading because it is usually nothing, and the one page that mattered looks the same as the forty that did not.

Each cause has a specific fix:

| Cause               | Example                             | Fix                                       |
| ------------------- | ----------------------------------- | ----------------------------------------- |
| Not actionable      | CPU high with no user impact        | Remove it or make it a ticket             |
| Threshold too tight | Fires at every daily traffic peak   | Tune from history, or use a [burn rate alert](#burn-rate-alert) |
| Flapping            | Fires and clears every few minutes  | Longer window, separate clear threshold   |
| Duplicates          | One outage sends 200 pages          | Group and deduplicate                     |
| Stale               | Nobody knows why it exists          | Review and delete                         |
| Expected            | Deployment restarts trigger a page  | Silence during known changes              |

Review alerts as a routine. At the end of each on-call shift, go through every page and ask three questions: was it real, did it need a human, and was the action clear? Track the share of pages that needed action. If a hypothetical shift had 40 pages and 6 needed action, precision was 15%, and the other 34 pages are the list to work through.

Repeated interruptions also disrupt sleep and can drive experienced engineers to leave the team.

Teams often hesitate to delete alerts in case one turns out to matter. Moving to [symptom-based alerting](#symptom-based-alerting) reduces that risk, because user impact is covered whatever the cause. If you are unsure about a specific alert, demote it to a ticket for a month and see whether anyone misses it.

_Usage:_

"I muted the channel, it's just the same five alerts all day."

"That's alert fatigue, and it's how we missed the payment page last week. Let's review every alert from this rota on Friday."

### Symptom-based alerting

Symptom-based alerting is paging on what users experience, such as failed or slow requests, instead of on internal conditions that may or may not matter. A symptom is what is wrong from the user's side. A cause is why it is happening.

| Aspect                     | Cause-based [alert](#alert) | Symptom-based alert              |
| -------------------------- | ------------------------------- | -------------------------------- |
| Example                    | CPU above 90%                   | Checkout errors threatening the [SLO](#slo) |
| Fires with no user harm    | Often                           | Rarely                           |
| Catches unforeseen failures | Only the listed causes         | Any cause with visible impact    |
| Explains why               | Sometimes                       | No, this needs investigation     |

Cause-based alerts fail in two directions. One replica dies behind a load balancer, traffic moves to the others, no user notices, and someone is still woken at 3am. On another night a bad configuration fails every payment while CPU, memory and disk stay normal, and nothing pages. Teams recognise this as being woken for harmless events and missing the serious ones.

To implement it, measure an [SLI](#sli) close to the user and alert on it, commonly with a [burn rate alert](#burn-rate-alert). Keep cause data on dashboards, such as the [golden signals](#golden-signals), where it helps with diagnosis once a page has fired.

Some causes still warrant an alert, because by the time the symptom appears it is too late to act cheaply. A disk filling up, a certificate about to expire, a quota nearly reached and a failing backup job all belong here. These alerts give lead time, and many can be tickets.

Low-traffic services need extra care, because one failure in ten requests reads as a 10% error rate. Add a minimum request count to the rule, or use [synthetic monitoring](#synthetic-monitoring) to provide steady traffic. For data pipelines, the symptom is usually stale data, so alert on freshness.

_Usage:_

"Should we page when a Kafka broker goes down?"

"Not by itself. With symptom-based alerting we page if producers start failing, and a lost broker opens a ticket for the morning."

### Burn rate alert

A burn rate [alert](#alert) fires based on how fast a service is consuming its [error budget](#error-budget). Burn rate is the observed error rate divided by the error rate the [SLO](#slo) allows. A burn rate of 1 spends the budget exactly over the SLO window, and a burn rate of 10 spends it in a tenth of the window.

For a 99.9% SLO over 30 days, the allowed error rate is 0.1%. An observed error rate of 1.44% is a burn rate of 14.4, and at that pace the 720-hour budget lasts 50 hours.

Plain thresholds handle this badly. A rule such as "errors above 0.1% for 5 minutes" pages on small blips that cost almost no budget, while a steady leak just under the threshold never fires.

The Google SRE Workbook recommends pairing a long and a short window at several burn rates:

| Severity | Long window | Short window | Burn rate | Budget spent | Error rate at 99.9% |
| -------- | ----------- | ------------ | --------- | ------------ | ------------------- |
| Page     | 1 hour      | 5 minutes    | 14.4      | 2%           | 1.44%               |
| Page     | 6 hours     | 30 minutes   | 6         | 5%           | 0.6%                |
| Ticket   | 3 days      | 6 hours      | 1         | 10%          | 0.1%                |

Budget spent is burn rate x long window / 720 hours, so the first row is 14.4 x 1 / 720 = 2%.

Both windows must exceed the threshold. The long window shows that a meaningful amount of budget has gone. The short window, one twelfth of the long one, confirms the problem is still happening, so the alert clears soon after a fix.

Low-traffic services need adjustment, since one failure in 50 requests is a 2% error rate. Add a minimum request count or synthetic traffic.

_Usage:_

"Error rate is only 0.7%. Is that worth waking anyone?"

"On a 99.9% SLO that's a burn rate of 7, so the budget is gone in about four days. If it holds, the six-hour burn rate alert will page."

### On-call

On-call is an arrangement in which an engineer is responsible, for a set period, for responding to production [alerts](#alert) within an agreed time. Teams share the duty through a rota, commonly changing weekly or daily.

A typical setup has several layers:

| Role               | Responsibility                                          |
| ------------------ | ------------------------------------------------------- |
| Primary            | Receives the page, acknowledges it and starts the response |
| Secondary          | Takes the page if the primary does not acknowledge in time |
| Escalation contact | A manager or subject expert for decisions and extra help |

Acknowledging a page means "I have this". If nobody acknowledges within a set time, often 5 to 15 minutes, the paging tool escalates to the next layer. Required response times usually depend on the [severity level](#severity-level).

Rota size determines how sustainable the duty is. With six people on a weekly rota, each engineer is on-call one week in six. Teams spread across time zones can use a follow-the-sun rota, where each region covers its own daytime so nobody is paged overnight.

An unhealthy rota is easy to recognise. People dread their week, sleep badly, and one person gets called regardless of who is on duty because only they understand a particular system. The usual causes are noisy alerts that produce [alert fatigue](#alert-fatigue), knowledge held by one person, and a missing [runbook](#runbook). Track pages per shift, night-time pages and hours spent, and treat a bad trend as a production problem.

Some practices help. Write handoff notes at the end of each shift covering open issues. Let new engineers shadow a shift before taking one alone. Compensate on-call with pay or time off, including recovery time after a bad night. Quiet shifts can go toward reducing [toil](#toil).

_Usage:_

"Priya got paged about the billing job again and she's not even on-call this week."

"Because she's the only one who knows it. Let's write the runbook and have someone else shadow her next time."

### Runbook

A runbook is a written procedure for diagnosing and handling a specific [alert](#alert) or operational task. Its reader is an [on-call](#on-call) engineer who may not know the service and may have just been woken up.

A typical runbook follows this order:

| Section         | Contents                                                     |
| --------------- | ------------------------------------------------------------ |
| What this means | The alert in plain language, and the likely user impact      |
| Check           | Dashboards, queries and commands that confirm the problem    |
| Mitigate        | Ordered, safe actions with exact commands                    |
| Verify          | What recovery looks like                                     |
| Escalate        | Who to contact and when                                      |
| Context         | Links to architecture notes and past [incidents](#incident) |

The problem a runbook solves shows up at 3am. A page says "queue depth high" with no link. The wiki page someone finds is two years old and refers to a host that no longer exists, so the responder ends up phoning a colleague who is asleep. Linking the runbook directly from the alert removes the search.

Write for someone under pressure. Give exact commands with placeholders, the expected output of each step, and warnings on anything dangerous. Put [mitigation](#mitigation) ahead of root-cause analysis, because the first goal is to restore service.

A stale runbook can do harm, since a responder will follow its instructions against a system that has since changed. Keep runbooks near the code, review them when the service changes, and update them after each use. A [game day](#game-day) is a good way to find out whether one still works.

If a runbook is a fixed sequence that needs no judgement, it is [toil](#toil) and a candidate for automation, either as a script a person triggers or as a fix that removes the page altogether.

_Usage:_

"The alert fired but I have no idea what this consumer even does."

"Follow the runbook link in the page. Step two tells you how to drain the queue safely, and it lists who to call if that fails."

## Section 5 — Incidents

### Incident

An incident is an unplanned disruption or degradation of a service that requires a coordinated response. A server crash absorbed by redundancy is a normal event. The same crash causing failed user requests is an incident. Many teams also declare incidents for security events and serious near misses.

An incident generally moves through these stages:

| Step    | Who                       | What happens                                                        |
| ------- | ------------------------- | ------------------------------------------------------------------- |
| Detect  | [Alert](#alert) or customer | The problem is noticed                                        |
| Triage  | [On-call](#on-call) engineer | Assesses impact and sets a [severity level](#severity-level) |
| Declare | On-call engineer          | Opens a channel and names an [incident commander](#incident-commander) |
| Mitigate | Responders               | Applies a [mitigation](#mitigation) to restore service          |
| Resolve | Responders                | Confirms recovery and closes the incident                           |
| Learn   | Team                      | Writes a [blameless postmortem](#blameless-postmortem)        |

The cost of having no process is easy to recognise. Five engineers debug separately in direct messages, two of them push different changes at the same moment, support has no idea what to tell customers, and nobody knows who is in charge. Declaring an incident creates one place to talk and clear roles.

Declare early. A false alarm costs a few minutes to close, while a late declaration leaves people working without coordination during the period that matters. Written criteria remove the need for judgement under stress, for example any customer-visible impact lasting over 10 minutes, any risk to data, or any problem needing more than one team.

During the response, keep a timestamped record of observations and actions, since memory of the sequence fades quickly and the review depends on it. Send stakeholders updates at a regular interval, even when the update is that nothing has changed.

Each incident spends [error budget](#error-budget), and its timings feed measures such as [MTTD](#mttd) and [MTTR](#mttr).

_Usage:_

"Should I open an incident? It might just be a blip on the payments graph."

"Open it. If it's nothing, we close it in five minutes, and if it's real we already have a channel and a commander."

### Severity level

A severity level is a ranking of an [incident](#incident)'s impact that determines how urgently people respond and who gets involved. Most organisations use labels such as SEV1 to SEV4 or P0 to P3, with lower numbers meaning more serious. Exact definitions vary between companies.

Here is an example scale:

| Level | Impact                                        | Response                                  | Example                       |
| ----- | --------------------------------------------- | ----------------------------------------- | ----------------------------- |
| SEV1  | Critical service down, or data at risk        | Page now, [incident commander](#incident-commander), executive updates | Checkout down for all users |
| SEV2  | Major degradation, or key feature down for some | Page now, incident commander            | Search failing in one region  |
| SEV3  | Minor or partial impact with a workaround     | Handle in working hours                   | Report exports running slow   |
| SEV4  | Cosmetic or no user impact                    | Ticket                                    | Broken chart on an admin page |

Define levels by user impact, and not by the technical cause or by who is complaining. Measurable criteria help: share of users affected, revenue at risk, data loss, security exposure. Without them, the familiar result is an argument in the incident channel about whether this is a SEV1 or a SEV2 while nobody works on the fix. Scales also decay in two directions. If everything is labelled SEV1, the label stops producing urgency. If teams label things low to avoid scrutiny, serious problems miss review.

When unsure, choose the higher level and downgrade later. Severity is expected to change as responders learn more, and each change should be recorded in the timeline.

Severity should drive process automatically. For example, SEV1 and SEV2 might require a [blameless postmortem](#blameless-postmortem), a status page update and customer notification.

Avoid using severity counts as a performance target for teams. Judging a team on its number of SEV1s encourages under-reporting, and hidden incidents cannot be learned from.

_Usage:_

"Is this a SEV1? Only the EU region is affected."

"EU is 40% of our customers and they can't pay. By our criteria that's a SEV1, and we can downgrade if the impact narrows."

### Incident commander

An [incident](#incident) commander (IC) is the person who coordinates the response: assigning work, tracking progress, making decisions and keeping communication flowing. The IC does not debug. The role is adapted from the Incident Command System, which US emergency services developed in the 1970s to coordinate wildfire response.

The role exists because of what happens without it. The most senior engineer starts debugging and becomes absorbed. Nobody updates support, and two people apply conflicting fixes. Someone reading a stack trace cannot also track five workstreams.

Larger incidents split the work into roles:

| Role                | Responsibility                                  |
| ------------------- | ----------------------------------------------- |
| Incident commander  | Owns the overall response and decisions         |
| Operations lead     | Directs technical investigation and changes     |
| Communications lead | Updates stakeholders and customers              |
| Scribe              | Records the timeline, findings and decisions    |

In a small incident, one person may hold several of these roles. If the IC needs to go deep on a technical problem, they should hand over command first.

The IC runs a repeating loop: what do we know, what are we trying, who owns it, and when do we check back? Each action gets a named owner and a time, as in "Sam, roll back payments and report in five minutes". The IC keeps the group focused on [mitigation](#mitigation) over root-cause analysis while users are affected.

The IC does not need to be the most senior person or the deepest expert on the system. The job calls for coordination skills, and authority during the incident comes from the role.

Handovers should be explicit and announced in the channel. In long incidents, rotate the IC every few hours to avoid exhausted decisions. Teams can build this skill by shadowing and during a [game day](#game-day).

_Usage:_

"Everyone's in the call and three people are suggesting different fixes."

"We need an incident commander. I'll take it. Dana, you own the rollback, and everyone else hold changes until she reports back."

### Mitigation

Mitigation is an action that reduces or stops user impact during an [incident](#incident), even while the underlying fault is still present. A rollback that restores service without explaining the bug is a mitigation. The permanent fix comes afterwards.

Responders usually choose from a familiar set of options:

| Mitigation           | Use when                              | Watch for                                 |
| -------------------- | ------------------------------------- | ----------------------------------------- |
| Roll back            | A recent release lines up with onset  | Schema or data changes may not reverse    |
| Disable a feature flag, a switch that turns code off without a deploy | A new feature is at fault | Other code may depend on it |
| Drain or fail over   | One host, zone or region is bad       | The rest must absorb the traffic          |
| Shed load            | Demand exceeds capacity               | Some users get errors by design           |
| Restart              | A process is stuck or leaking         | Destroys evidence, and the fault may recur |

The common failure is spending 90 minutes establishing the root cause while users stay down, when a rollback was available in the first ten. Stop the impact first and explain it later. If an incident began right after a change, reversing that change is usually the first thing to try.

Capture evidence where it is cheap. A heap dump, thread dump or saved [log](#log) excerpt taken before a restart may be the only record of the faulty state. Do not hold up recovery to collect it.

Apply one change at a time where possible, so you can tell what worked, and confirm the effect using a user-level [SLI](#sli) instead of assuming.

A mitigation also needs a follow-up. Temporary measures such as extra replicas or a disabled job tend to become permanent unless someone tracks them. The time taken to reach this point is what the "restore" version of [MTTR](#mttr) measures.

_Usage:_

"I think I've nearly found the bug, give me another twenty minutes."

"Users are still down. Roll back as the mitigation now, then keep digging with the pressure off."

### Blameless postmortem

A blameless postmortem is a written review of an [incident](#incident) that examines how the system and organisation allowed it to happen, without assigning fault to individuals. It starts from the assumption that people acted reasonably given what they knew at the time.

The reason is practical. If admitting a mistake leads to punishment, people stop reporting mistakes and leave details out, and the organisation loses the information it needs. The recognisable failure is a report concluding "engineer ran the wrong command, reminded to be more careful". Nothing changes, and the same failure returns. Better questions are why one command could do that much damage, why nothing asked for confirmation, and why staging behaved differently.

A typical document contains:

| Section                                             | Contents                                         |
| --------------------------------------------------- | ------------------------------------------------ |
| Summary                                             | What happened, in a few sentences                |
| Impact                                              | Users affected, duration, [error budget](#error-budget) spent |
| Timeline                                            | Timestamped events and decisions                 |
| [Contributing factors](#contributing-factor)  | Conditions that allowed or worsened the failure  |
| What went well                                      | Practices worth keeping                          |
| Action items                                        | Specific changes, each with an owner and due date |

Blameless does not mean unaccountable. People are still expected to take part honestly and to complete their actions. Many teams refer to roles in the document, such as "the [on-call](#on-call) engineer", instead of names.

Action items are where postmortems most often fail. Vague, unowned items such as "improve [monitoring](#monitoring)" are forgotten once the document is filed. Write specific, checkable items, such as "page when payment errors burn budget at 14.4 times the allowed rate". Prefer changes to systems over training or reminders, because systems do not depend on someone remembering. Track items to completion with the same priority as feature work.

Hold the review within a few days and share the result widely.

_Usage:_

"The postmortem draft says Tom caused the outage by deploying on Friday."

"Rewrite that. Ask why our pipeline let a broken config reach every region, because that's still true whoever deploys next."

### Contributing factor

A contributing factor is a condition that helped an [incident](#incident) happen or made it worse. Incident reviews use the term because failures in complex systems rarely have one cause. Several ordinary conditions, each harmless alone, combine.

Consider a hypothetical incident in which a configuration change sets a timeout to zero and takes an API down for 50 minutes:

| Contributing factor                       | Effect                            | Action                                              |
| ----------------------------------------- | --------------------------------- | --------------------------------------------------- |
| Config values were not validated          | The bad value was accepted        | Add schema validation                               |
| Change applied to all regions at once     | Local mistake became global       | Roll out region by region                           |
| [Alerts](#alert) watched only CPU     | Detected after 20 minutes, by support | Add [symptom-based alerting](#symptom-based-alerting) |
| [Runbook](#runbook) had an old command | Recovery took 15 minutes longer  | Update and test it                                  |

A trigger is different from a factor. The trigger is the event that set off the failure, here the config change. The factors explain why that event did so much damage, and most of them existed long before the day of the incident.

Asking for a single root cause tends to stop the investigation at the first plausible answer, which is often the last human action: "an engineer pushed bad config". Fixing only that leaves every other condition in place, ready to combine with the next mistake. The symptom is a team that closes each incident with a fix and still sees the same kind of failure return with a different trigger.

"Human error" is not a useful factor on its own. Treat it as the start of the question: what made this action easy to get wrong and hard to notice?

Not every factor deserves work. In a [blameless postmortem](#blameless-postmortem), record a decision for each one: fix it, monitor it or accept the risk explicitly.

_Usage:_

"Can we just put 'bad deploy' as the root cause and close this?"

"The deploy was the trigger. No canary and a silent alert were contributing factors, and those will bite us again."

## Section 6 — Capacity and Performance

### Capacity planning

Capacity planning is forecasting future demand and arranging enough resources to serve it while still meeting [SLOs](#slo). It covers compute, storage, network, database connections and third-party quotas.

The process follows a repeating sequence, shown here with hypothetical numbers:

| Step | Action                    | Example                                      |
| ---- | ------------------------- | -------------------------------------------- |
| 1    | Measure current peak      | 6,000 requests per second                    |
| 2    | Forecast growth           | 10% a month for 6 months gives about 10,630  |
| 3    | Measure unit capacity     | [Load testing](#load-testing) shows 500 per instance within SLO |
| 4    | Calculate need            | 10,630 / 500 = 21.26, so 22 instances        |
| 5    | Add redundancy            | To survive losing one of 3 zones, 11 per zone, 33 total |
| 6    | Review                    | Compare forecast with actual each quarter    |

Step 5 is often skipped. With 22 instances spread over three zones, losing a zone leaves 14 or 15, which cannot carry the forecast peak. With 33, the remaining two zones hold 22.

Forecasts need two inputs. Organic growth is the steady trend visible in historical data. Inorganic growth comes from launches, marketing campaigns and large new customers, and it appears in no graph until it happens. The familiar failure is a launch that takes the site down because nobody told the infrastructure team about it.

Lead times shape the plan. Quota increases, hardware orders and splitting a database can take weeks. Autoscaling, which adds instances automatically as load rises, does not remove the need to plan. It reacts with a delay, stops at quota limits and does not help components that cannot scale out, such as a single primary database.

Plan around [saturation](#saturation) instead of CPU alone, and keep headroom, because [queueing delay](#queueing-delay) rises sharply near capacity. Re-measure unit capacity after major releases, since a performance regression changes every number above.

_Usage:_

"Marketing is running a TV ad in March. Do we need to do anything?"

"Yes, get the expected traffic from them now. Capacity planning needs a number, and our database quota takes weeks to raise."

### Load testing

Load testing is sending realistic traffic to a system at expected volumes to check whether it meets its performance targets. A typical question is whether the service can handle next month's peak with p99 latency under 300 ms.

It belongs to a family of tests that ask different questions:

| Test                                | Question                          | Load shape                 | Typical duration   |
| ----------------------------------- | --------------------------------- | -------------------------- | ------------------ |
| Load test                           | Does it meet targets at expected load? | Ramp to peak and hold | Minutes to an hour |
| [Stress test](#stress-testing) | Where and how does it break?     | Increase past capacity     | Until failure      |
| [Soak test](#soak-test)       | Does it degrade over time?        | Steady and moderate        | Hours to days      |

The result is only as good as the realism of the test. The common symptom is a system that passes at 5,000 requests per second and falls over in production at 2,000. Usual reasons are simulated users all requesting the same cached URL, a test database far smaller than production, missing background jobs, and an environment that differs from production.

Derive the traffic mix from production [logs](#log). Use data volumes close to production. Ramp up gradually and hold long enough to get past cold caches. Report [percentiles](#percentile) and error rates, not averages.

Watch the load generator too. If it runs out of CPU, the latencies it reports are wrong. Prefer an open workload, which sends requests at a fixed arrival rate whatever the responses do. Tools that wait for each reply slow down along with the server and hide the delay, an error called [coordinated omission](#coordinated-omission).

[Little's Law](#littles-law) gives a quick sanity check: 50 clients that each wait for a reply at 200 ms average latency cannot exceed 250 requests per second.

Results feed directly into [capacity planning](#capacity-planning).

_Usage:_

"The load test passed at twice our peak, so Black Friday is safe."

"It also hit one product page with an empty cart table. Replay real traffic against a production-sized database before we trust that."

### Stress testing

Stress testing is pushing a system beyond its expected capacity to find where it breaks, how it fails and whether it recovers. [Load testing](#load-testing) asks whether a system meets its targets at expected demand.

A stress test usually runs in phases:

| Phase    | Action                               | Record                                   |
| -------- | ------------------------------------ | ---------------------------------------- |
| Baseline | Run at normal load                   | Latency and error rate                   |
| Ramp     | Raise load in steps, holding each    | The point where the [SLO](#slo) fails |
| Break    | Continue until errors dominate       | First resource to reach [saturation](#saturation) |
| Recover  | Drop back to normal load             | Time to recover, or failure to recover   |

The first useful output is the knee: the load at which latency starts rising sharply because [queueing delay](#queueing-delay) takes over. Plan capacity below the knee, not at the point of collapse.

How the system fails matters as much as where. In a graceful failure, it rejects excess requests quickly and keeps serving what it can. In a collapse, it accepts everything, queues grow, timeouts trigger retries, and it spends its time on requests whose clients have already given up. Throughput then falls as load rises.

The recovery phase catches a difficult case known as a metastable failure, where the system stays broken after the original trigger has gone. Retries or cold caches keep load above what it can handle. In production this appears as a traffic spike that has passed while the service stays down until someone restarts it or blocks traffic. A [thundering herd](#thundering-herd) is a frequent cause.

Findings usually lead to load shedding (rejecting excess work on purpose), bounded queues, timeouts, and retry budgets that cap retries as a share of requests.

Run stress tests in an isolated environment, and confirm that each [alert](#alert) fires at the expected point.

_Usage:_

"We know it handles 8,000 requests a second. What happens at 12,000?"

"Nobody knows. Let's stress test it in staging, then drop the load and see if it recovers without a restart."

### Soak test

A soak test runs a system at steady, realistic load for an extended period, typically hours or days, to expose problems that only appear over time. It is also called an endurance test. A short [load test](#load-testing) can pass while the system is slowly heading toward failure.

Soak tests find a specific class of defect:

| Problem                         | What you see over time                         |
| ------------------------------- | ---------------------------------------------- |
| [Memory leak](#memory-leak) | Memory floor climbs until the process is killed |
| Connection leak                 | Open connections rise until the pool runs out  |
| Disk growth                     | [Logs](#log) or temp files fill the disk   |
| Data growth                     | Queries slow down as tables grow               |
| Time-based events               | Token expiry, scheduled jobs, log rotation     |

The recognisable symptom is a service that runs well after each deployment and dies after a few days. Teams that deploy often may never see the problem, because each release restarts the process and resets the clock. It then surfaces during a holiday change freeze, when the service runs uninterrupted for longer than it ever has.

Run the test at roughly typical production load. Make it longer than the longest cycle you care about. If a batch job runs daily or an access token lasts 24 hours, a soak of under a day will not exercise them.

Watch the trend instead of the level. Graph each [gauge](#gauge) for memory, connections, threads and disk, and compare the first hour with the last. A flat line is healthy, and a slope gives a time to failure. In a hypothetical test where memory grows 20 MB per hour with 1.6 GB free, the process runs out after 80 hours. An 8-hour soak would not crash, but its slope reveals the problem.

_Usage:_

"It passed the load test, but in production it falls over every four days."

"An hour-long test can't catch that. Run a soak test over the weekend and watch the memory and connection graphs."

### Little's Law

Little's Law states that the average number of items in a stable system equals the average arrival rate multiplied by the average time each item spends in the system. John Little published a proof in 1961.

```text
L = λ × W
```

L is the average number in the system, λ (lambda) is the arrival rate, and W is the average time in the system. The units must match, so convert milliseconds to seconds when the rate is per second. The law holds whatever the arrival pattern or distribution of service times, as long as the system is stable over the period measured.

It answers common sizing questions, shown here with hypothetical numbers:

| Question                                         | Calculation | Answer             |
| ------------------------------------------------ | ----------- | ------------------ |
| Requests in flight at 200 per second and 250 ms? | 200 x 0.25  | 50 concurrent      |
| Maximum rate for 100 connections held 50 ms each? | 100 / 0.05 | 2,000 per second   |

The law explains a confusing kind of outage, where a slow dependency exhausts threads in a service that has no bug. At 200 requests per second and 250 ms, a service holds 50 requests in flight. If a dependency slows and requests now take one second, the same traffic needs 200 in flight. With a pool of 100 threads, requests begin to queue or fail although traffic never changed.

The law describes averages, so it says nothing about [percentiles](#percentile) or short bursts, and sizing needs headroom on top. It does not apply while arrivals exceed completions, because the queue keeps growing and no steady average exists. Related limits appear as [saturation](#saturation) and [queueing delay](#queueing-delay).

_Usage:_

"How big should the connection pool be for the new service?"

"Little's Law says 500 queries a second at 20 ms is 10 in flight on average. Start around 20 for bursts and watch wait time."

### Queueing delay

Queueing delay is the time a request spends waiting for a busy resource before any work on it begins. Total latency is queueing delay plus service time, the time spent doing the work.

A simple queueing model, known as M/M/1, assumes one server, random arrivals and exponentially distributed service times. In this model the average total time is S / (1 - U), where S is service time and U is utilisation. With a 10 ms service time:

| Utilisation | Average wait | Average total |
| ----------- | ------------ | ------------- |
| 50%         | 10 ms        | 20 ms         |
| 80%         | 40 ms        | 50 ms         |
| 90%         | 90 ms        | 100 ms        |
| 95%         | 190 ms       | 200 ms        |
| 99%         | 990 ms       | 1,000 ms      |

Moving from 90% to 99% utilisation is 10% more traffic and ten times the latency. Real systems have different numbers, since multiple workers soften the curve and bursty traffic worsens it, but the shape holds.

The symptom is a service that is fine all day until a small rise at peak makes latency jump. Engineers search for a bad query or a recent deployment that does not exist. Each request takes no longer to process once it starts, so [profiling](#profiling) shows nothing new. In a [distributed trace](#distributed-trace), the time appears as a gap before a [span](#span) begins or between client and server spans.

To respond, keep utilisation below the steep part of the curve through [capacity planning](#capacity-planning), reduce service time or add workers. Bound queues and reject work when they fill, because a request served after its client has given up is wasted. Track queue depth or wait time as a [saturation](#saturation) signal. [Little's Law](#littles-law) connects the two: 30 requests waiting at 200 per second means a 150 ms average wait.

_Usage:_

"Traffic is up 8% and p99 tripled. Did someone ship something?"

"Nothing shipped. We're at 92% utilisation, so that's queueing delay. Add two instances and it should drop."

### Coordinated omission

Coordinated omission is a measurement error in which a load generator slows down when the system under test slows down, and so never records the latency of requests it should have sent. Gil Tene gave the problem its name. It makes [tail latency](#tail-latency) look far better than it is.

The mechanism is a closed loop. Many tools wait for each response before sending the next request. During a stall, the tool sends nothing, so the tool and the server in effect cooperate to leave out samples at the moment the measurements would be worst.

Take a hypothetical test intended to send 100 requests per second for 60 seconds over one connection. Responses normally take 5 ms, and the server stalls once for 10 seconds.

| Measure                   | Recorded by closed-loop tool | Measured from intended send times |
| ------------------------- | ---------------------------- | --------------------------------- |
| Requests counted          | 5,001                        | 6,000                             |
| Requests delayed by stall | 1                            | 1,000                             |
| [p99](#percentile)    | 5 ms                         | About 9.4 s                       |
| Maximum                   | 10 s                         | 10 s                              |

The 1,000 requests scheduled during the stall would have waited between nearly 0 and 10 seconds. The tool saw only one of them.

The symptom is a [load test](#load-testing) reporting a p99 of a few milliseconds while users see multi-second pauses in production. A high maximum next to a low p99 is a warning sign.

To avoid it, use an open-loop generator that launches requests on a fixed schedule regardless of outstanding responses, and measure latency from the intended start time.

Production measurement can have the same blind spot. A service that starts its timer when a handler begins does not count [queueing delay](#queueing-delay) before that point. Measure at the client or load balancer as well.

_Usage:_

"The benchmark says p99 is 4 ms, but we saw a ten-second GC pause during the run."

"Then it's coordinated omission. The tool stopped sending during the pause, so rerun it with a constant arrival rate."

### Thundering herd

A thundering herd is many clients or processes acting at the same moment and overwhelming a shared resource. Something usually synchronises them: a cache expiry, a service restart, a scheduled job or a reconnect after a failure.

| Trigger                      | What happens                            | Countermeasure                                     |
| ---------------------------- | --------------------------------------- | -------------------------------------------------- |
| Popular cache key expires    | Every request hits the database at once | Request coalescing, where one fetches and others wait |
| Many keys share one lifetime | They all expire together                | Add a random offset to each expiry                 |
| Service recovers from outage | All clients reconnect and retry         | Exponential backoff with jitter                    |
| Midnight scheduled job       | The whole fleet runs it simultaneously  | Randomise start times                              |
| Cold restart                 | Empty caches send all reads downstream  | Warm caches and ramp traffic gradually             |

The recognisable symptom is a service that comes back after an outage and immediately falls over again. Engineers restart it, it survives for seconds, and the cycle repeats. Load at recovery can exceed the normal peak, because clients have accumulated retries and every one of them tries at once.

Randomness breaks the synchronisation. If 1,000 clients each retry after exactly 5 seconds, the server receives 1,000 requests in the same instant, round after round. Spreading those retries randomly over 30 seconds gives about 33 per second. This random spread is called jitter. Exponential backoff, which lengthens the wait after each failure, is not enough alone, because clients that failed together still retry together at 1, 2 and 4 seconds.

During recovery, the usual [mitigation](#mitigation) is to admit traffic in stages at the load balancer and reject the excess until caches are warm. The recovery phase of [stress testing](#stress-testing) can expose this behaviour in advance.

_Usage:_

"We brought the API back and it died again in thirty seconds."

"That's a thundering herd, every mobile client is retrying at once. Let traffic in 10% at a time and add jitter to the client backoff."

### Memory leak

A memory leak is memory that a program keeps allocated after it no longer needs it, so usage climbs until the process slows down or is killed. In manually managed languages such as C, it is memory that was never freed. In garbage-collected languages, the collector cannot release anything still referenced, so a leak is usually an object kept reachable by accident. Common causes are unbounded caches, global maps keyed by request or user, event listeners never removed, and threads blocked forever while holding references.

The usual symptom is that a restart fixes everything for a while. The memory [gauge](#gauge) climbs steadily, the process is killed for exceeding its limit, it restarts, and the cycle repeats at a regular interval.

The shape of the memory graph helps separate a leak from normal behaviour:

| Pattern                         | Likely meaning                                |
| ------------------------------- | --------------------------------------------- |
| Sawtooth with a flat floor      | Normal garbage collection                     |
| Sawtooth with a rising floor    | Leak                                          |
| Rises, then plateaus            | A cache filling to its limit                  |
| Sudden step with a traffic burst | Large allocation, not necessarily a leak     |
| Process memory grows, heap flat | Native memory use or fragmentation            |

Small leaks add up under traffic. If each request retains 2 KB at a hypothetical 100 requests per second, the process leaks 200 KB per second, or 720 MB per hour. A container with a 2 GB limit and a 500 MB baseline has 1.5 GB of headroom and would be killed after about two hours.

To diagnose a leak, take two heap snapshots some time apart under load, compare which object types grew, and find what is holding references to them. Heap [profiling](#profiling) supports this directly. A [soak test](#soak-test) can reproduce the problem before release, whereas frequent deployments can hide it by restarting the process.

_Usage:_

"The worker gets OOM-killed every six hours, like clockwork."

"Sounds like a memory leak. Grab two heap dumps an hour apart and diff them to see what's growing."

### Profiling

Profiling is measuring where a running program spends resources, such as CPU time, memory or waiting time, down to the level of individual functions. It replaces guesses about performance with data. The familiar failure it prevents is spending two days optimising a function that turns out to account for 1% of runtime, where even a perfect fix could save at most 1%.

Different profile types answer different questions:

| Type       | Question                      | Useful when                           |
| ---------- | ----------------------------- | ------------------------------------- |
| CPU        | Which code is on the CPU?     | High CPU or slow computation          |
| Heap       | What is holding memory now?   | Investigating a [memory leak](#memory-leak) |
| Allocation | What allocates the most?      | Heavy garbage collection              |
| Off-CPU    | Where are threads blocked?    | Slow service with low CPU             |

Profilers work in two main ways. A [sampling](#sampling) profiler interrupts the program at a fixed frequency, such as 99 times per second, and records the call stack. Its overhead is low, and functions that use a lot of time appear in many samples. An instrumenting profiler records every function entry and exit, which is exact but slower and can distort the behaviour being measured. Sampling is the usual choice in production.

A CPU profile does not show waiting. A request that uses 10 ms of CPU and blocks for 900 ms on a database barely appears in it, which is why a slow service can produce a CPU profile with nothing interesting in it. Use an off-CPU profile or a [distributed trace](#distributed-trace) for that case.

Continuous profiling runs low-overhead sampling across production all the time. It lets you compare before and after a deployment, or inspect the moment of an [incident](#incident) without reproducing it. Results are commonly displayed as a [flame graph](#flame-graph).

_Usage:_

"I'm going to rewrite the JSON parser, it has to be what's slow."

"Profile it first. Last time we guessed, the parser was 2% of CPU and the real cost was in logging."

### Flame graph

A flame graph is a visualisation of [profiling](#profiling) data that shows sampled call stacks, with each box's width proportional to the share of samples in which that function appeared. Brendan Gregg created the format.

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

## Section 7 — Learning from Change

### Chaos engineering

Chaos engineering is the practice of deliberately injecting failures into a system, as a controlled experiment, to test whether it behaves as expected. It grew out of work at Netflix, whose Chaos Monkey tool randomly terminated production instances.

The practice is an experiment with a defined method:

| Step | Action                 | Example                                       |
| ---- | ---------------------- | --------------------------------------------- |
| 1    | Define steady state    | Checkout success stays at or above 99.9%      |
| 2    | Form a hypothesis      | Losing one cache node will not affect checkout |
| 3    | Limit the blast radius | 1% of traffic in one zone                     |
| 4    | Set abort conditions   | Stop if the [SLI](#sli) drops below target |
| 5    | Inject the fault       | Terminate a cache node                        |
| 6    | Observe and compare    | Did steady state hold?                        |
| 7    | Fix and repeat         | Address what was learned                      |

The problem it addresses is redundancy that has never been exercised. A team has a standby database and a failover procedure that has never run. On the day the primary fails, the standby turns out to have stale configuration or too little capacity.

Typical injected faults include instance termination, added network latency, packet loss, dependency errors, DNS failures, full disks and clock skew. Added latency is often more revealing than a crash, because a slow dependency ties up threads and connections while a dead one fails quickly.

Some preconditions apply. You need enough [observability](#observability) to see the effect, an [SLO](#slo) to define normal, and a fast way to stop. If you already expect the system to fail the experiment, fix it first. Start in staging and move to production as confidence grows, since only production has real traffic and configuration.

Experiments that affect users spend [error budget](#error-budget), so run them when budget is available. A [game day](#game-day) complements this work by exercising the people as well as the software.

_Usage:_

"We've got three replicas, so losing one is fine."

"Have we ever tried it? Let's run a chaos engineering experiment on 1% of traffic and see whether failover works."

### Game day

A game day is a scheduled exercise in which a team simulates a failure and practises detecting it, responding to it and recovering from it. [Chaos engineering](#chaos-engineering) tests how software behaves. A game day also tests people, tools and procedures.

An exercise usually runs through these phases:

| Phase   | What happens                                                 |
| ------- | ------------------------------------------------------------ |
| Plan    | Choose the scenario, scope, participants and abort criteria  |
| Brief   | Tell stakeholders the time and the possible impact           |
| Inject  | A facilitator triggers or simulates the failure              |
| Respond | [On-call](#on-call) engineers work it as a real [incident](#incident) |
| Recover | Restore normal operation and confirm the system is healthy   |
| Debrief | Record the gaps and assign actions                           |

The problem it addresses is discovering a broken procedure during a real outage. The first attempt to restore a database from backup should not be the one that reveals the backup is corrupt or that the restore takes nine hours. Game days also expose missing credentials, an out-of-date [runbook](#runbook), an [alert](#alert) that never fires, dashboards some responders cannot open, and uncertainty over who may make a decision.

The format can match the team's maturity. A tabletop exercise talks through the scenario without touching any system and costs little. A staging exercise runs real actions safely. A production exercise is the closest to reality and needs strong safeguards.

Keep the exercise realistic. Use the normal tools, practise the [incident commander](#incident-commander) role, and have an observer note where people got stuck. Label all communication as an exercise so nobody mistakes it for a real outage, and stop at once if a real incident begins.

Track the resulting actions as you would those from a [blameless postmortem](#blameless-postmortem). Repeat the exercise periodically, because systems and teams change.

_Usage:_

"Our DR plan says we can restore in an hour. Has anyone timed it?"

"No. Let's schedule a game day next month, restore last night's backup and see how long it takes."

### DORA metrics

DORA metrics are measures of software delivery performance from the DevOps Research and Assessment (DORA) programme. The research originally identified four key metrics. DORA renamed and narrowed one of them in 2023 and added a fifth in 2024.

| Metric                          | Group       | What it measures                                                  |
| ------------------------------- | ----------- | ----------------------------------------------------------------- |
| Change lead time                | Throughput  | Time from commit to running in production                         |
| Deployment frequency            | Throughput  | How often deployments happen                                      |
| Failed deployment recovery time | Throughput  | Time to recover from a deployment needing immediate intervention  |
| [Change fail rate](#change-failure-rate) | Instability | Share of deployments needing immediate intervention |
| Deployment rework rate          | Instability | Share of deployments that are unplanned and caused by a production [incident](#incident) |

Failed deployment recovery time replaced "time to restore service". It counts only failures caused by a change, so it excludes events such as a data centre outage, and it is narrower than a general [MTTR](#mttr).

DORA's research has found that throughput and stability tend to go together: teams that deploy frequently in small batches generally also have fewer and shorter failures.

The usual misuse is turning these measures into targets or league tables. Once deployment frequency is a goal, teams split releases into trivial deployments. Once failure rate is a goal, failures get reclassified as something else. Comparisons between teams also mislead, since a mobile app that waits on store review and a web service have different constraints. Use the metrics to follow one team's trend and to find its bottleneck.

Agree what counts as a deployment and a failure, and pull the data from the deployment pipeline and incident tracker.

These metrics describe delivery and say nothing directly about user experience. Pair them with an [SLO](#slo), since a team can score well on DORA while its service misses its reliability target.

_Usage:_

"The VP wants every team ranked on DORA metrics this quarter."

"Then we'll get lots of one-line deploys. Let's show each team's own trend and use it to find where changes get stuck."

### Change failure rate

Change failure rate is the share of deployments to production that need immediate intervention afterwards, such as a rollback or hotfix. It is one of the [DORA metrics](#dora-metrics), where it is named "change fail rate". The formula is failed deployments divided by total deployments. If a team ships 120 deployments in a quarter and 6 need a rollback or hotfix, the rate is 5%.

The number depends on what the team counts as a failure:

| Event                                                  | Usually counts?                |
| ------------------------------------------------------ | ------------------------------ |
| Deployment rolled back because of errors               | Yes                            |
| Hotfix shipped to correct a release                    | Yes                            |
| Caught by a canary, a release sent to a small share of traffic first | Depends, so record the policy |
| Bug found weeks later and fixed in normal work         | No                             |
| Outage caused by a cloud provider                      | No                             |
| Planned rollback for business reasons                  | No                             |

Without agreed definitions, the figure can drift away from reality. The telltale sign is a rate of 1% on a team whose engineers remember constant firefighting. Often failures are being fixed forward in later releases and never labelled as failures. Linking deployments to [incidents](#incident) and to hotfix commits gives a more honest count.

The rate says nothing about severity. A change that takes down checkout counts the same as one with a minor effect, so read it alongside [error budget](#error-budget) spend and recovery time.

Deploying less often to reduce failures tends to backfire. Larger batches carry more change per release and are harder to diagnose when they go wrong. To lower the rate, make changes smaller, strengthen automated tests, validate configuration, release to a growing share of traffic, and act on the [contributing factors](#contributing-factor) found in past failures.

_Usage:_

"Our change failure rate is 2%, so why does [on-call](#on-call) feel so bad?"

"We only count rollbacks. Half our bad releases get patched forward, so let's count those hotfixes too."

### Production readiness review

A production readiness review (PRR) is a structured check that a service can be run safely in production, done before it launches or before another team takes responsibility for it. Google's [site reliability engineering](#site-reliability-engineering) teams helped popularise the practice.

A review typically covers these areas:

| Area             | Questions                                                                  |
| ---------------- | -------------------------------------------------------------------------- |
| Targets          | Are [SLIs](#sli) and [SLOs](#slo) defined?                         |
| Visibility       | Is there enough [observability](#observability) to debug it?           |
| Alerting         | Does each [alert](#alert) reflect user impact and link a [runbook](#runbook)? |
| Capacity         | Has [load testing](#load-testing) established its limits?            |
| Failure handling | What happens when each dependency fails?                                   |
| Change safety    | Can a release be rolled back quickly?                                      |
| Data             | Are backups taken, and has a restore been tested?                          |
| Security         | How are secrets and access managed?                                        |
| Ownership        | Who is [on-call](#on-call), and do they know the service?              |

The problem it prevents is a service that ships on a Friday and pages someone who has never seen it. There are no dashboards, no runbook and no list of dependencies, so the team discovers each gap during its first [incident](#incident). A review moves that discovery to a point where fixes are cheaper.

Timing affects the outcome. A review held a week before launch produces a list nobody has time to address, so start the conversation at the design stage. The output should separate blocking items, which must be fixed before launch, from non-blocking items tracked with owners and dates. When a team launches with a known gap, a named person should accept that risk explicitly.

A review can also decay into bureaucracy, where a long generic checklist becomes box-ticking and teams route around it. Scale the rigour to how critical the service is, so an internal tool gets a lighter review than a payment path.

_Usage:_

"We're launching the new billing service next Tuesday. Can SRE take the pager?"

"Not before a production readiness review. Right now it has no runbook, no SLO and an untested restore."
