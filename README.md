<div align="center">

# PolicyGraph

### The reasoning layer for incident response.

**From alert to root cause in seconds — not hours.**

PolicyGraph turns raw alerts into ranked, evidence-backed root cause hypotheses. Every conclusion is traceable. Every action is policy-controlled. Every investigation makes the next one smarter.

[Why PolicyGraph](#why-policygraph) · [How it works](#how-it-works) · [Capabilities](#capabilities) · [Trust by design](#trust-by-design) · [Roadmap](#roadmap)

---

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python 3.12+](https://img.shields.io/badge/python-3.12%2B-blue.svg)](pyproject.toml)
[![Tests: 146 passing](https://img.shields.io/badge/tests-146%20passing-brightgreen.svg)](tests/)
[![Status: Production sprint](https://img.shields.io/badge/status-production%20sprint-orange.svg)](FUTURE_RELEASES.md)

</div>

---

## The problem

When PagerDuty fires at 3 AM, the on-call engineer opens twelve tabs.

Logs in one place. Metrics in another. Recent deploys in a third. K8s state in a fourth. Slack threads in a fifth. They spend the next forty minutes correlating timestamps in their head, hunting for the one config change that broke production.

This is the **largest unautomated cost in modern operations.** Not the outage itself — the *time-to-understanding* before the outage can even be addressed. Across mid-size SaaS organizations, MTTR averages 4–6 hours, with **70% of that time spent in investigation**, not remediation.

LLM chatbots don't fix this. They guess. They hallucinate root causes. They have no access to your cluster, no memory of your past incidents, no understanding of which signals to trust.

**PolicyGraph is the reasoning layer that does.**

---

## Why PolicyGraph

> **PolicyGraph is to incident response what an experienced staff engineer is to a war room — except it never sleeps, never forgets, and runs in parallel across every alert simultaneously.**

|                                | Traditional AIOps                  | LLM chatbot                          | **PolicyGraph**                                                        |
| ------------------------------ | ---------------------------------- | ------------------------------------ | ---------------------------------------------------------------------- |
| Multi-source investigation     | Limited to ingested telemetry      | None — only what's pasted into chat  | **Live K8s, logs, GitLab, Jira, Confluence, Slack — in parallel**      |
| Root cause reasoning           | Pattern matching on past incidents | Plausible-sounding text              | **Deterministic correlation + LLM re-ranking on ambiguous cases**      |
| Confidence calibration         | Implicit                           | Always 100% confident, often wrong   | **Every hypothesis carries a numeric confidence + supporting evidence** |
| Memory across incidents        | Static rules                       | None                                 | **Persistent knowledge graph — every investigation teaches the next** |
| Customization                  | Vendor-locked                      | Prompt engineering                   | **Skill-driven — investigators write their own playbooks in markdown**|
| Action safety                  | Manual approval                    | Suggest only                         | **Policy-controlled execution gateway with HITL on high-risk actions**|
| Auditability                   | Partial                            | Zero                                 | **Every signal, every conclusion, every action — fully traceable**    |

---

## How it works

```
┌──────────────────────────────────────────────────────────────────────┐
│                         PolicyGraph                                  │
│                                                                      │
│   Alert                                                              │
│     │                                                                │
│     ▼                                                                │
│   Intake & Normalization  ─────►  Skill matching                     │
│     │                              (markdown playbooks)              │
│     ▼                                                                │
│   ┌────────────────────────────────────────────┐                     │
│   │   Investigation chains run in parallel     │                     │
│   │                                            │                     │
│   │   ▸ Kubernetes  (pods, events, PVCs, …)    │                     │
│   │   ▸ Logs        (Cloud Logging, errors)    │                     │
│   │   ▸ Change      (GitLab commits, MRs)      │                     │
│   │   ▸ +pluggable  (Jira, Slack, custom…)     │                     │
│   └────────────────────────────────────────────┘                     │
│     │                                                                │
│     ▼                                                                │
│   Correlation engine                                                 │
│     ├─ Deterministic joins (causal patterns, time, entity)           │
│     ├─ Hypothesis ranking (configurable scoring)                     │
│     └─ LLM re-rank on ambiguous cases (skill-driven prompt)          │
│     │                                                                │
│     ▼                                                                │
│   Output                                                             │
│     ├─ Ranked root cause hypotheses with evidence                    │
│     ├─ Unified incident timeline                                     │
│     ├─ Entity relationship graph                                     │
│     └─ Suggested remediation actions                                 │
│     │                                                                │
│     ▼                                                                │
│   Knowledge graph  ◄────────  every investigation feeds the next     │
└──────────────────────────────────────────────────────────────────────┘
```

The system is **deterministic where it can be, and intelligent where it must be.** Joins, scoring, and entity correlation are mechanical and reproducible. The LLM is reserved for the genuinely ambiguous cases — and even there, it operates from skill-defined prompts loaded as markdown, not from a hardcoded prompt.

---

## Capabilities

### 🧠 Multi-chain investigation
Three production-ready investigation chains ship today, with a clean interface to add your own:

- **Kubernetes** — pods, events, PVCs, quotas, node pressure (read-only via [`kr8s`](https://kr8s.org)).
- **Logs** — Google Cloud Logging with pattern matching for OOM, dependency failures, and provisioning errors.
- **Change Intelligence** — GitLab commits and merge requests in the incident time window, scored by file-pattern risk.

Chains run **in parallel** with bounded timeouts. Partial results never block the rest.

### 🔗 Cross-cluster, cross-project resolution
The Cluster Resolver maps services to GKE clusters across GCP projects, so an alert in one project can trigger investigation in the right cluster automatically. No more "wrong kubectl context" mistakes at 3 AM.

### 🎯 Deterministic correlation, intelligent ranking
- **Joins** by service, namespace, time overlap, and configured causal patterns.
- **Hypothesis scoring** driven by `config/ranker.yaml` — severity weights, root cause categories, and causal pattern bonuses are all data, not code.
- **LLM re-ranking** kicks in only when the deterministic ranker can't separate top hypotheses. The system prompt itself lives in [`config/skills/system/llm-ranker/SKILL.md`](config/skills/system/llm-ranker/SKILL.md) — editable by anyone on the team.

### 📚 Skill-driven everything
Investigators write **alert skills** as markdown files in [`config/skills/alerts/`](config/skills/). Each skill maps an alert pattern to investigation guidance and remediation hints — no Python, no YAML schema gymnastics.

System skills (LLM prompts for ranking, Slack reports, Jira tickets) live in [`config/skills/system/`](config/skills/) and are loaded at startup. Change a prompt, restart, done.

### 🧬 Persistent incident memory
Every investigation is stored — locally as JSON by default, or in [Cognee](https://github.com/topoteretes/cognee) for full knowledge-graph search. When a new incident comes in, similar past incidents are automatically surfaced to inform ranking and reduce repeat root-cause analysis.

### ⚡ Built for scale from day one
- **Distributed queue** — [Arq](https://arq-docs.helpmanual.io/) + Redis. Multiple worker processes consume investigations in parallel. The API stays responsive.
- **Stateless API + workers** — share state via Redis-backed incident store. Horizontally scale either tier independently.
- **Graceful degradation** — Redis unreachable? API runs investigations synchronously in-process. LLM unreachable? Deterministic ranker still ships ranked hypotheses.

### 🔐 Trust by design
- **Read/write separation enforced in code** — `K8sReader` has no write methods. `K8sExecutor` is gated behind a future policy engine.
- **Every conclusion carries supporting evidence** — pod names, log refs, commit SHAs, MR links. No "the AI said so."
- **Confidence is a first-class field** — calibrated, configurable, never inflated.
- **Audit-ready** — full investigation history preserved, including the chain results, the timeline, the graph, and the LLM rationale (when used).

---

## What makes it different

### 🎓 The architecture of a senior engineer
Most "AI for incidents" products are a wrapper around an LLM. PolicyGraph is the opposite: a **deterministic reasoning system** that uses LLMs sparingly, where they actually help.

This matters because hallucinations in incident response are not a UX problem — they cause production outages.

### 🪟 Glass-box, not black-box
Every hypothesis is explainable. Every score is configurable. Every prompt is editable. There is no "trust the model" — there is only "trust the evidence."

### 🪶 Lives in your infrastructure, not someone else's cloud
PolicyGraph runs entirely in your environment. Your alerts, your logs, your secrets never leave. The LLM provider is pluggable — bring OpenAI, bring Anthropic, bring an on-prem model.

### 🧱 Designed to compound
The first ten incidents are useful. The hundredth is dramatically better, because the knowledge graph has learned the shape of *your* infrastructure, *your* failure modes, *your* remediation playbooks.

This is a moat that gets deeper with use.

---

## What ships today

| Component                              | Status     | Notes                                                |
| -------------------------------------- | ---------- | ---------------------------------------------------- |
| Multi-chain investigation              | ✅ Shipped  | Kubernetes, Logs, Change                            |
| Deterministic correlation engine       | ✅ Shipped  | Configurable via `config/ranker.yaml`               |
| LLM-driven re-ranking                  | ✅ Shipped  | Skill-driven prompt, fully optional                 |
| Skills system (alerts + system)        | ✅ Shipped  | Markdown-based, hot-loadable                         |
| Service registry (services.yaml)       | ✅ Shipped  | Maps services → log queries, repos, clusters        |
| Multi-cluster resolver                 | ✅ Shipped  | Cross-GCP-project K8s investigation                 |
| Persistent incident memory             | ✅ Shipped  | Local JSON or Cognee knowledge graph                |
| Distributed queue (Arq + Redis)        | ✅ Shipped  | Stateless API + horizontally-scaling workers        |
| Provider integrations                  | ✅ Shipped  | GCP Logging, GitLab, Jira, Confluence, Slack         |
| Test coverage                          | ✅ Shipped  | 146 tests across all critical paths                 |
| API authentication & webhook HMAC      | 🚧 In sprint| Auth middleware + signed webhooks                   |
| PostgreSQL persistence + audit ledger  | 🚧 In sprint| Permanent storage, immutable audit trail            |
| Docker + Helm                          | 🚧 In sprint| Production deployment artifacts                     |
| Policy-controlled execution gateway    | 📋 v0.5    | HITL approval flow for write actions                |
| Self-hosted UI                         | 📋 v0.6    | Investigator + on-call console                      |

See [`FUTURE_RELEASES.md`](FUTURE_RELEASES.md) for the full roadmap.

---

## The opportunity

Incident response is a **$30B+ market** today (Datadog, Splunk, PagerDuty, ServiceNow combined). Every major vendor in this space is bolting LLMs onto pre-AI architectures — and producing exactly the hallucination-prone "summaries" that on-call engineers ignore.

PolicyGraph is built **AI-first, deterministic-first, evidence-first.** It is the first incident response system designed for the era where LLMs are tools, not oracles.

The wedge is multi-chain, evidence-backed root cause analysis. The expansion is autonomous remediation under policy control. The moat is the knowledge graph that compounds with every incident.

---

## Trust by design

PolicyGraph is engineered for environments where wrong answers cause outages and incorrect actions cause data loss.

- **No write operations without policy approval.** The execution gateway (v0.5) enforces this in code.
- **No conclusion without evidence.** Every hypothesis includes the signals, log entries, and commits that support it.
- **No surprises.** Every prompt, every score, every join is editable configuration. Nothing is hidden in code.
- **No vendor lock-in.** Storage backends, LLM providers, alert sources, and chains are all pluggable interfaces.
- **No data egress.** Runs entirely in your environment. Bring your own LLM endpoint.

---

## Roadmap

### v0.2 — Production Hardening *(in flight)*
API authentication, webhook HMAC verification, PostgreSQL persistence with immutable audit ledger, Dockerfile + docker-compose, CI/CD.

### v0.3 — Execution Gateway
Policy-controlled write actions. HITL approval flow integrated with Slack and Jira. K8sExecutor with rollback support.

### v0.4 — Strategy Memory
Active wiring of incident memory into LLM prompts. The system learns "for incidents like this, the remediation that worked was X."

### v0.5 — Investigator Console
Self-hosted web UI for browsing investigations, replaying timelines, and approving actions.

### v0.6 — Observability Provider Expansion
Datadog, New Relic, Splunk, Elasticsearch, AWS CloudWatch.

### v0.7 — Multi-tenant
Tenant isolation, per-tenant skills, per-tenant memory.

### v0.8 — Autonomous Remediation
Closed-loop execution for low-risk, high-confidence remediation under strict policy bounds.

---

## License

MIT — built to be adopted, extended, and deployed everywhere.

---

<div align="center">

**PolicyGraph turns 3 AM into a non-event.**

</div>
