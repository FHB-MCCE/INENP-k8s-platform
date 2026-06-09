# AI Usage

This project used AI assistance as a supporting tool during planning, implementation and documentation. The team treated AI output as a draft or review aid, not as an authoritative source.

## Supported Activities

- Compared the assignment pillars with the issue board to identify uneven ownership and rebalance work across team members.
- Drafted initial Terraform, Kubernetes and GitOps snippets that were then reviewed, adjusted and validated by the responsible team member.
- Helped structure PR descriptions, issue checklists and documentation sections so that project progress remains traceable.
- Suggested verification steps for Terraform, kubectl and Argo CD checks after infrastructure and platform changes.
- Helped review wording for architecture, cost and operations documentation.

## Human Review

All relevant implementation decisions stayed with the team:

- platform architecture and technology choices
- GCP project, region, domain and cost trade-offs
- Terraform apply approval and cluster changes
- pull request review and merge decisions
- final interpretation of test and verification results

AI-generated suggestions were checked against repository conventions, official tool behavior and actual command output before being accepted.

## Boundaries

- Secrets and private credentials are not documented in generated project files.
- AI output is not used as a substitute for pull request review.
- Generated configuration is validated through the same Terraform, Kubernetes and GitOps checks as hand-written changes.
- Final submission material is reviewed by the team before delivery.
