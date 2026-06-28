# Architecture Review Framework

A reusable checklist for reviewing cloud architecture designs before build, before major change, and before production promotion. Use it in design reviews, ADRs, and as a self-assessment before presenting to stakeholders.

This framework maps to the [engineering principles](../engineering-principles/README.md) and the [Opare Method](../README.md#the-opare-method). Not every item applies to every workload — mark N/A with brief justification rather than skipping silently.

---

## How to Use This Framework

1. **Prepare context** — business problem, constraints, diagrams, draft ADRs.
2. **Walk each section** — assign owner, capture answers, flag gaps.
3. **Record trade-offs** — where sections conflict, document the decision.
4. **Produce actions** — blockers before build, improvements for backlog.
5. **Archive** — attach completed checklist to ADR or review record.

A design is ready to proceed when blockers are resolved or explicitly accepted as documented risk.

---

## Finding Severity

Every review finding must be prioritised — not treated equally. Assign one severity per finding when walking the checklist.

| Severity | Label | Meaning | Action |
|----------|-------|---------|--------|
| **P0** | Launch Blocker | Unacceptable risk to security, data, availability, or compliance if shipped as-is | Must resolve or escalate as accepted risk with named owner and executive sign-off before production |
| **P1** | Significant Risk | Material gap — likely incident, cost overrun, or operational failure under realistic conditions | Must have mitigation plan, owner, and timeline before launch; may proceed with documented accepted risk |
| **P2** | Improvement Opportunity | Valid gap that does not block launch — reduces debt, improves operability, or strengthens future scale | Backlog item with owner; track but do not hold release |

**Rules:**

- A checklist item is not complete because it was discussed — it is complete when the finding has a severity and disposition (fixed, mitigated, accepted, deferred).
- Multiple P0 findings mean the design is not ready. Do not dilute P0 by overusing it — reserve for genuine launch blockers.
- P2 items must not silently become accepted permanent gaps. Assign a revisit trigger or backlog reference.
- Accepted P0 or P1 risks appear in the review output under **Accepted Risks** with justification and owner.

When recording findings in the review summary, prefix each item: `[P0]`, `[P1]`, or `[P2]`.

---

## Review Checklist

### Business Problem

- [ ] What business outcome does this system enable?
- [ ] Who are the stakeholders and users?
- [ ] What does success look like — measurable criteria?
- [ ] What is in scope and explicitly out of scope?
- [ ] What is the timeline and what happens if we slip?

### Functional Requirements

- [ ] What are the core capabilities the system must provide?
- [ ] What are the integration points — upstream and downstream systems?
- [ ] What data flows in, out, and through the system?
- [ ] Are there regulatory or compliance functional requirements?
- [ ] What are the expected usage patterns — steady, bursty, batch?

### Non-Functional Requirements

- [ ] Availability target (e.g., 99.9%) and acceptable downtime window
- [ ] Latency targets — p50, p95, p99 where relevant
- [ ] Throughput / requests per second at launch and projected scale
- [ ] RTO and RPO for disaster recovery
- [ ] Data retention and deletion requirements
- [ ] Concurrency and session expectations

### Networking

- [ ] VPC design — CIDR, subnets (public/private/isolated), AZ layout
- [ ] Ingress path — ALB, NLB, API Gateway, CloudFront, WAF
- [ ] Egress path — NAT Gateway, VPC endpoints, internet gateway
- [ ] DNS and certificate management (Route 53, ACM)
- [ ] Cross-VPC or cross-account connectivity if applicable (TGW, peering, PrivateLink)
- [ ] Security groups and NACLs — least privilege, documented rules
- [ ] Data transfer paths and cost implications (cross-AZ, cross-region, egress)

### Security

- [ ] Data classification and handling requirements
- [ ] Threat model — assets, actors, attack surfaces
- [ ] Encryption at rest — services, keys (KMS), rotation
- [ ] Encryption in transit — TLS everywhere across trust boundaries
- [ ] Secrets management — Secrets Manager, Parameter Store, no secrets in code
- [ ] WAF, Shield, DDoS considerations for public endpoints
- [ ] Vulnerability management — patching, scanning, dependency updates
- [ ] Audit logging — CloudTrail, Config, application audit trail
- [ ] Compliance requirements (SOC 2, PCI, HIPAA, GDPR) mapped to controls

### IAM

- [ ] Human access model — SSO, roles, no long-lived console passwords
- [ ] Service-to-service identity — IAM roles, not access keys
- [ ] Least privilege — scoped policies, resource-level permissions
- [ ] Separation of duties — who can deploy, who can access data, who can administer IAM
- [ ] Cross-account access pattern if applicable
- [ ] Break-glass / emergency access procedure documented
- [ ] IAM policy review process — no `*` on production resources without justification

### Reliability

- [ ] Single points of failure identified and mitigated or accepted
- [ ] Multi-AZ deployment where required by SLO
- [ ] Health checks — liveness vs readiness, dependency checks
- [ ] Timeouts, retries, circuit breakers on external calls
- [ ] Idempotency for retried operations
- [ ] Graceful degradation under partial failure
- [ ] Backup strategy and restore testing plan
- [ ] Dependency SLOs understood — what happens when they fail?

### Scalability

- [ ] Scaling dimensions identified — compute, storage, read, write
- [ ] Bottleneck analysis at 2x and 10x projected load
- [ ] Auto-scaling triggers and limits defined
- [ ] Stateless compute where horizontal scaling is required
- [ ] Caching strategy for hot paths
- [ ] Service quotas and rate limits reviewed
- [ ] Load testing plan before production traffic

### Observability

- [ ] SLIs and SLOs defined
- [ ] Metrics — infrastructure and application — with dimensions
- [ ] Structured logging with correlation IDs
- [ ] Distributed tracing across service boundaries
- [ ] Dashboards for on-call and stakeholders
- [ ] Alerting strategy — symptom-based, runbook links, escalation
- [ ] Log and metric retention aligned with compliance and cost
- [ ] Synthetic monitoring or canaries for critical paths

### Cost

- [ ] Estimated monthly cost at launch and at projected scale
- [ ] Largest cost drivers identified
- [ ] Tagging strategy for allocation (team, env, product)
- [ ] Budgets and anomaly alerts configured
- [ ] Right-sizing and reserved capacity strategy
- [ ] Data transfer and storage tier choices justified
- [ ] Cost of redundancy and DR included in estimate

### Operations

- [ ] Deployment pipeline — CI/CD, approvals, environments
- [ ] Rollback strategy documented and tested
- [ ] Infrastructure as code — Terraform/CDK, review process
- [ ] Runbooks for common incidents
- [ ] On-call model and escalation path
- [ ] Change management — windows, notifications, feature flags
- [ ] Environment parity — dev/staging/prod consistency
- [ ] Post-incident review process

### Disaster Recovery

- [ ] DR strategy — backup/restore, pilot light, warm standby, multi-region active
- [ ] RTO and RPO aligned with business requirements
- [ ] DR region and failover mechanism documented
- [ ] Data replication and consistency model across sites
- [ ] DR testing cadence defined
- [ ] Runbook for failover and failback

### Failure Modes

- [ ] Failure modes and effects analysis (FMEA) or equivalent
- [ ] Blast radius for each component failure
- [ ] Cascading failure paths identified
- [ ] Human error scenarios — misconfiguration, bad deploy, wrong delete
- [ ] Dependency failure scenarios — third-party API, AWS regional issue
- [ ] Data corruption and recovery paths
- [ ] Communication plan during major incident

### Trade-offs

- [ ] Alternatives considered — at least two viable options
- [ ] Explicit list of what was gained and what was given up
- [ ] Principles deprioritised and why (reliability vs cost, speed vs security)
- [ ] Reversibility — how hard is it to change this decision later?
- [ ] Technical debt accepted — documented with trigger to revisit

### Future Improvements

- [ ] Known limitations of current design
- [ ] Triggers for architectural revisit (scale, compliance, new region)
- [ ] Backlog items from this review
- [ ] Migration or evolution path — v1 to v2
- [ ] Deprecation plan for interim solutions

---

## Review Output Template

Use this summary when closing a review:

```markdown
## Architecture Review Summary

**System:** [Name]
**Date:** [YYYY-MM-DD]
**Reviewers:** [Names]
**Outcome:** Approved | Approved with conditions | Not approved

### P0 — Launch Blockers
- [Item requiring resolution before production]

### P1 — Significant Risks
- [Risk] — [Mitigation or acceptance plan] — [Owner] — [Due]

### P2 — Improvement Opportunities
- [ ] [Item] — [Owner] — [Backlog ref or revisit trigger]

### Accepted Risks
- [P0/P1 risk accepted] — [Justification] — [Owner] — [Sign-off if P0]

### Action Items
- [ ] [Action] — [Owner] — [Due]

### ADRs Required
- [Decision to record]
```

---

## Related Resources

- [Engineering Principles](../engineering-principles/README.md)
- [Architecture Tiers](../architecture-tiers.md)
- [Organisational Constraints](../organisational-constraints.md)
- [Principle Conflicts](../principle-conflicts.md)
- [ADR Template](../architecture-decision-records/adr-template.md)
- [Documentation Standards](../standards/documentation-standards.md)
