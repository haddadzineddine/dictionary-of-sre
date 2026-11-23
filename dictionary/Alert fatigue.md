---
description: When frequent or low-value alerts train responders to ignore them, so real problems get missed or handled late.
---

[Alert](./Alert.md) fatigue is the loss of attention that happens when responders receive so many low-value alerts that they stop treating each one as meaningful. In practice, the phone goes off, the [on-call](./On-call.md) engineer acknowledges it without reading because it is usually nothing, and the one page that mattered looks the same as the forty that did not.

Each cause has a specific fix:

| Cause               | Example                             | Fix                                       |
| ------------------- | ----------------------------------- | ----------------------------------------- |
| Not actionable      | CPU high with no user impact        | Remove it or make it a ticket             |
| Threshold too tight | Fires at every daily traffic peak   | Tune from history, or use a [burn rate alert](./Burn%20rate%20alert.md) |
| Flapping            | Fires and clears every few minutes  | Longer window, separate clear threshold   |
| Duplicates          | One outage sends 200 pages          | Group and deduplicate                     |
| Stale               | Nobody knows why it exists          | Review and delete                         |
| Expected            | Deployment restarts trigger a page  | Silence during known changes              |

Review alerts as a routine. At the end of each on-call shift, go through every page and ask three questions: was it real, did it need a human, and was the action clear? Track the share of pages that needed action. If a hypothetical shift had 40 pages and 6 needed action, precision was 15%, and the other 34 pages are the list to work through.

Repeated interruptions also disrupt sleep and can drive experienced engineers to leave the team.

Teams often hesitate to delete alerts in case one turns out to matter. Moving to [symptom-based alerting](./Symptom-based%20alerting.md) reduces that risk, because user impact is covered whatever the cause. If you are unsure about a specific alert, demote it to a ticket for a month and see whether anyone misses it.

_Usage:_

"I muted the channel, it's just the same five alerts all day."

"That's alert fatigue, and it's how we missed the payment page last week. Let's review every alert from this rota on Friday."
