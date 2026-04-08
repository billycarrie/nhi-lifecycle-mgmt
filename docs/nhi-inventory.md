
[nhi-inventory.md](https://github.com/user-attachments/files/26558647/nhi-inventory.md)
# **NHI Inventory — Carrie Cares Tenant**

> Live inventory of all Non-Human Identities registered in the Carrie Cares Entra ID tenant. This document serves as the authoritative source of record for NHI governance, audit evidence, and lifecycle tracking.

---

## **Inventory Summary**

| Field | Detail |
|---|---|
| **Tenant** | Carrie Cares (CarrieCares.onmicrosoft.com) |
| **Total NHIs** | 3 |
| **Last Reviewed** | April 2026 |
| **Reviewed By** | Billy Carrie — IAM Engineer |

---

## **NHI Registry**

### **1. retailsync-pro-connector**

| Field | Detail |
|---|---|
| **Display Name** | retailsync-pro-connector |
| **NHI Type** | Service Principal |
| **Application (Client) ID** | a855fd3a-fafc-4084-9348-32b5d4719ab2 |
| **Credential Type** | Client Secret |
| **Rotation Due** | 30 days before expiry |
| **API Permissions** | User.Read.All (Application) — admin consented |
| **Owner** | Billy Carrie |
| **Environment** | Production |
| **Purpose** | Third-party inventory sync integration (RetailSync Pro) |
| **Risk Level** | Medium — application permission, broad user read access |
| **Review Cadence** | Quarterly |
| **Decommission Trigger** | RetailSync Pro contract end or replacement |
| **Status** | ✅ Active |

---

### **2. carrieretail-cicd-deployer**

| Field | Detail |
|---|---|
| **Display Name** | carrieretail-cicd-deployer |
| **NHI Type** | Workload Identity Federation |
| **Credential Type** | Federated — OIDC (no stored secret) |
| **Federated Credential Name** | github-main-branch-federation |
| **Rotation Required** | No — OIDC tokens are short-lived and issued at runtime |
| **API Permissions** | User.Read (Delegated) — Microsoft default |
| **Owner** | Billy Carrie |
| **Environment** | Production / CI-CD |
| **Purpose** | GitHub Actions pipeline authentication to Azure — passwordless deployment |
| **Risk Level** | Low — no stored credential, scoped to specific repo and branch |
| **Review Cadence** | Bi-annually |
| **Decommission Trigger** | Pipeline retirement or repo archive |
| **Status** | ✅ Active |

---

### **3. ai-agent-customer-support-copilot**

| Field | Detail |
|---|---|
| **Display Name** | ai-agent-customer-support-copilot |
| **NHI Type** | AI Agent Identity |
| **Application (Client) ID** | 14520851-b3c0-41cd-8c40-fc70b1632e0a |
| **Credential Type** | Client Secret |
| **Rotation Due** | 6/7/2026 (30 days before expiry) |
| **API Permissions** | User.Read (Delegated) — sign in and read user profile |
| **Owner** | Billy Carrie |
| **Business Owner** | Customer Support Lead |
| **Environment** | Pilot / Non-Production |
| **Purpose** | Copilot-powered customer support agent — order status queries |
| **Risk Level** | Medium — customer-facing, delegated user access |
| **Review Cadence** | Quarterly |
| **Decommission Trigger** | Pilot end date or model replacement |
| **Status** | ✅ Active — Pilot |

---

## **Governance Gaps & Findings**

| Finding | Severity | Affected NHI | Remediation |
|---|---|---|---|
| No managed identity option available (no Azure subscription) | Low | All | Add Azure subscription to enable managed identity for workloads |
| Client secret used for AI agent (vs certificate) | Low | ai-agent-customer-support-copilot | Migrate to certificate auth when pilot graduates to production |
| Secret rotation is manual | Medium | retailsync-pro-connector, ai-agent-customer-support-copilot | Implement automated rotation via Key Vault when Azure subscription available |

---

## **Rotation Schedule**

| NHI | Credential | Expiry | Rotation Due | Owner |
|---|---|---|---|---|
| retailsync-pro-connector | Client secret | 6mo from creation | 30 days before expiry | Billy Carrie |
| carrieretail-cicd-deployer | OIDC federated | No expiry | N/A | Billy Carrie |
| ai-agent-customer-support-copilot | Client secret | 7/7/2026 | 6/7/2026 | Billy Carrie |

---

## **Audit Evidence**

This inventory document, combined with portal screenshots in the `/docs` folder, constitutes the NHI audit evidence package for the Carrie Cares tenant. It demonstrates:

- Named ownership for all NHIs
- Documented credential types and expiry dates
- Explicit permission grants with admin consent
- Risk classification per NHI
- Defined decommission triggers

---

*Last updated: April 2026 — Billy Carrie*
