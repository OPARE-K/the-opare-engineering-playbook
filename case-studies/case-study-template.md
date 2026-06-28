# Case Study: [System or Scenario Name]

> Copy this template into a new directory (e.g., `case-studies/01-event-driven-order-processing/README.md`). Replace all placeholders. Delete instructional comments before publishing.

**Author:** [Name]  
**Date:** YYYY-MM-DD  
**Status:** Draft | Published | Archived  
**Related ADRs:** [ADR-0001](../architecture-decision-records/0001-example.md), [ADR-0002](../architecture-decision-records/0002-example.md)

---

## Business Problem

What problem does this system solve, and for whom?

- Stakeholders and users
- Business outcome and success criteria
- Why now — urgency, regulatory, competitive, technical debt
- Scope boundaries — what this system does and does not do

---

## Assumptions

What are we assuming to be true for this design to hold?

| Assumption | Impact if wrong |
|------------|-----------------|
| [e.g., Peak traffic ≤ 5k RPS] | [Would require horizontal scaling redesign] |
| [e.g., Data residency in eu-west-1 only] | [Multi-region redesign required] |

---

## Requirements

### Functional

- [Core capability 1]
- [Core capability 2]
- [Integration requirements]

### Non-Functional

| Attribute | Target |
|-----------|--------|
| Availability | e.g., 99.9% |
| Latency (p95) | e.g., < 200ms |
| Throughput | e.g., 2k RPS at launch |
| RTO / RPO | e.g., 4h / 1h |
| Data retention | e.g., 7 years audit logs |

---

## Architecture Overview

High-level description of components, boundaries, and responsibilities.

```
[Insert architecture diagram — link or embed]
```

| Component | Responsibility | AWS Service |
|-----------|----------------|-------------|
| [Name] | [What it does] | [Service] |

---

## Traffic Flow

Describe how a typical request or data flow moves through the system.

```
[Insert sequence or traffic flow diagram]
```

1. [Step 1 — e.g., Client → CloudFront → ALB]
2. [Step 2]
3. [Step 3]

Note synchronous vs asynchronous paths, retry boundaries, and idempotency points.

---

## AWS Services and Justification

| Service | Purpose | Why this service |
|---------|---------|------------------|
| [e.g., Amazon SQS] | [Async order processing] | [Decouples producers/consumers; handles burst; DLQ for poison messages] |

Link each choice to [engineering principles](../engineering-principles/README.md) where relevant.

---

## Alternatives Considered

| Alternative | Pros | Cons | Verdict |
|-------------|------|------|---------|
| [e.g., Synchronous API chain] | Simpler initial build | Tight coupling; cascade failures | Rejected |
| [e.g., Kafka on EC2] | Full control | Operational overhead | Rejected for v1 |

---

## Security Design

- **Data classification:** [Levels and where data lives]
- **Encryption:** At rest (KMS), in transit (TLS)
- **Network segmentation:** [VPC, subnets, endpoints]
- **Secrets:** [Secrets Manager, rotation]
- **Threat considerations:** [Relevant threats and mitigations]

See [Security by Design](../engineering-principles/security-by-design.md).

---

## Networking Design

- **VPC layout:** CIDR, subnets, AZs
- **Ingress:** [ALB, API Gateway, CloudFront, WAF]
- **Egress:** [NAT, VPC endpoints]
- **Security groups / NACLs:** Key rules and rationale
- **DNS and certificates:** [Route 53, ACM]
- **Data transfer notes:** Cross-AZ, egress cost considerations

---

## IAM Design

- **Human access:** [SSO roles, break-glass]
- **Service roles:** [Per-workload IAM roles, least privilege]
- **Cross-account / cross-service:** [If applicable]
- **Policy highlights:** [Key permissions and denials]

---

## Reliability Design

- **Multi-AZ:** [Where and why]
- **Health checks and auto-healing**
- **Backup and restore**
- **Dependency failure behaviour**
- **SLO / error budget**

See [Reliability](../engineering-principles/reliability.md).

---

## Failure Analysis

| Failure Mode | Detection | Impact | Mitigation / Recovery |
|--------------|-----------|--------|------------------------|
| [e.g., AZ outage] | [Health checks, Route 53] | [Reduced capacity] | [Multi-AZ failover] |
| [e.g., Downstream API timeout] | [Latency alarms] | [Degraded checkout] | [Circuit breaker, queue] |
| [e.g., Bad deployment] | [Error rate spike] | [5xx for subset of users] | [Rollback, canary] |

Include blast radius analysis and cascading failure paths.

---

## Monitoring and Logging

| Signal | Source | Alert / Dashboard |
|--------|--------|-------------------|
| [e.g., API error rate] | [CloudWatch / ALB metrics] | [PagerDuty if > 1% for 5m] |
| [e.g., Queue depth] | [SQS metrics] | [Scale consumers] |

- **Structured logging:** Fields, retention
- **Tracing:** [X-Ray or equivalent]
- **Synthetic checks:** [If any]

See [Observability](../engineering-principles/observability.md).

---

## Cost Analysis

| Component | Est. monthly (launch) | Est. monthly (projected) | Notes |
|-----------|----------------------|--------------------------|-------|
| [Compute] | $ | $ | |
| [Storage] | $ | $ | |
| [Data transfer] | $ | $ | |
| **Total** | $ | $ | |

- Largest cost drivers
- Optimisation opportunities deferred
- Tagging for allocation

See [Cost Awareness](../engineering-principles/cost-awareness.md).

---

## Terraform Structure

> Document when IaC is part of the case study. Phase 0 contains no Terraform — this section is for future use.

```
modules/
├── networking/
├── compute/
└── data/
environments/
├── dev/
├── staging/
└── prod/
```

- **Module boundaries and rationale**
- **State management** — backend, locking, per-env state
- **Variable and output conventions**
- **What is intentionally not in Terraform** (if anything)

---

## Testing Approach

| Layer | What is tested | How |
|-------|----------------|-----|
| Unit | [Business logic] | [Framework] |
| Integration | [Service boundaries] | [Localstack, test containers, dev env] |
| Load | [Throughput, latency] | [k6, Locust, etc.] |
| Chaos / resilience | [Failure injection] | [Game days, fault injection] |
| Security | [IAM, exposure] | [Scanner, manual review] |

---

## Interview Questions

Questions a reviewer or interviewer might ask — with brief pointers to where the answer lives in this document.

1. **Why did you choose [service X] over [service Y]?** → [Alternatives Considered]
2. **What happens when [dependency] fails?** → [Failure Analysis]
3. **How would you scale this to 10x traffic?** → [Reliability / Scalability sections]
4. **What is the biggest security risk in this design?** → [Security Design]
5. **What would you do differently with more time or budget?** → [Future Improvements]

---

## Lessons Learned

Honest retrospective — what worked, what didn't, what surprised you.

- **What worked well:**
- **What we'd change:**
- **What we underestimated:**

---

## Future Improvements

| Improvement | Trigger | Priority |
|-------------|---------|----------|
| [e.g., Multi-region active-active] | [Traffic in second geography] | Medium |
| [e.g., Replace polling with event-driven] | [Queue depth sustained high] | Low |

---

## References

- [Architecture review checklist](../architecture-review-framework/README.md)
- [Related ADRs](../architecture-decision-records/)
- [AWS documentation links]
- [External references](../references/README.md)
