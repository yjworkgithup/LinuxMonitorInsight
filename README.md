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

Here is the full English translation of the comparison table.

---

## A More Complete and Honest Comparison

| Dimension | Zabbix | Datadog | Sensu Go | Telegraf+InfluxDB | **LinuxMonitorInsight** |
|---|---|---|---|---|---|
| **Deployment Model** | Agent (Zabbix Agent 2) + SNMP | Heavy Agent + Kernel Modules | Agent (sensu-agent) | Agent (telegraf) | **Zero Agent, Pure SSH** |
| **Agent Intrusiveness** | Medium (install, configure, upgrade) | High (kernel probes, eBPF) | Medium (Go binary) | Medium (complex plugin ecosystem) | **Zero Footprint** |
| **Offline Node Semantics** | Unavailable (binary) | No Data (manual config required) | unknown (no alert by default) | Series gap (requires `fill()`) | **UNKNOWN is a first-class citizen with dedicated UI color** |
| **Channel Fault Attribution** | ❌ None | ❌ None | ❌ None | ❌ None | **✅ `conn_sick` separates channel vs. node responsibility** |
| **Behavior During Mass Outage** | Alert avalanche | Alert avalanche + bill explosion | Silent data loss | Series holes | **✅ Zero false positives, state machine freezes evidence** |
| **Threshold Source** | Magic numbers + templates | ML "magic" (black box) | Magic numbers | Continuous query config | **✅ Measurement-derived + audit flywheel** |
| **Self-Check / Self-Audit** | ❌ | Yes (but paid) | ❌ | ❌ | **✅ precision@horizon + monthly false-positive rate** |
| **HPC-Native Features** | ❌ | GPU monitoring (extra fee) | ❌ | ❌ | **✅ IO storm, straggler, GPU job accounting** |
| **Digital Twin / Prediction** | ❌ | Yes (Forecast, enterprise edition) | ❌ | ❌ | **✅ EWMA + Welford + self-audited accuracy** |
| **Open Source License** | GPL v2 | Commercial (per-host billing) | MIT | MIT | **(Your license?)** |
| **Community / Ecosystem** | Large (20+ years) | Massive (enterprise standard) | Small (CNCF fringe) | Large (InfluxData ecosystem) | **Individual / small team maintenance** |
| **Learning Curve** | Steep | Gentle (but deep) | Medium | Medium | **Steep (high concept density)** |
| **Scalability Validation** | 10K+ nodes | 100K+ containers | Medium | High (time-series write) | **✅ 5K nodes + 1h outage real-world tested** |

---

## What We Are NOT Good At (Be Honest)

| Scenario | Why Choose the Competitor |
|---|---|
| **Container / K8s-native monitoring** | Prometheus + Grafana is the de facto standard; we have Kubernetes SD but not its ecosystem |
| **Log aggregation** | Loki / ELK are proper logging systems; we only collect metrics |
| **APM / Distributed tracing** | Jaeger / Tempo are proper tracing; we do not cover this |
| **SaaS hosting** | Datadog / New Relic are turnkey; you must operate this yourself |
| **Non-Linux platforms** | Windows / AIX / network gear → Zabbix / SNMP is more suitable |
| **Long-term historical trends** | InfluxDB / TimescaleDB are optimized for time-series; we use in-memory state machines |
| **Multi-tenant / ACL** | Zabbix has mature user permission systems; we assume single-team use |

---

## Maintainer Risk

> **Single-point maintainer**: The current core code is maintained by an individual with no foundation backing. If you need 7×24 commercial support, Zabbix or Datadog is more suitable.
>
> **Concept barrier**: `conn_sick`, `epistemic state machine`, and `digital twin` are not everyday vocabulary for operations teams. Expect 2–4 hours of conceptual training before adoption.
