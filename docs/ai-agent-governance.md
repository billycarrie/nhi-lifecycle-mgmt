
[ai-agent-governance.md](https://github.com/user-attachments/files/26558612/ai-agent-governance.md)
# **AI Agent Identity Governance**

> Governance framework and lifecycle record for AI agent identities in the Carrie Cares Entra ID tenant. Demonstrated using `ai-agent-customer-support-copilot` as the reference implementation.

---

## **Why AI Agents Require Distinct NHI Governance**

AI agents — including Copilot extensions, LLM-backed services, and autonomous workflow bots — are Non-Human Identities. They authenticate, access data, and take actions on behalf of the organization. Most organizations govern them as if they were standard service principals. They are not.

| Dimension | Standard Service Principal | AI Agent Identity |
|---|---|---|
| **Behavior** | Deterministic — does exactly what it is programmed to do | Probabilistic — behavior influenced by model outputs |
| **Data access risk** | Scoped to API permissions | May surface, summarize, or relay data in unexpected ways |
| **Audit requirement** | Standard sign-in logs | Requires prompt/response logging in addition to identity logs |
| **Ownership** | Technical owner (developer/engineer) | Requires both technical and business owner |
| **Decommission trigger** | Service end | Model replacement, behavior change, or policy violation |
| **Review cadence** | Annually or bi-annually | Quarterly minimum — model behavior can drift |

---

## **AI Agent Identity Record**

### **ai-agent-customer-support-copilot**

| Field | Detail |
|---|---|
| **Display Name** | ai-agent-customer-support-copilot |
| **NHI Type** | AI Agent Identity |
| **NHI Subtype** | Copilot Extension — Customer-Facing |
| **Application (Client) ID** | 14520851-b3c0-41cd-8c40-fc70b1632e0a |
| **Object ID** | 83e641c2-4e4c-4adc-b515-7fa58f226c4e |
| **Tenant ID** | 816170cf-fac8-440b-b561-fa4706a4db1c |
| **Credential Type** | Client Secret |
| **Secret Description** | ai-agent-copilot-secret-q2-2026 |
| **Secret Expiry** | 7/7/2026 |
| **Rotation Due** | 6/7/2026 (30 days before expiry) |
| **Technical Owner** | Billy Carrie — IAM Engineer |
| **Business Owner** | Customer Support Lead |
| **Environment** | Pilot / Non-Production |
| **Purpose** | Answer customer order status queries via Copilot interface |
| **Data Access** | User profile read (delegated) — no order data direct access |
| **Risk Level** | Medium — customer-facing, delegated access |
| **Review Cadence** | Quarterly |
| **Decommission Trigger** | Pilot end date, model replacement, or policy violation |
| **Status** | ✅ Active — Pilot |

---

## **Permission Decision Record**

All permissions requested for this AI agent were evaluated against the principle of least privilege. The following table documents both approved and denied permissions with explicit rationale.

| Permission | Type | Decision | Rationale |
|---|---|---|---|
| **User.Read** | Delegated | ✅ Approved | Minimum required — agent greets user by name using signed-in profile |
| **User.ReadAll** | Delegated | ❌ Denied | Agent does not need access to all user profiles — only the signed-in user |
| **Mail.Read** | Delegated | ❌ Denied | No email functionality required for order status queries |
| **Calendars.Read** | Delegated | ❌ Denied | Out of scope for customer support use case |
| **Directory.Read.All** | Application | ❌ Denied | Organizational directory access not required — excessive for this agent |

> This permission decision record serves as audit evidence demonstrating that least privilege was actively enforced, not assumed.

---

## **AI Agent Governance Framework**

### **Classification Criteria**

An identity should be classified as an AI Agent NHI if it meets any of the following:

- Backed by a large language model (LLM) or generative AI service
- Behavior is probabilistic rather than fully deterministic
- Interacts with end users directly (customer-facing or employee-facing)
- Accesses or surfaces organizational data via AI-driven responses
- Operates as part of an autonomous or semi-autonomous workflow

### **Onboarding Requirements**

| Requirement | Standard NHI | AI Agent NHI |
|---|---|---|
| Named technical owner | ✅ Required | ✅ Required |
| Named business owner | Optional | ✅ Required |
| Permission decision record | Recommended | ✅ Required |
| Data access classification | Recommended | ✅ Required |
| Prompt/response logging plan | Not applicable | ✅ Required |
| Quarterly review cadence | Optional | ✅ Required |
| Decommission trigger documented | Recommended | ✅ Required |

### **Ongoing Review Checklist**

| Check | Frequency | Action if Failed |
|---|---|---|
| Secret expiry within 60 days | Monthly | Rotate immediately |
| Business owner still active | Quarterly | Reassign or decommission |
| Permissions still reflect minimum required | Quarterly | Remove excess permissions |
| Agent behavior aligned with documented purpose | Quarterly | Escalate to business owner |
| No new permissions added without approval | Quarterly | Review audit log, escalate if unauthorized |
| Usage volume within expected parameters | Quarterly | Investigate anomalous spikes |

### **Decommission Process**

```
Step 1 — Confirm with business owner that agent is no longer needed
Step 2 — Disable the app registration (Deactivate — do not delete immediately)
Step 3 — Monitor for 7 days — confirm no authentication failures reported
Step 4 — Revoke all client secrets
Step 5 — Remove all API permissions
Step 6 — Delete the app registration
Step 7 — Update NHI inventory — mark status as Decommissioned
Step 8 — Document decommission date, reason, and approver
```

---

## **AI Agent Risk Considerations**

| Risk | Description | Mitigation |
|---|---|---|
| **Data oversharing** | Agent surfaces data beyond what the user should see | Scope permissions to minimum, implement output filtering |
| **Prompt injection** | Malicious user input manipulates agent behavior | Input validation, system prompt hardening |
| **Credential exposure** | Client secret leaked via logs or misconfiguration | Short expiry (3 months), Key Vault storage |
| **Scope creep** | Additional permissions added without review | Permission decision record, quarterly review |
| **Shadow AI agents** | Teams deploy AI agents without IAM involvement | Discovery scanning, onboarding policy enforcement |
| **Model drift** | Agent behavior changes as underlying model updates | Quarterly behavior review with business owner |

---

## **Future State — Recommended Improvements**

| Improvement | Priority | Dependency |
|---|---|---|
| Migrate from client secret to certificate-based auth when pilot graduates to production | High | None |
| Implement Azure Key Vault for secret storage | High | Azure subscription |
| Add automated secret rotation | Medium | Azure subscription + Key Vault |
| Enable Microsoft Purview audit logging for agent interactions | Medium | Purview license |
| Deploy identity-based Conditional Access policy scoped to AI agent sign-ins | Low | Entra P2 (already available) |

---

## **Reference Implementation**

**ai-agent-customer-support-copilot** was onboarded following this framework:

- ✅ Classified as AI Agent NHI subtype
- ✅ Technical and business owner documented
- ✅ Permission decision record completed — 4 permissions denied, 1 approved
- ✅ Client secret — 3 month expiry (shorter than standard reflecting higher risk)
- ✅ Quarterly review cadence set
- ✅ Decommission trigger documented
- ✅ Recorded in NHI inventory

---

*Last updated: April 2026 — Billy Carrie*
