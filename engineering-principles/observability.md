# Observability

## What This Principle Means

Observability is the ability to understand the internal state of a system from its external outputs — logs, metrics, and traces — without deploying new instrumentation every time a new question arises. It is the foundation of **debugging, capacity planning, incident response, and continuous improvement**.

Observability is not monitoring alone. Monitoring tells you when predefined thresholds are breached. Observability lets you ask **arbitrary questions** about system behaviour — "why did latency spike for this tenant at 14:32?" — and get answerable signals.

The three pillars:

- **Metrics** — aggregated, time-series measurements (latency, error rate, throughput, saturation)
- **Logs** — discrete events with context (structured, searchable, correlated)
- **Traces** — request paths across services (distributed tracing)

---

## Why It Matters in Cloud Architecture

Distributed systems fail in ways that are invisible from a single server log. A request may traverse API Gateway, Lambda, SQS, ECS, RDS, and a third-party API. Without correlated telemetry, incidents become guesswork — and mean time to recovery (MTTR) grows.

Cloud-native services emit rich telemetry, but **default metrics are rarely sufficient** for business-level understanding. You must design what to measure, where to emit it, how long to retain it, and who gets alerted.

Observability also supports **SLOs and error budgets** — the operational contract between reliability targets and release velocity. You cannot manage an error budget you cannot measure.

---

## Design Questions to Ask

- What are the **SLIs** (Service Level Indicators) — latency, availability, error rate, throughput — for this workload?
- What **SLOs** does the business require, and what is the error budget?
- What **metrics** are emitted at each tier — infrastructure, platform, application, business?
- Are logs **structured** (JSON) with consistent fields — trace ID, tenant ID, request ID?
- Is **distributed tracing** implemented across service boundaries?
- What is the **alerting strategy** — symptom-based vs cause-based; who gets paged vs emailed?
- What is **retention** for logs and metrics — compliance, debugging, cost?
- Where do dashboards live, and are they **actionable** for on-call engineers?
- Can we **correlate** a user report to a specific trace, log line, and deployment?
- What **synthetic checks** or canaries exist for critical paths?
- Are observability tools **included in cost planning** — CloudWatch, X-Ray, third-party APM?
- What happens to telemetry when the system is **under failure** — do logs and metrics still flow?

---

## Common Mistakes to Avoid

| Mistake | Why it hurts |
|---------|--------------|
| Alerting on every metric threshold | Alert fatigue; real incidents missed |
| Unstructured printf-style logs | Cannot query or correlate at scale |
| No trace ID propagation across services | Cannot follow a request end-to-end |
| Logging sensitive data (PII, tokens, passwords) | Compliance violation and security risk |
| Metrics without labels/dimensions | Cannot slice by tenant, region, or version |
| Observability added after launch | Blind during the most fragile period |
| Dashboards that only show green | No visibility into degradation before failure |
| Infinite log retention without cost consideration | Bill shock from CloudWatch Logs |
| Relying only on infrastructure metrics | Application-level failures invisible |
| No runbook link from alerts | On-call receives a page with no context |

---

## In Practice

Design observability **with the architecture**, not after. Every new service boundary is a place for metrics, structured logs, and trace propagation.

Start with **user- and business-visible symptoms** — can users complete checkout, is API latency within SLO — then drill down to infrastructure causes.

Review dashboards and alerts quarterly. Remove noise, add coverage for incidents that were hard to diagnose.

Observability supports the Opare Method question: **What could fail?** — and the follow-up: **How will we know when it does?**
