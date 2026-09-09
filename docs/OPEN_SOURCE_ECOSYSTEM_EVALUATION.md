# BUDE HOME CBE - Open-Source Ecosystem Evaluation

- **Status:** Production architecture decision support
- **Version:** 1.0
- **Prepared:** 2026-09-09
- **Scope:** Non-Zerodha and Zerodha open-source, open-core, and source-available projects
- **Related:** [Zerodha-Inspired Architecture](ZERODHA_INSPIRED_FOSS_ARCHITECTURE.md), [Production Plan](PRODUCTION_IMPLEMENTATION_PLAN.md), [Investor Plan](INVESTOR_BUSINESS_PLAN.md)

> This is a screening document, not a direction to install every project. The intended architecture remains a small BUDE transactional core with a few isolated, replaceable satellites.

---

## 1. Executive decision

The wider ecosystem review supports the existing core architecture and improves the satellite selection process.

**Keep as the production baseline:**

- FastAPI modular monolith for BUDE's unique PG domain.
- Managed PostgreSQL/PostGIS as the only authority for organizations, properties, beds, stays, invoices, payments, deposits, entitlements, complaints, and consent.
- Flutter for owner, staff, and tenant mobile experiences.
- Managed identity, object storage, edge, database, and delivery providers during the pilot.
- PostgreSQL outbox and workers before adding a message bus or workflow engine.

**Run competitive proofs instead of preselecting a brand:**

- Support: LibreDesk vs Chatwoot vs Zammad, with a managed service as the control.
- Internal BI: Metabase vs Apache Superset.
- Back office: Frappe/ERPNext first; compare Odoo Community only if a documented gap appears.
- Product analytics: PostHog vs Matomo after privacy, consent, and data-minimization design.
- Identity: managed provider first; Keycloak or ZITADEL only when SSO, residency, economics, or exit requirements justify self-operation.

**Do not introduce yet:**

- A second backend-as-a-service, generic PMS, workflow engine, search cluster, self-hosted object store, or subscription-billing platform.
- Kafka, Kubernetes, OpenSearch, Temporal, or ClickHouse without a measured trigger.
- Directus, n8n, Sentry, or HashiCorp Vault under an assumption that source-available means OSI open source. Their current terms must be reviewed for the exact intended use.
- MinIO Community as a new dependency. Its official repository is archived and states that it is no longer maintained.

The goal is not maximum reuse. The goal is the smallest secure system that avoids rebuilding commodity capabilities while keeping BUDE's strategic and high-integrity domain under BUDE control.

---

## 2. Evaluation method

### Hard gates

A project is disqualified from production if any answer is no:

1. Is the exact version and every required plugin licensed for the intended commercial use?
2. Can it run without direct write access to BUDE core tables?
3. Can PII, KYC, payment details, and cross-tenant data be excluded or minimized?
4. Can backup, clean restore, upgrade, rollback, export, and removal be demonstrated?
5. Is there an active security policy and a practical supported-version strategy?
6. Can two named people operate it and meet patch and incident SLAs?
7. Does customer-critical BUDE functionality continue when it is unavailable?

### Weighted score

Score each criterion from 0 to 5. A proof may proceed at 70/100. Production requires 80/100, no hard-gate failure, and no score below 3 for security, isolation, or recovery.

| Criterion | Weight | Evidence |
|---|---:|---|
| Functional fit | 15 | Required workflows completed without unsafe customization |
| Security and identity | 15 | Security policy, MFA/SSO/RBAC, audit, secure configuration, advisory history |
| Data isolation and privacy | 15 | Minimum data map, tenant tests, retention/deletion, encrypted transport/storage |
| Recovery and upgrade | 15 | Timed clean restore, upgrade, rollback, outage and capacity tests |
| License and commercial fit | 10 | Counsel-approved exact commit, edition, dependencies, plugins, modifications, exposure |
| Maintenance and governance | 10 | Release cadence, supported branches, maintainers, issue handling, bus factor |
| Integration and exit | 10 | Versioned API/events, export, idempotency, removal without core-data loss |
| Total cost of ownership | 10 | Infrastructure, paid features, engineering, on-call, patching and migration |

