# References

Curated external resources that inform the thinking in this playbook. These are starting points — not endorsements of every opinion in every link. Prefer official documentation and primary sources where possible.

This list will grow as case studies and ADRs cite specific standards, papers, and guides.

---

## AWS Official

| Resource | Use for |
|----------|---------|
| [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) | Six pillars — operational excellence, security, reliability, performance, cost, sustainability |
| [AWS Architecture Center](https://aws.amazon.com/architecture/) | Reference architectures and best practices |
| [AWS Security Documentation](https://docs.aws.amazon.com/security/) | Shared responsibility, IAM, encryption, compliance |
| [AWS Pricing Calculator](https://calculator.aws/) | Cost estimation for ADRs and case studies |
| [AWS Service Quotas](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html) | Scale limits and quota increase planning |

---

## Reliability and Operations

| Resource | Use for |
|----------|---------|
| [Google SRE Book](https://sre.google/sre-book/table-of-contents/) | SLOs, error budgets, toil, incident response |
| [Site Reliability Workbook](https://sre.google/workbook/table-of-contents/) | Practical SRE implementation |
| [AWS Builders' Library](https://aws.amazon.com/builders-library/) | How Amazon teams build and operate systems |

---

## Security

| Resource | Use for |
|----------|---------|
| [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services) | Baseline security configuration |
| [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) | Identity and access design |
| [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Application security risks |

---

## Architecture Decision Records

| Resource | Use for |
|----------|---------|
| [Michael Nygard — Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) | Original ADR concept |
| [ADR GitHub Organisation](https://adr.github.io/) | Templates and tooling patterns |

---

## Networking (AWS)

| Resource | Use for |
|----------|---------|
| [VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/) | VPC, subnets, routing, endpoints |
| [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/) | Private service connectivity |

---

## Cost and FinOps

| Resource | Use for |
|----------|---------|
| [AWS Cost Management](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-cost-management.html) | Budgets, Cost Explorer, anomalies |
| [FinOps Foundation](https://www.finops.org/) | Organisational cost optimisation practices |

---

## How to Use This Directory

- **When writing an ADR or case study**, add citations here if the resource is reused across multiple documents
- **Prefer linking once** in references and referencing from other docs — avoid duplicating long URL lists
- **Note the date accessed** for fast-moving docs if the specific version mattered for a decision

---

## Contributing References

When adding a reference:

1. Confirm it is publicly accessible and likely to remain stable
2. Add one line on **what it is used for** in this playbook
3. Group under the appropriate category or propose a new category if needed

Do not add generic "top 10 cloud tips" listicles. Quality over quantity.
