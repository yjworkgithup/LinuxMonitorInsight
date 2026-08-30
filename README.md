# LinuxMonitorInsight
  SSH agentless


# LinuxMonitorInsight

**Agentless, SSH-based monitoring for HPC & AI clusters — the monitor that tells the truth when everything is on fire.**

> **Hero (3-sentence version):** Most monitors answer a network outage by lying (5,000 false "dead" nodes) or going silent (nodes vanish, no explanation). LinuxMonitorInsight gives a third answer: *"My channel is sick — your nodes are not dead. Go check the switch."* It is a verifiable epistemic state machine, battle-tested on a 5,000-node cluster through a 1-hour+ outage with zero false OFFLINE verdicts.

---

## When the network dies for 30 minutes, what does your monitor say?

Most monitoring systems have only two responses to a disaster: **lie, or go silent.**

- **Lie** — mark 5,000 healthy machines as "dead," fire 5,000 alerts, and let the on-call engineer drown in the storm.
- **Go silent** — nodes quietly vanish from the dashboard; 5,000 becomes 4,987 with no explanation. You can't tell whether the machines died or the monitor went blind.

**LinuxMonitorInsight gives a third answer**: *"My channel is sick — your nodes are not dead. Go check the switch."*

This is not rhetoric. It is a verifiable epistemic state machine, validated in production on a 5,000-node cluster through a 1-hour+ network outage.

## Head-to-head

| Dimension | Prometheus | Nightingale (n9e) | Ganglia | **LinuxMonitorInsight** |
|---|---|---|---|---|
| **Deployment** | `node_exporter` on every host | `categraf`/agent on every host | `gmond` agent on every host | **Pure SSH. Zero agents. Zero footprint.** |
| **When a node goes quiet** | Series silently vanishes (`absent_over_time` must be hand-written) | Depends on manual NoData config | Greyed out, binary up/down | **Conservation law: nodes never vanish; UNKNOWN is a first-class state** |
| **Behavior during an outage** | Mass false "down" — or total silence | Mass false alerts | Mass grey-out | **Separates "channel sick" from "node dead." No guilt by association.** |
| **Where thresholds come from** | Magic numbers | Magic numbers | Magic numbers coupled to poll interval | **Derived from measurement + audit flywheel; every constant has a birth certificate** |
| **Knows its own error rate?** | No | No | No | **Yes — precision@horizon self-audit; false-positive rate goes on the books** |
| **HPC/AI-native features** | None | None | Basic load only | **IO-storm (NFS D-state) detection, GPU job accounting, straggler detection, digital twin** |
| **Validated scale** | Generic large clusters | Generic large clusters | Small–medium | **5,000 nodes + 1-hour outage, real-world validated** |

## The honest details

**vs Prometheus** — Prometheus is a superb time-series database, but it has no concept of an *entity*. A lost node = a vanished series; you must rebuild conservation yourself with `absent_over_time` plus an external inventory. Its alerts **auto-resolve when a target disappears** — the ecosystem's most famous counter-intuitive trap. We built conservation into the kernel.

**vs Nightingale (n9e)** — Nightingale layers enterprise alerting on the Prometheus ecosystem, but still requires an agent on every host and still has no "I don't know" state. In an outage it mass-false-alarms; we mass-say *"it's the channel, not the nodes."*

**vs Ganglia** — The HPC old-timer. Its host table persists (it got conservation right), but it is binary up/down only: `down` is a static terminal with no aging, no escalation, no attribution — and its thresholds are coupled to the polling interval. We inherited its conservation and added the entire epistemic ladder it lacks.

## The state ladder: `fresh → stale → timeout → unknown → offline`

Each transition is a verdict with a specific burden of evidence:

- **TIMEOUT** answers *"is the node silent beyond noise?"* (wall clock)
- **UNKNOWN** answers *"is it the node, or is it my channel?"* (attribution)
- **OFFLINE** answers *"is the node truly dead?"* — and requires the previous questions to be answered first

No rung may be skipped. `ONLINE → OFFLINE` directly is a **forbidden edge, enforced in code, not in documentation.**

## Our three bottom lines

1. **Conservation law** — `healthy + degraded + timeout + unknown + offline ≡ inventory total`. The denominator comes from an inventory source independent of the collection pipeline, so conservation holds even when the collector itself crashes.
2. **No guilt by association** — during channel-wide sickness (`conn_sick`), silence is booked to the channel ledger and node-side evidence is frozen. One hour of outage produced **zero false OFFLINE verdicts**.
3. **Self-audit** — every wall-clock constant has a derivation and a track record; the false-positive rate goes on the monthly books. Before monitoring others, we monitor ourselves.

## Built for HPC & AI clusters

- **IO-storm detection** — NFS D-state signatures, load/CPU divergence, disk-age analysis
- **GPU job-level accounting** — waste leaderboards ranked by money, not percentages
- **Straggler detection** — find the one node holding up a 3,000-rank MPI barrier
- **Digital twin** — self-audited prediction accuracy *and* consistency (calm-period stability, not just calibration)
- **Observer self-protection** — capacity governor, circuit breakers, and bounded-honesty degrade modes that reset themselves when the channel recovers

---

**LinuxMonitorInsight** — *We don't sell a thermometer. We sell a forensic accountant for your compute assets.*
