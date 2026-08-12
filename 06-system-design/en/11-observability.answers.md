# Observability — solutions

> 🌐 Russian version: [11-observability.answers.md](../ru/11-observability.answers.md)

---

## Question 1
In microservices a request passes through **many processes and machines**; you can't
attach one debugger to "the whole system", and logs are scattered across services. You can
understand behavior only **from external signals** (logs/metrics/traces) collected
centrally. Without them, during an incident you can't see **which service and step** the
problem is at.

## Question 2
- **Monitoring** — watching **predefined** metrics and alerting (CPU, errors): answers
  questions you foresaw (**known unknowns**).
- **Observability** — a property of the system letting you **ask a new question after the
  fact** about its behavior from already-collected data, without redeploying (**unknown
  unknowns**).
Monitoring is a special case; observability is broader.

## Question 3
- **Logs** — "what happened" (discrete events);
- **Metrics** — "how much and how fast" (numeric aggregates over time);
- **Traces** — "which path the request took" (through which services and where it spent
  time).

## Question 4
**Structured logs** (JSON with fields) can be **searched, filtered, and aggregated** by
machine, rather than grepping free text. A **correlation/trace id** — a shared request
identifier set at the entry and **propagated** into all services and their logs/spans: by
it you gather **all** the records of one request across the whole system and reconstruct
the picture.

## Question 5
**Percentiles**: p95 = "95% of requests are faster than this value". They show the **tail**
of the distribution — the slow requests that hit real users. The **average** hides the
tail: with lots of fast ones and a few very slow ones the average looks fine, while p99
looks terrible (and that's what users feel). **RED** (service): Rate, Errors, Duration.
**USE** (resource): Utilization, Saturation, Errors.

## Question 6
**Distributed tracing** follows **one request across all services**. The request gets a
**trace id**; each step/operation — a **span** (with start time/duration, a parent). Spans
stitch into a **trace tree**, showing **which service and step** took how long and where a
failure occurred — the bottleneck in the chain is immediately visible.

## Question 7
Alert on **symptoms that affect the user** (a rising error rate, p99/p95 degradation, an
SLO breach), not on every twitching metric or technical indicator with no impact. Too many
alerts → **fatigue** and ignoring real incidents. After an alert, diagnose the cause from
metrics/traces/logs. Alerts should be **actionable** (there's something to do) and **rare**
relative to noise.
