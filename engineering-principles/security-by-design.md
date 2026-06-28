# Security by Design

## What This Principle Means

Security by design means security requirements are embedded in architecture from the first sketch — not bolted on before launch or after an audit finding. It treats security as a **property of the system**, shaped by identity, network boundaries, data classification, access patterns, and threat model — not as a checklist of controls applied at the end.

This principle assumes **zero trust at the boundary**: every request, every connection, and every identity is verified and authorised according to least privilege. Trust is earned per transaction, not inherited from network location.

---

## Why It Matters in Cloud Architecture

Cloud environments are dynamic, shared-responsibility, and exposed by default. A misconfigured S3 bucket, an overly permissive IAM role, or a security group allowing `0.0.0.0/0` can expose data at scale before anyone notices.

Security failures are asymmetric: attackers need one gap; defenders need consistent coverage. A single architectural mistake — public database, long-lived root credentials, unencrypted data at rest — can invalidate every other control.

Security by design also reduces rework. Retrofitting encryption, segmentation, or fine-grained IAM into a live system is expensive, risky, and often incomplete.

---

## Design Questions to Ask

- What is the **data classification** of information this system handles? Where does sensitive data live, move, and rest?
- Who are the **actors** (users, services, admins, third parties), and what can each one do?
- What is the **threat model** — what are we protecting against, and what is out of scope?
- How is **identity** established and verified for humans and machines?
- What is the **least-privilege IAM** model? Can we scope permissions to specific resources and actions?
- How are **secrets** stored, rotated, and accessed? Are they ever in code, env vars in plain text, or logs?
- What are the **network boundaries** — VPC, subnets, security groups, NACLs, PrivateLink, WAF?
- Is traffic **encrypted in transit** everywhere it crosses a trust boundary? TLS version and cipher policy?
- Is data **encrypted at rest**? Who holds the keys (CMK, AWS-managed, customer-managed)?
- How do we **audit** access and changes — CloudTrail, Config, VPC Flow Logs, application audit logs?
- What is the **blast radius** of a compromised credential or instance?
- How do we **patch and update** — AMIs, containers, dependencies, managed services?
- What **compliance frameworks** apply (SOC 2, PCI, HIPAA, GDPR), and what do they require architecturally?

---

## Common Mistakes to Avoid

| Mistake | Why it hurts |
|---------|--------------|
| Security groups open to the world "temporarily" | Temporary becomes permanent; scanners find it quickly |
| One IAM role with `*` permissions for all services | Compromise of one workload compromises everything |
| Long-lived access keys instead of IAM roles | Keys leak; rotation is rarely enforced |
| Trusting "private subnet" as sufficient security | Lateral movement inside a VPC is common after initial breach |
| Logging disabled or retained for too short a period | Incidents cannot be investigated; compliance fails |
| Secrets in Terraform state, CI logs, or environment variables in plain text | State and logs are attack surfaces |
| No separation between environments in IAM or network | Dev credentials or misconfigurations reach production data |
| Assuming managed services are "secure by default" | Default configs often allow public access or weak encryption |
| Ignoring supply chain — unvetted container images, dependencies | Attack vector grows with every third-party layer |
| Security as a gate at the end of the project | Findings require architectural rework under deadline pressure |

---

## In Practice

Security by design does not mean maximum security everywhere. It means **appropriate security for the data and risk** — documented, justified, and testable.

A public marketing site and a payment processing API require different models. The goal is to match controls to classification and threat, not to apply the same lock to every door.

Document security decisions in ADRs: why this boundary, why this IAM shape, what was accepted as residual risk.
