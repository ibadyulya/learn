# Observability (logs, metrics, traces)

> 🌐 Russian version: [11-observability.md](../ru/11-observability.md)

How to understand what's happening inside a distributed system when "just attach a
debugger" isn't possible. Reads as a story; adjacent to
[microservices](./08-microservices-vs-monolith.md) and
[error handling](../../04-backend/en/05-validation-error-handling.md).

The through-line of the topic:

> **Observability — the ability to understand a system's internal state from its
> external signals. Three pillars: logs (what happened), metrics (how much and how
> fast), traces (a request's path through services). In distribution, without them
> you're blind: a request goes through a dozen services, and "poke into one to look"
> doesn't work.**

---

## Why, and how it differs from monitoring

In a monolith you can attach a debugger; in distribution a request passes through many
processes and machines — you need to understand the system **from external signals**,
without going inside.
- **Monitoring** — watching **known** metrics (CPU, errors): answers questions you **asked
  in advance** ("known unknowns").
- **Observability** — the ability to ask a **new** question about behavior **after the
  fact**, without redeploying: to diagnose what you didn't foresee ("unknown unknowns").
Monitoring is a subset; the goal is to be able to answer "why slow/broken" from
already-collected signals.

---

## The three pillars

**1. Logs — "what happened".** Records of events. Key — **structured** (JSON, not free
text): you can search and aggregate by fields. A **correlation/trace id** is mandatory to
tie one request's records across all services. Levels (error/warn/info/debug). Don't log
secrets/PII.

**2. Metrics — "how much and how fast".** Numeric aggregates over time: RPS, latency
(p50/p95/p99), error rate, resource usage. Cheap to store, feed **dashboards and alerts**.
Useful sets:
- **RED** (for services): Rate (requests), Errors, Duration (latency);
- **USE** (for resources): Utilization, Saturation, Errors.

**3. Traces — "the request's path".** **Distributed tracing**: one request gets a **trace
id**, and each step in each service — a **span**; spans stitch into a tree, showing where
time went and which service lagged/failed. Indispensable in microservices. The standard —
**OpenTelemetry**.

---

## What ties it all together

A **correlation id (trace id)** — a shared identifier set at the entry (the gateway) and
**propagated** through all services and into all logs/spans. By it you assemble the full
picture of one request: a log showed an error → by the trace id you found the trace → saw
which service and step → looked at the detailed logs there.

---

## Alerting — on symptoms, not noise

Alerts — on what **affects the user** (rising errors, p99 degradation, an SLO breach), not
on every twitch (one metric spiked). Too many alerts → **fatigue** and ignoring (like
flaky tests). Rule: **alert on symptoms**, diagnose the cause from metrics/traces/logs.

---

## Interview phrasing

> "Observability — understanding a system's internal state from external signals; in
> distribution you're blind without it. Three pillars: logs — what happened (structured,
> with a trace id, no secrets); metrics — numeric aggregates over time (RPS, p95/p99,
> error rate; the RED set for services, USE for resources) feeding dashboards and alerts;
> traces — distributed tracing, where a request carries a trace id and spans stitch into a
> tree showing where time was lost. It's all tied by a correlation/trace id propagated
> through services. I alert on symptoms/SLO, not noise, to avoid fatigue. The
> instrumentation standard is OpenTelemetry."

---

See [11-observability.tasks.md](./11-observability.tasks.md) — tasks. Solutions in
[11-observability.answers.md](./11-observability.answers.md).
