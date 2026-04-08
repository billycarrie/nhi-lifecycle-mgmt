
[service-principal-lifecycle.md](https://github.com/user-attachments/files/26558704/service-principal-lifecycle.md)
# **Service Principal Lifecycle Runbook**

> Operational runbook for the creation, governance, and decommissioning of service principals in the Carrie Cares Entra ID tenant. Demonstrated using `retailsync-pro-connector` as the reference implementation.

---

## **Overview**

A service principal is the identity object created in an Entra ID tenant when an application is registered. Unlike managed identities, service principals require active credential management — making lifecycle governance critical.

| Attribute | Detail |
|---|---|
| **Reference NHI** | retailsync-pro-connector |
| **Credential Type** | Client Secret |
| **Tenant** | Carrie Cares (CarrieCares.onmicrosoft.com) |
| **Risk Without Governance** | Orphaned apps, expired or leaked secrets, over-permissioned access |

---

## **Phase 1 — Onboarding**

### **Step 1: Business Justification**

Before creating any service principal, the following must be documented:

| Question | Required Answer |
|---|---|
| What is the business purpose? | e.g. Third-party inventory sync via RetailSync Pro |
| Who is the human owner? | Named individual, not a team or shared mailbox |
| What is the minimum permission required? | Documented with rationale |
| What is the expected lifespan? | Tied to contract, project, or service lifecycle |
| What environment will this run in? | Production / Non-production / Pilot |

### **Step 2: App Registration**

```
Microsoft Entra Admin Center
→ Entra ID → App registrations → + New registration
→ Name: [descriptive-lowercase-hyphenated]
→ Supported account types: Single tenant (My organization only)
→ Redirect URI: Leave blank for daemon/service apps
→ Register
```

**Naming convention:**
```
[org]-[service]-[function]
Example: carrieretail-retailsync-connector
```

### **Step 3: Credential Configuration**

**Client secret (when managed identity or federation is not possible):**

```
Certificates & secrets → Client secrets → + New client secret
→ Description: [app-name]-[env]-secret-[quarter-year]
→ Expiry: 6 months maximum (never use 24 months default)
→ Add → Copy value immediately
```

**Permission type decision:**

| Scenario | Permission Type | Reason |
|---|---|---|
| App runs as itself, no user context | Application | Daemon/service flow — client credentials grant |
| App acts on behalf of a signed-in user | Delegated | Authorization code or implicit flow |
| CI/CD pipeline | Neither — use federation | No credential needed |

### **Step 4: API Permissions**

```
API permissions → + Add a permission → Microsoft Graph
→ Select permission type (Application or Delegated — see table above)
→ Search and add minimum required permission
→ Grant admin consent for [tenant]
```

**Document every permission decision:**

| Permission Requested | Type | Approved / Denied | Reason |
|---|---|---|---|
| User.Read.All | Application | Approved | Required for inventory sync to read all user profiles |
| Directory.Read.All | Application | Denied | Broader than required — not needed for this use case |

### **Step 5: Assign an Owner**

```
Owners → + Add owners → [named individual]
```

> ⚠️ **No service principal should exist without a named owner.** Ownerless apps are the most common NHI finding in SMB audits and the primary cause of ungoverned credential sprawl.

---

## **Phase 2 — Ongoing Governance**

### **Quarterly Review Checklist**

| Check | Action if Failed |
|---|---|
| Owner is still active in the organization | Reassign to new owner or decommission |
| Secret expiry is within 90 days | Initiate rotation immediately |
| Permissions still match minimum required | Remove excess permissions |
| App is still in active use | Verify with business owner or check sign-in logs |
| No new permissions have been added without approval | Review audit log, escalate if unauthorized |

### **Sign-in Log Review**

```
Entra ID → Monitoring → Sign-in logs
→ Filter: Application → [app name]
→ Review: Last sign-in date, IP, success/failure rate
```

> If an app has not authenticated in 90+ days, flag for decommission review.

### **Secret Rotation Process**

```
Certificates & secrets → + New client secret
→ Create new secret BEFORE deleting old one
→ Update secret in consuming application/pipeline
→ Verify application is authenticating with new secret
→ Delete old secret
→ Update NHI inventory with new expiry date
```

> ⚠️ Always create-then-delete, never delete-then-create. Deleting first can cause an authentication outage.

---

## **Phase 3 — Decommissioning**

### **Decommission Triggers**

- Contract or service end
- Application migration to managed identity or federation
- Business owner departure with no successor
- 90+ days of zero sign-in activity
- Security incident involving the credential

### **Decommission Steps**

```
Step 1 — Confirm with business owner that the app is no longer needed
Step 2 — Check sign-in logs for any recent authentication activity
Step 3 — Revoke / delete all client secrets
Step 4 — Remove all API permissions
Step 5 — Delete the app registration
Step 6 — Update NHI inventory — mark status as Decommissioned
Step 7 — Document decommission date and reason
```

> ⚠️ Deleting the app registration also deletes the service principal object in the tenant. Verify no downstream dependencies before deletion.

---

## **Common SMB Anti-Patterns**

| Anti-Pattern | Risk | Correct Approach |
|---|---|---|
| Using a shared user account instead of a service principal | Account tied to a person — leaves when they do | Register a dedicated app identity |
| Setting secret expiry to 24 months (default) | Long window for credential exposure if leaked | Set maximum 6 months, prefer 3 months |
| No owner assigned | Orphaned app — no accountability | Always assign a named owner at creation |
| Granting Directory.ReadWrite.All "just in case" | Massively over-permissioned | Start with minimum, add only what is proven necessary |
| Storing client secrets in code repositories | Secret exposure via git history | Use Key Vault references or environment variables |

---

## **Reference Implementation**

**retailsync-pro-connector** was created following this runbook:

- Business justification documented (RetailSync Pro integration)
- Single tenant, no redirect URI (daemon app)
- Client secret — 6 month expiry with descriptive name
- User.Read.All (Application) — admin consented, rationale documented
- Owner assigned (Billy Carrie)
- Recorded in NHI inventory

---

*Last updated: April 2026 — Billy Carrie*
