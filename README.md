[nhi-README.md](https://github.com/user-attachments/files/26558475/nhi-README.md)
# **NHI Lifecycle Management**

> Non-Human Identity governance framework built and documented in a live Microsoft Entra ID sandbox — Carrie Cares tenant.

---

## **What This Repo Is**

This repo documents an Entra ID tenant of a small-to-mid-size retail/ecommerce company undergoing identity program maturation.

Every configuration, runbook, and governance record here reflects real objects created in a sandbox environment.

---

## **The SMB Context**

| Field | Detail |
|---|---|
| **Organization** | Carrie Cares (CarrieCares tenant) |
| **Industry** | Retail / Ecommerce |
| **Size** | ~200 employees, Azure-first |
| **IAM Maturity** | Building from the ground up |
| **Challenge** | Shared accounts, unowned app registrations, secrets with 24-month expiry, no lifecycle process |

This mirrors the reality of organizations encountered in M&A security work — acquired companies with identity debt and no established NHI program.

---

## **NHI Types Covered**

| # | NHI Type | App Name | Credential Pattern | Use Case |
|---|---|---|---|---|
| 1 | **Service Principal** | `retailsync-pro-connector` | Client secret — 6mo rotation | Third-party inventory sync integration |
| 2 | **Workload Identity Federation** | `carrieretail-cicd-deployer` | OIDC — no stored secret | GitHub Actions → Azure deployment pipeline |
| 3 | **AI Agent Identity** | `ai-agent-customer-support-copilot` | Client secret — 3mo rotation | Copilot-powered customer support pilot |

---

## **Repo Structure**

```
nhi-lifecycle-mgmt/
│
├── README.md                          # This file — overview and context
│
├── docs/
│   ├── nhi-inventory.md               # Live inventory of all NHIs in the tenant
│   ├── service-principal-lifecycle.md # Runbook — SP onboarding to decommission
│   ├── workload-identity-federation.md# Guide — passwordless CI/CD auth pattern
│   └── ai-agent-governance.md         # Governance record — AI agent NHI subtype
│
└── scripts/
    └── Get-NHIInventory.ps1 (coming soon)           # PowerShell — discover and report all NHIs 
```

---

## **Core Governance Principles**

**1. Every NHI must have an owner**
No app registration exists without a named human owner accountable for its lifecycle. Ownerless apps are one of tthe top NHI audit findings in SMB environments.

**2. Credential type follows use case**
- Azure-native workloads → managed identity (no credential)
- CI/CD pipelines → workload identity federation (no stored secret)
- Third-party integrations → client secret with enforced short expiry
- Legacy on-prem → certificate-based auth

**3. Least privilege by default**
Every NHI is granted the minimum permission required. Permission decisions are documented with explicit rationale — not just what was granted, but what was considered and denied.

**4. Lifecycle is not optional**
Every NHI has a defined onboarding process, review cadence, rotation schedule, and decommission trigger. If any of these are undefined, the NHI is ungoverned by definition.

**5. AI agents are NHIs**
AI agents, Copilot extensions, and LLM-backed services require the same identity governance as any other non-human principal — ownership, scoped access, audit trail, and lifecycle accountability.

---

## **Key Governance Concepts Demonstrated**

| Concept | Where Documented |
|---|---|
| Service principal vs managed identity tradeoff | `service-principal-lifecycle.md` |
| Delegated vs Application permission types | `service-principal-lifecycle.md` |
| Passwordless auth via OIDC federation | `workload-identity-federation.md` |
| Legacy (secret in GitHub) vs modern (OIDC) pattern | `workload-identity-federation.md` |
| AI agent as distinct NHI subtype | `ai-agent-governance.md` |
| Permission decision documentation | `ai-agent-governance.md` |
| NHI inventory as audit evidence | `nhi-inventory.md` |

---

## **Author**

**Billy Carrie** — IAM Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/billycarrie/)