GitHub stars are discovery signals only. They do not score security, correctness, support, licensing, or suitability.

### License posture labels

- **Permissive:** typically MIT, BSD, Apache 2.0, PostgreSQL License, or MPL-style terms. Notices and dependency obligations still apply.
- **Copyleft:** GPL or AGPL family. Legal review, modification tracking, notices, and source-offer obligations may apply.
- **Open-core/mixed:** the core and enterprise folders/features have different terms.
- **Source-available:** code can be inspected but use is restricted; do not call it open source without confirming OSI status.
- **Verify:** the exact release, image, plugin, and dependency set must be checked before proof or production.

This classification is engineering triage, not legal advice.

---

## 3. Candidate landscape - 53 projects

### 3.1 Application platform and content

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [FastAPI](https://github.com/fastapi/fastapi) | Python API framework | Permissive/MIT | **Adopt baseline** | Explicit domain/API design and fast delivery; keep business logic framework-light |
| [Appwrite](https://github.com/appwrite/appwrite) | Self-hostable BaaS | Permissive core; verify edition | **Defer** | Duplicates auth, database, storage, messaging and functions already covered; its own docs say self-hosting means managing updates and scaling |
| [PocketBase](https://github.com/pocketbase/pocketbase) | Embedded backend for small applications | Permissive/MIT | **Reject for core; allow prototypes** | Single-node simplicity is useful for tools, not high-integrity multi-tenant payments and bookings |
| [Hasura GraphQL Engine](https://github.com/hasura/graphql-engine) | GraphQL over databases | Open-core; verify current edition | **Defer** | Adds another authorization and API surface; BUDE needs deliberate domain commands, not unrestricted data CRUD |
| [Strapi](https://github.com/strapi/strapi) | Headless CMS | Permissive core/open-core | **Candidate for public content only** | Useful for articles, policies, city guides and SEO content; never for live inventory or prices |
| [Directus](https://directus.com/license) | Data studio/headless CMS | Source-available MSCL with future GPL conversion | **Defer pending legal review** | Current license is not a simple permissive FOSS grant; it also risks exposing core tables as generic data APIs |

Decision: keep FastAPI. If marketing content becomes an engineering bottleneck, compare Strapi with a managed CMS using a separate content database.

### 3.2 Identity and access

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [Keycloak](https://github.com/keycloak/keycloak) | OIDC/SAML identity and SSO | Apache 2.0 | **Trigger-based finalist** | Mature and capable but operationally significant; good when enterprise federation becomes real |
| [ZITADEL](https://github.com/zitadel/zitadel) | Multi-tenant identity infrastructure | AGPLv3 with documented exceptions | **Trigger-based finalist** | Modern APIs and PostgreSQL; self-hosting transfers infrastructure, DDoS, availability and communications responsibility to BUDE |
| [authentik](https://github.com/goauthentik/authentik) | SSO/identity provider and proxy | MIT core, enterprise directory separate | **Internal-SSO candidate** | Strong for staff tools; validate customer-scale identity, supported upgrades and enterprise feature boundaries |
| [Ory Kratos](https://github.com/ory/kratos) | Headless authentication/identity | Apache 2.0 core | **Defer** | Flexible but requires more UI, policy and operational assembly than a pilot should own |

Decision: use a managed identity provider for the pilot. BUDE core still owns organization membership, property assignment and authorization. Run Keycloak vs ZITADEL only when at least one trigger occurs: enterprise SAML/SCIM demand, contractual data-residency need, unacceptable managed cost, or an approved provider-exit program.

### 3.3 ERP, CRM and back office

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [Frappe Framework](https://github.com/frappe/frappe) / [ERPNext](https://github.com/frappe/erpnext) | ERP, approvals, accounting and internal workflows | MIT framework / GPLv3 ERP | **Pilot first** | Broad commodity back-office coverage and existing alignment; keep outside the customer trust boundary |
| [Odoo Community](https://github.com/odoo/odoo) | ERP/CRM/business applications | LGPLv3 community plus proprietary enterprise/apps | **Second-line comparison** | Large ecosystem, but edition/module licensing and customization TCO need careful review |
| [Dolibarr](https://github.com/Dolibarr/dolibarr) | ERP and CRM | GPLv3 | **Watch** | Lighter option but weaker strategic fit than the Frappe pilot unless team capability favors PHP |
| [Twenty](https://github.com/twentyhq/twenty) | Modern CRM | Open-core/mixed; verify exact version | **CRM-only challenger** | Focused CRM UX may beat ERP CRM, but adds PostgreSQL, Redis and another app to operate |
| [EspoCRM](https://github.com/espocrm/espocrm) | CRM and sales automation | GPLv3 core; extensions may differ | **Watch** | Mature CRM option; adopt only if sales requirements exceed Frappe and justify a dedicated system |

Decision: do not run ERPNext, Odoo and a separate CRM together. Pilot Frappe/ERPNext for leads, onboarding, vendors, expenses and approvals. Add a challenger only against documented failures.

### 3.4 Customer support

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [LibreDesk](https://github.com/abhinavxd/libredesk) | Lightweight single-binary support desk | AGPLv3 | **Proof finalist** | Low operational footprint and good API fit, but relatively young |
| [Chatwoot](https://github.com/chatwoot/chatwoot) | Omnichannel support and live chat | MIT core/open-core | **Proof favorite** | Mature channels and ecosystem; heavier stack and some roles, SSO and SLA features are outside Community Edition |
| [Zammad](https://github.com/zammad/zammad) | Full helpdesk/customer support | AGPLv3 | **Proof finalist** | Established helpdesk capabilities and foundation ownership; heavier operations |
| [FreeScout](https://github.com/freescout-help-desk/freescout) | Lightweight shared mailbox/helpdesk | Copyleft core plus paid modules; verify | **Fallback** | Simple and resource-efficient, but required API/SSO/modules may change TCO |

Decision: run one scripted bake-off using the same 20 support scenarios. The winner receives complaint ID and minimum context only. BUDE remains authoritative for customer-visible complaint state. Include a managed helpdesk as the control so self-hosting labor is visible.

### 3.5 Campaigns and notifications

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [listmonk](https://github.com/knadh/listmonk) | Bulk email and newsletters | AGPLv3 | **Campaign proof** | Small footprint and strong bulk-mail fit; BUDE owns consent and suppression |
| [Mautic](https://github.com/mautic/mautic) | Marketing automation | GPLv3 | **Defer** | Powerful but materially heavier than early owner education campaigns need |
| [Novu](https://github.com/novuhq/novu) | Multi-channel notification orchestration | MIT core/commercial enterprise folders | **Trigger-based proof** | Useful after multiple channels/templates/preferences become hard to operate; not the delivery provider itself |
| [ntfy](https://github.com/binwiederhier/ntfy) | Push notifications | Dual/mixed terms; verify exact use | **Watch** | Could support internal alerts, but mobile customer push already has mature managed delivery paths |

Decision: keep transactional and security messages in BUDE adapters. Use listmonk only for consented campaigns. Consider Novu when at least three channels have duplicated routing, template, preference and retry logic.

### 3.6 Events, workflows and automation

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [NATS JetStream](https://github.com/nats-io/nats-server) | Messaging and durable streams | Apache 2.0 | **Trigger-based** | Small compared with Kafka; add only after independent consumer/lag/isolation triggers |
| [Temporal](https://github.com/temporalio/temporal) | Durable long-running workflows | MIT server | **Defer** | Excellent for complex recovery workflows, but a new stateful control plane and programming model |
| [Windmill](https://github.com/windmill-labs/windmill) | Internal scripts, flows and apps | AGPLv3 core/open-core | **Internal automation candidate** | Good for controlled runbooks; workers execute code and require strict isolation, secrets and RBAC |
| [Activepieces](https://github.com/activepieces/activepieces) | Low-code integrations | Open-core/mixed; verify | **Watch** | Useful for non-critical internal automation; prevent customer credentials and core writes |
| [n8n](https://github.com/n8n-io/n8n) | Workflow automation | Source-available Sustainable Use License | **Internal-only legal review** | n8n explicitly says it is not OSI open source and restricts some product/backend uses |

Decision: PostgreSQL outbox plus idempotent workers remains the baseline. Never place booking, payment, authorization or KYC correctness inside a low-code flow. Temporal requires several failure-prone multi-day workflows and a quantified reduction in custom recovery logic. Internal automation may use only scoped APIs.

### 3.7 BI and product analytics

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [Metabase](https://github.com/metabase/metabase) | Internal BI | AGPL core/open-core | **Proof finalist** | Fast path for common internal dashboards; customer embedding and permissions need edition review |
| [Apache Superset](https://github.com/apache/superset) | Enterprise BI/exploration | Apache 2.0 | **Proof finalist** | Flexible and permissive but generally heavier to operate and curate |
| [Lightdash](https://github.com/lightdash/lightdash) | Metrics layer/BI for dbt teams | Open-core/mixed; verify | **Watch** | Valuable after a dbt-based analytics team exists, not before |
| [PostHog](https://github.com/PostHog/posthog) | Product analytics, flags and replay | MIT outside enterprise directory; mixed | **Privacy proof finalist** | Strong integrated capabilities; self-hosting is heavy and session replay is high privacy risk |
| [Matomo](https://github.com/matomo-org/matomo) | Web/product analytics | GPLv3 | **Privacy proof finalist** | Mature privacy-oriented analytics; still requires consent, minimization, retention and access design |
| [Plausible](https://github.com/plausible/analytics) | Lightweight web analytics | AGPLv3 | **Public-site candidate** | Narrow, privacy-friendly website analytics; not a full product-event platform |

Decision: Metabase vs Superset for internal BI using curated, masked, read-only views. PostHog vs Matomo only after an approved event dictionary; session replay is off by default and prohibited on KYC, payment, complaint and authenticated personal-data screens.

### 3.8 Search and feature delivery

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [Meilisearch](https://github.com/meilisearch/meilisearch) | Typo-tolerant application search | MIT core plus BSL enterprise portions | **First search challenger** | Developer-friendly; verify feature/edition boundary and build an index from replaceable events |
| [Typesense](https://github.com/typesense/typesense) | Instant application search | GPLv3 server/open-core services | **Second search challenger** | Good product-search fit; validate geospatial, filters, HA and license obligations |
| [OpenSearch](https://github.com/opensearch-project/OpenSearch) | Distributed search/log analytics | Apache 2.0 | **Defer** | Capable but expensive in memory and operations for pilot listing volume |
| [Flagsmith](https://github.com/Flagsmith/flagsmith) | Feature flags/remote config | Permissive core/open-core | **Trigger-based proof** | Useful for controlled mobile rollouts once database/config flags become unsafe or slow |
| [Unleash](https://github.com/Unleash/unleash) | Feature management | Apache 2.0 core/open-core | **Trigger-based proof** | Mature alternative; compare targeting, audit, SSO and cost |

Decision: PostgreSQL full-text/trigram/geospatial queries first. Test Meilisearch only after realistic data demonstrates search SLO failure or ranking needs PostgreSQL cannot meet. Search indexes are disposable projections and never availability truth.

### 3.9 Observability, security and secrets

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector) | Vendor-neutral telemetry pipeline | Apache 2.0 | **Adopt baseline** | Portable instrumentation and export; redact at source and collector |
| [Grafana OSS stack](https://github.com/grafana/grafana) | Dashboards, metrics, logs and traces | AGPL and component-specific terms | **Adopt selectively** | Strong ecosystem; use only needed components and keep public embedding out |
| [SigNoz](https://github.com/SigNoz/signoz) | Integrated OpenTelemetry observability | Open-core/mixed; verify | **Managed/control candidate** | Unified experience may reduce tool count; self-hosting still adds ClickHouse and operations |
| [Trivy](https://github.com/aquasecurity/trivy) | Dependency, image, IaC and SBOM scanning | Apache 2.0 | **Adopt in CI** | Broad preventive scanning with low integration cost |
| [Dependency-Track](https://github.com/DependencyTrack/dependency-track) | SBOM/vulnerability portfolio | Apache 2.0 | **Adopt at portfolio trigger** | Useful when multiple deployables make spreadsheet/SBOM review unreliable |
| [OWASP ZAP](https://github.com/zaproxy/zaproxy) | Dynamic application security testing | Apache 2.0 | **Adopt in staging CI** | Automates baseline DAST; does not replace manual authorization testing |
| [OpenBao](https://github.com/openbao/openbao) | Secrets and encryption management | MPL 2.0 | **Trigger-based alternative** | Genuine FOSS secrets platform, but operating availability/unseal/recovery safely is substantial |

Decision: use cloud secret management first, OpenTelemetry, Trivy and ZAP immediately. Add Dependency-Track when CI reports no longer cover the portfolio. Compare OpenBao only if multi-cloud/on-premises requirements justify a dedicated control plane. [HashiCorp Vault](https://github.com/hashicorp/vault/blob/main/LICENSE) is source-available under BSL for current releases and is not the default FOSS choice.

### 3.10 Storage, documents, billing and domain systems

| Project | Typical use | License posture | BUDE decision | Reason |
|---|---|---|---|---|
| [SeaweedFS](https://github.com/seaweedfs/seaweedfs) | Distributed S3/file storage | Apache 2.0 core | **Future self-host finalist** | Credible maintained alternative, but storage integrity/HA/recovery should not burden the pilot team |
| [MinIO Community](https://github.com/minio/minio) | S3-compatible object storage | AGPLv3; repository archived in 2026 | **Do not adopt new** | Official repository says no longer maintained; use managed storage or evaluate maintained alternatives |
| [Gotenberg](https://github.com/gotenberg/gotenberg) | HTML/office-to-PDF service | Permissive/MIT | **Trigger-based utility** | Useful for reproducible receipts/agreements; sandbox inputs and keep evidence in core |
| [Documenso](https://github.com/documenso/documenso) | Electronic document signing | AGPLv3 core/open-core | **Legal/compliance proof only** | Self-host docs warn the operator owns network/SSRF controls; signing is not automatically legally sufficient in India |
| [Lago](https://github.com/getlago/lago) | Usage/subscription billing | AGPLv3 | **Defer to owner SaaS billing only** | Could support complex plans later; never the tenant rent/deposit ledger |
| [Kill Bill](https://github.com/killbill/killbill) | Subscription billing/payments | Apache 2.0 | **Defer** | Mature and powerful but too heavy for initial simple owner subscriptions |
| [QloApps](https://github.com/qloapps/qloapps) | Hotel PMS and booking engine | OSL/AFL and module-specific terms | **Reference workflows; reject as core** | Nightly hotel inventory and module terms do not match monthly individual-bed PG stays |

Decision: managed S3-compatible storage with independent exports and restore tests. Generate PDFs in the application first, then extract Gotenberg if required. Build BUDE's tenant financial ledger. Evaluate Lago only when owner subscription pricing becomes genuinely complex.

---

## 4. Recommended shortlist by phase

| Phase | Adopt or evaluate | Explicitly excluded |
|---|---|---|
| Pilot foundation | FastAPI, PostgreSQL/PostGIS, Flutter, OpenTelemetry, Trivy, ZAP, managed identity/storage/edge/communications | Alternative BaaS, self-host identity/storage, search cluster, workflow engine |
| Internal operations proof | Frappe/ERPNext; support bake-off; Metabase vs Superset; listmonk | Multiple CRMs/helpdesks/BI tools in production |
| Product learning proof | Matomo vs PostHog with approved event dictionary; PostgreSQL search baseline | Session replay on sensitive screens, customer-facing BI |
| Scale-triggered | NATS, Meilisearch/Typesense, Flagsmith/Unleash, Novu | Kafka/OpenSearch/Temporal until smaller options fail measured needs |
| Later strategic | Keycloak/ZITADEL, OpenBao, SeaweedFS, Temporal, Lago | Self-hosting based only on VM-price comparisons |

### Current preferred production set if all proofs pass

1. FastAPI core and worker.
2. Managed PostgreSQL/PostGIS.
3. Managed identity with BUDE authorization.
4. Managed private object storage.
5. Flutter clients.
6. Frappe/ERPNext for selected internal workflows.
7. One support winner, probably Chatwoot or LibreDesk after evidence.
8. listmonk for consented bulk campaigns.
9. One internal BI winner, probably Metabase after evidence.
10. OpenTelemetry with managed or minimal Grafana-compatible observability.

This is already near the pilot deployable limit. Every additional service must replace something or pass an architecture exception.

---

## 5. Six-week task and micro-task evaluation plan

### E1 - Freeze requirements and disqualifiers (Days 1-3)

- [ ] Name product, security, legal, platform and operational reviewers.
- [ ] Convert each satellite category into executable user scenarios.
- [ ] Mark customer-critical, PII, KYC, payment, tenant-scope and audit scenarios.
- [ ] Approve hard gates and weighted score before installing candidates.
- [ ] Define the managed control and current manual cost for each bake-off.
- [ ] Cap simultaneous proofs at two and time-box each to five engineering days.
- [ ] Create an evidence template: version, digest, license, SBOM, scans, config, tests, cost and decision.

Gate: criteria are frozen; no project wins because of brand, stars or founder preference.

### E2 - Paper screen all candidates (Days 2-7)

- [ ] Record repository, exact release, image publisher and immutable digest.
- [ ] Record core, enterprise, plugin, dependency and trademark terms.
- [ ] Confirm security policy, advisory channel, supported releases and recent security fixes.
- [ ] Inspect required databases, queues, caches, object stores, workers and minimum resources.
- [ ] Map all inbound/outbound network access and third-party telemetry.
- [ ] Map stored PII, encryption, retention, deletion, export and audit capability.
- [ ] Reject archived, unknown-license, unpatchable or core-database-writing candidates.
- [ ] Calculate monthly TCO including operator and on-call hours.

Gate: only candidates with no known hard-gate failure enter hands-on proof.

### E3 - Support bake-off (Week 2)

- [ ] Install pinned LibreDesk, Chatwoot and Zammad in disposable isolated environments.
- [ ] Execute identical email, chat, assignment, escalation, attachment, export and deletion scenarios.
- [ ] Verify roles, SSO/MFA, audit, API/webhooks, rate limits and edition/paywall boundaries.
- [ ] Send only synthetic data and stable BUDE complaint/customer IDs.
- [ ] Simulate BUDE outage, helpdesk outage, duplicate webhook and malicious attachment.
- [ ] Perform backup, clean restore, upgrade, rollback and full removal.
- [ ] Compare resource use and projected operator cost with a managed control.
- [ ] Select one winner or the managed control; uninstall all losers.

Gate: BUDE complaint truth and customer workflows survive complete helpdesk loss.

### E4 - Back-office and BI proofs (Weeks 2-3)

- [ ] Configure only lead, onboarding, vendor, expense and approval workflows in Frappe/ERPNext.
- [ ] Record gaps before considering Odoo or a dedicated CRM.
- [ ] Exclude KYC and payment credentials from all projections.
- [ ] Build a versioned outbox-to-back-office adapter with retries and reconciliation.
- [ ] Run Metabase and Superset against identical masked, read-only curated views.
- [ ] Test tenant isolation, query timeouts, export controls, roles and OLTP load protection.
- [ ] Restore both tools and measure dashboard portability.
- [ ] Choose one BI tool and remove the other.

Gate: internal tools save measured time and cannot alter or exhaust the core.

### E5 - Product analytics privacy proof (Week 4)

- [ ] Approve an event dictionary with purpose, lawful basis, fields, owner and retention.
- [ ] Prohibit names, contact details, government IDs, addresses, payment data and signed URLs.
- [ ] Compare Matomo and PostHog using identical synthetic mobile/web events.
- [ ] Keep session replay disabled; document any future exceptional review path.
- [ ] Test consent withdrawal, deletion, identity reset, export and retention expiry.
- [ ] Verify mobile performance, offline buffering, duplicate events and kill switch.
- [ ] Compare managed versus self-hosted TCO and data locations.
- [ ] Choose one narrow deployment or defer both.

Gate: analytics can be disabled or deleted without affecting the product and passes privacy review.

### E6 - Core baseline challenge tests (Week 4)

- [ ] Load realistic property, bed, amenity, rule and geospatial test data.
- [ ] Benchmark PostgreSQL FTS/trigram/PostGIS relevance and p95 latency.
- [ ] Benchmark outbox throughput, lag, retries, replay and poison-message handling.
- [ ] Model notification routing complexity across current channels.
- [ ] Model multi-day workflow recovery complexity in application code.
- [ ] Do not install Meilisearch, NATS, Novu or Temporal unless its trigger fails.
- [ ] If a trigger fails, prove the smallest challenger with a disposable projection.
- [ ] Publish retain-baseline or adopt-challenger ADRs with measurements.

Gate: specialized infrastructure exists only to solve a reproduced problem.

### E7 - Adversarial operations review (Week 5)

- [ ] Generate and retain SBOMs for every finalist.
- [ ] Run source, dependency, image, IaC, secret and DAST scans.
- [ ] Test default credentials, reset, revocation, CSRF, SSRF and webhook signatures.
- [ ] Test cross-tenant access and over-privileged service accounts.
- [ ] Test disk full, database unavailable, provider timeout, lost worker and corrupt backup.
- [ ] Time clean restore to a new environment and compare with RPO/RTO.
- [ ] Verify logs and telemetry contain no prohibited fields.
- [ ] Confirm removal and data deletion evidence.

Gate: unresolved critical/high issues, restore failure or isolation failure blocks adoption.

### E8 - Portfolio decision and production freeze (Week 6)

- [ ] Review evidence and independent security/legal objections.
- [ ] Count deployables, databases, queues, caches, secrets, backups and alerts.
- [ ] Require two owners, runbook, patch SLA, upgrade calendar and exit plan per component.
- [ ] Calculate 12-month risk-adjusted TCO against managed controls.
- [ ] Approve adopt, managed, defer, reject and re-review dates.
- [ ] Delete rejected proof environments, credentials and synthetic data.
- [ ] Update architecture, threat model, data map, SBOM and production tasks.
- [ ] Schedule quarterly license, maintenance, security, recovery and value reviews.

Gate: the pilot remains within six product-team-owned deployables unless leadership approves a measured exception.

---

## 6. Trigger register

| Trigger | Measure | Candidate response |
|---|---|---|
| PostgreSQL search misses SLO after tuning | Realistic load test and production trace | Meilisearch first, Typesense second |
| Three independent event consumers or sustained outbox lag | Consumer inventory and lag SLO | NATS JetStream proof |
| Critical multi-day workflows require complex manual recovery | Incident and recovery-code review | Temporal proof |
| Three channels duplicate routing, preferences, templates and retries | Notification code/incident review | Novu proof |
| Mobile rollout needs targeting, kill switches and audited changes | Release risk review | Flagsmith vs Unleash proof |
| Identity cost, federation or residency becomes material | Contract, SSO pipeline and cost data | Keycloak vs ZITADEL proof |
| Secret rotation/multi-environment access exceeds managed fit | Secrets inventory and rotation evidence | OpenBao proof |
| Managed object storage fails cost/residency/control gates | Risk-adjusted TCO and recovery proof | SeaweedFS proof |
| Owner pricing becomes usage-based or multi-dimensional | Pricing catalog and billing exception count | Lago vs Kill Bill proof |

---

## 7. Architecture rules for every adopted project

- One authority per fact; satellite data is a projection or satellite-native commodity state.
- No direct writes to BUDE tables and no shared database owner credentials.
- No synchronous dependency from check-in, allocation, invoicing, payment, reconciliation or access control to a satellite.
- Versioned APIs/events, idempotent consumers, retry limits, dead-letter handling and reconciliation.
- Private network by default, staff MFA/SSO, least privilege and audited break-glass access.
- Separate data, secrets, retention, backup, restore and operational owner.
- Immutable image digest and SBOM; no `latest` tags.
- Automatic security alerts with critical/high remediation SLA.
- Configuration and necessary customization in version control.
- Quarterly clean restore and annual removal exercise.
- No fork unless strategic, upstream is attempted and divergence budget is approved.
- Exit without losing authoritative BUDE data.

---

## 8. Primary official references

- [Appwrite self-hosting responsibilities](https://appwrite.io/docs/advanced/self-hosting)
- [Keycloak repository and Apache 2.0 license](https://github.com/keycloak/keycloak)
- [ZITADEL self-hosting requirements](https://zitadel.com/docs/self-hosting/manage/requirements)
- [ZITADEL self-hosted responsibility split](https://zitadel.com/docs/legal/service-description/cloud-service-description)
- [authentik mixed core/enterprise license](https://github.com/goauthentik/authentik/blob/main/LICENSE)
- [Ory Kratos security/support policy](https://github.com/ory/kratos/security)
- [Chatwoot production architecture](https://developers.chatwoot.com/self-hosted/deployment/architecture)
- [Chatwoot Community Edition feature comparison](https://www.chatwoot.com/pricing/self-hosted-plans)
- [Zammad repository and AGPL license](https://github.com/zammad/zammad)
- [n8n Sustainable Use License](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)
- [Temporal server MIT license](https://github.com/temporalio/temporal/blob/main/LICENSE)
- [Windmill self-hosting and execution isolation](https://www.windmill.dev/docs/advanced/self_host)
- [PostHog mixed MIT/enterprise license](https://github.com/PostHog/posthog/blob/master/LICENSE)
- [Meilisearch mixed MIT/BSL license](https://github.com/meilisearch/meilisearch/blob/main/LICENSE)
- [OpenSearch Apache 2.0 license](https://github.com/opensearch-project/OpenSearch/blob/main/LICENSE.txt)
- [SeaweedFS repository and Apache 2.0 license](https://github.com/seaweedfs/seaweedfs)
- [MinIO archived community repository](https://github.com/minio/minio)
- [OpenBao overview](https://openbao.org/docs/what-is-openbao/)
- [Sentry self-hosted FSL notice](https://github.com/getsentry/develop/blob/master/src/docs/self-hosted/index.mdx)
- [Directus current license](https://directus.com/license)
- [Documenso self-hosting security responsibilities](https://github.com/documenso/documenso/blob/main/apps/docs/content/docs/self-hosting/index.mdx)
- [Lago repository and AGPL license](https://github.com/getlago/lago)
- [Kill Bill repository and Apache 2.0 license](https://github.com/killbill/killbill)
- [QloApps repository, domain and license](https://github.com/qloapps/qloapps)

Recheck every version, license, security status, edition boundary, commercial term and operational claim immediately before proof and again before production adoption.
