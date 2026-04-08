NHI Lifecycle Management

Non-Human Identity governance framework built and documented in a live Microsoft Entra ID sandbox — Carrie Cares tenant (SMB simulation).

What This Repo Is
Most IAM documentation describes how NHI governance should work. This repo documents how it actually works — built hands-on in a live Entra ID P2 tenant simulating a small-to-mid-size retail/ecommerce company undergoing identity program maturation.
Every configuration, runbook, and governance record here reflects real objects created in a sandbox environment, not theoretical examples.

The SMB Context
Organization: Carrie Cares (CarrieCares tenant)
Industry: Retail / Ecommerce
Size: ~200 employees, Azure-first
IAM maturity: Building from the ground up
Challenge: Inconsistent NHI governance — shared accounts, unowned app registrations, secrets with 24-month expiry, no lifecycle process
This mirrors the reality of organizations encountered in M&A security work — acquired companies with identity debt and no established NHI program.

NHI Types Covered
NHI TypeApp NameCredential PatternUse CaseService Principalretailsync-pro-connectorClient secret — 6mo rotationThird-party inventory sync integrationWorkload Identity Federationcarrieretail-cicd-deployerOIDC — no stored secretGitHub Actions → Azure deployment pipelineAI Agent Identityai-agent-customer-support-copilotClient secret — 3mo rotationCopilot-powered customer support pilot

Repo Structure
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
    └── Get-NHIInventory.ps1           # PowerShell — discover and report all NHIs (coming soon)

Core Governance Principles
1. Every NHI must have an owner
No app registration exists without a named human owner accountable for its lifecycle. Ownerless apps are the #1 NHI audit finding in SMB environments.
2. Credential type follows use case

Azure-native workloads → managed identity (no credential)
CI/CD pipelines → workload identity federation (no stored secret)
Third-party integrations → client secret with enforced short expiry
Legacy on-prem → certificate-based auth

3. Least privilege by default
Every NHI is granted the minimum permission required. Permission decisions are documented with explicit rationale — not just what was granted, but what was considered and denied.
4. Lifecycle is not optional
Every NHI has a defined onboarding process, review cadence, rotation schedule, and decommission trigger. If any of these are undefined, the NHI is ungoverned by definition.
5. AI agents are NHIs
AI agents, Copilot extensions, and LLM-backed services require the same identity governance as any other non-human principal — ownership, scoped access, audit trail, and lifecycle accountability.

Key Governance Concepts Demonstrated
ConceptWhere DocumentedService principal vs managed identity tradeoffservice-principal-lifecycle.mdDelegated vs Application permission typesservice-principal-lifecycle.mdPasswordless auth via OIDC federationworkload-identity-federation.mdLegacy (secret in GitHub) vs modern (OIDC) patternworkload-identity-federation.mdAI agent as distinct NHI subtypeai-agent-governance.mdPermission decision documentationai-agent-governance.mdNHI inventory as audit evidencenhi-inventory.md

Author
Billy Carrie — IAM Engineer
