
[workload-identity-federation.md](https://github.com/user-attachments/files/26558830/workload-identity-federation.md)
# **Workload Identity Federation Guide**

> Passwordless authentication pattern for CI/CD pipelines using OpenID Connect (OIDC). Demonstrated using `carrieretail-cicd-deployer` federated with GitHub Actions.

---

## **Overview**

Workload Identity Federation allows external workloads — such as GitHub Actions pipelines — to authenticate to Microsoft Entra ID without storing any credentials. Instead of a client secret, GitHub issues a short-lived OIDC token at runtime that Entra ID trusts based on a pre-configured federation.

| Attribute | Detail |
|---|---|
| **Reference NHI** | carrieretail-cicd-deployer |
| **Federation Type** | GitHub Actions → Microsoft Entra ID |
| **Credential Stored** | None |
| **Token Lifetime** | Short-lived, issued at runtime by GitHub |
| **Tenant** | Carrie Cares (CarrieCares.onmicrosoft.com) |

---

## **The Problem This Solves**

### **Legacy Pattern — Client Secret in GitHub Secrets**

Most SMB teams default to this when setting up CI/CD pipelines to Azure:

```
1. Create app registration in Entra ID
2. Generate client secret
3. Store secret in GitHub → Settings → Secrets
4. Reference secret in GitHub Actions workflow
```

| Risk | Impact |
|---|---|
| Secret stored at rest in GitHub | Exposed if repository is compromised |
| Secret has long expiry (often 12-24 months) | Extended window of exposure |
| Manual rotation required | Rotation is skipped, secret goes stale |
| No audit trail on secret usage | Cannot detect misuse without additional tooling |
| Secret leaked via logs or misconfiguration | Full application access compromised |

### **Modern Pattern — Workload Identity Federation**

```
1. Create app registration in Entra ID
2. Configure federated credential pointing to GitHub repo
3. GitHub Actions requests OIDC token at runtime
4. Entra ID validates token against federation trust
5. Pipeline receives short-lived access token — no secret ever stored
```

| Benefit | Impact |
|---|---|
| No stored credential | Nothing to leak, rotate, or expire |
| Token scoped to specific repo and branch | Blast radius minimized if token is intercepted |
| Token expires in minutes | Unusable outside the pipeline run |
| Every token issuance logged in Entra sign-in logs | Full audit trail automatically |

---

## **Before vs After Comparison**

| Dimension | Legacy (Secret) | Modern (Federation) |
|---|---|---|
| Credential stored | Yes — GitHub Secrets | No — none exists |
| Rotation required | Yes — manual, often skipped | No — not applicable |
| Expiry risk | Yes — if not rotated | No — token auto-expires |
| Audit trail | None by default | Every issuance logged in Entra |
| Blast radius if compromised | Full app access | Single pipeline run only |
| Setup complexity | Low | Low — one-time configuration |

---

## **Implementation — Step by Step**

### **Step 1: Create the App Registration**

```
Entra ID → App registrations → + New registration
→ Name: carrieretail-cicd-deployer
→ Supported account types: Single tenant
→ Redirect URI: Leave blank
→ Register
```

**Record the following from the Overview page:**

| Value | Where to find it |
|---|---|
| Application (Client) ID | Overview page |
| Directory (Tenant) ID | Overview page |

### **Step 2: Configure the Federated Credential**

```
Certificates & secrets → Federated credentials tab → + Add credential
→ Scenario: GitHub Actions deploying Azure resources
```

| Field | Value |
|---|---|
| **Organization** | billycarrie (GitHub username) |
| **Repository** | nhi-lifecycle-mgmt |
| **Entity type** | Branch |
| **Branch name** | main |
| **Credential name** | github-main-branch-federation |

```
→ Add
```

> The Subject Identifier generated will be: `repo:carrieretailco/nhi-lifecycle-mgmt:ref:refs/heads/main`
> This means only the main branch of this specific repo can use this federation.

### **Step 3: Assign Azure RBAC (when subscription is available)**

Federation handles **authentication** — proving who the pipeline is.
Azure RBAC handles **authorization** — what the pipeline can do.

```
Azure Portal → Subscription or Resource Group
→ Access Control (IAM) → + Add role assignment
→ Role: Contributor (or least privilege role needed)
→ Assign access to: User, group, or service principal
→ Select: carrieretail-cicd-deployer
→ Review + assign
```

> ⚠️ Do not grant Contributor at subscription scope unless required. Scope to the specific resource group the pipeline deploys to.

### **Step 4: Configure GitHub Actions Workflow**

```yaml
name: Deploy to Azure

on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login (OIDC — no secret)
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

> Note: `AZURE_CLIENT_ID` and `AZURE_TENANT_ID` are not secrets in the security sense — they are non-sensitive identifiers. They are stored in GitHub Secrets for convenience, not security.

---

## **Governance Considerations**

| Topic | Guidance |
|---|---|
| **Scope federation tightly** | Use branch-level federation (main only), not wildcard or environment-level, to minimize who can authenticate |
| **One federation per environment** | Separate federated credentials for dev, staging, and production branches |
| **Review federation trusts quarterly** | Ensure the repo and branch still exist and are actively maintained |
| **Decommission with the pipeline** | When a repo is archived or a pipeline retired, delete the federated credential |
| **Monitor sign-in logs** | Entra sign-in logs capture every OIDC token issuance — alert on unexpected sources |

---

## **Reference Implementation**

**carrieretail-cicd-deployer** was configured following this guide:

- App registration created — single tenant, no redirect URI
- Federated credential configured — GitHub Actions, main branch
- Subject identifier scoped to `nhi-lifecycle-mgmt` repo
- No client secret created — zero stored credentials
- Owner assigned (Billy Carrie)
- Recorded in NHI inventory

---

*Last updated: April 2026 — Billy Carrie*
