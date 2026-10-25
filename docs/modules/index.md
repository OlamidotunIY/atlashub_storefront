# Module implementation plan — Atlas Storefront

**State:** Architecture/specification only, no module is built. This backlog supersedes the earlier declarative-only editor plan. DDD+Clean Architecture/Ports & Adapters with pragmatic CQRS; no additional backend GitHub repository.

## Independently deployed repositories and backend modules (no Turborepo/Nx)

| Location (proposed) | Responsibility | Key acceptance criteria |
|---|---|---|
| `atlashub_storefront_portal` (separate repo) | Dedicated merchant and creator frontend, theme marketplace/settings/domain UI | Independent deployment; no duplicated Atlas Web business operations |
| `atlashub_auth_web` (separate repo) | Shared registration/login/recovery frontend for all Atlas products | Individual registration independent of organization, central SSO with Java/IdP authority |
| `apps/management-api` | Independent TypeScript/Node DDD backend and PostgreSQL state for themes, installations, config, domains, creators, releases, assets, work items | Tenant/creator authorization on every action, mutation idempotency, contract tests with Java |
| `apps/build-worker` | Disposable/quarantined source intake, dependency validation, Vite/SSR compilation, audits, reproducible artifacts | No secrets/privileged network, per-build CPU/storage/time limits, only approved release promoted |
| `apps/publishing-worker` | Review release, install tenant Worker, sync domain routes/TLS, invalidate caches, report status, rollback | Durable workflows with retries/compensation/idempotency |
| `apps/storefront-runtime` | Cloudflare dispatch and per-installation untrusted user Worker execution, cache, SSR, quotas and egress restrictions | Unknown hostname fail closed, independent tenant Worker and public catalog context, checkout isolated |
| `atlashub_theme_sdk` (proposed separate repo) | Official Vite/React/React Router scaffold, required routes, defaults, dev mocks, sample settings | Repository not verified created; versions published separately, no secret API keys |
| `atlashub_theme_sdk` (planned SDK package) | Typed public storefront data client, hooks, safe cart/checkout launch, asset/settings access | No arbitrary admin APIs, org context server-attested, Java backend recalculates prices |
| `atlashub_theme_sdk` (planned CLI package) | login/logout/whoami, dev/validate/build/preview/publish, list/releases/unlist | Scope-limited CLI token, version conflict detection and detailed diagnostics |
| `contracts/` (in this backend) + published theme contracts (planned SDK repo) | Versioned theme manifest, API/schema/route compatibility | Defaults validate and published cross-repo contracts tested |
| `workers/template-runtime` (backend hosting) | Deployment/dispatch/Worker SSR adapter; runtime contract with SDK repo | SSR/prerender indexable pages plus compatible isolated Worker environment |
| `atlashub_storefront_portal` (separate repo) | Portal design system and merchant/creator widgets | No portal UI in hosting/backend repository |

## Bounded contexts and design documents

| Domain | Key aggregates/entities | Documentation |
|---|---|---|
| Creator accounts | CreatorProfile, APIGrant, SubmissionRole | [Portal and SSO](../architecture/portal-identity.md) |
| Theme registry | Theme, ThemeVersion, ThemeRelease | [Theme contract](../contracts/theme-manifest.md) |
| Editor/config | StorefrontConfiguration, ConfigRevision, ThemeSettings | [Developer toolkit](../developer/sdk-cli.md) |
| Build and approval | BuildJob, ValidationReport, ReviewCase | [Marketplace lifecycle](../architecture/marketplace-lifecycle.md) |
| Marketplace | Listing, EntitlementLink, Sale, CreatorEarning | [Marketplace lifecycle](../architecture/marketplace-lifecycle.md) |
| Installation/publication | Installation, Storefront, Publication, Rollback | [Runtime/hosting](../architecture/runtime-hosting.md) |
| Hosting/domains | WorkerDeployment, DomainBinding, CertificateStatus | [Domains/SEO/checkout](../architecture/domains-checkout-seo.md) |
| Assets/storage | Asset, SourceArchive, ReleaseArtifact, UploadSession | [Storage](../architecture/storage-data.md) |
| Commerce integration | VerifiedStorefrontContext, PublicProductView, CartSession | [Platform APIs](../contracts/platform-apis.md) |
| Security/limits | TenantPolicy, CapabilityPolicy, AuditEntry | [Threat model](../security/theme-sandbox.md) |
| Observability/finops | UsageRecord, CostAllocation, Alert, SLO | [Operations](../architecture/operations-costs.md) |

## Phase 0 — contracts, identity, environment and threat model

- Read Java IAM/auth and Atlas Web sign-in behavior; define central IdP OIDC/BFF SSO and creator registration without merchant registration. No claims existing AtlasHub auth is already OIDC-ready.
- Define API versioning, public product/card/checkout contracts, merchant entitlements, organization and storefront IDs, trusted tenant routing and published product read models.
- Define developer manifest, required pages, settings defaults, legal asset/capability schema, immutable release/status models.
- Decide component/runtime allowlist, CSP, same-origin and checkout rules, per-store budgets and PII lifecycle.
- Confirm selected Workers for Platforms account, untrusted mode, outbound restrictions, R2/CDN, Cloudflare for SaaS feature availability.
**Exit:** reviewable specs + cross-repo contract tests scaffolded; signed off threat model and budget.

## Phase 1 — proof of concept, no public marketplace

- Implement Vite React Router scaffold with Home, Collection, Product, Cart and Static pages, fixtures and settings SDK.
- Two separately authored React themes; two distinct organizations, domains, currencies and catalog data.
- Compile React Router SSR bundle for Workers for Platforms *untrusted user Worker* and deploy one Worker per installation via dispatch Worker.
- Verify CDN static assets, SSR HTML/meta/schema.org, browser hydration, cart-to-trusted-checkout handoff, cache purge after catalog change.
- Test malicious outbound fetch and same-origin checkout/account isolation.
- Compare same-theme/different-merchant installations; verify no cross-tenant leakage, CPU/memory/quota ceilings, p95 TTFB.
**Exit:** deployment, SSR, browser-security and economics feasible. If not, amend runtime ADR before shipping.

## Phase 2 — creator tooling and marketplace

- Authenticated CLI flows: `login`, `logout`, `whoami`, `theme init`, `dev`, `validate`, `build`, `preview`, `publish`, `versions`, `unlist`; one authoritative server-side validation pipeline.
- Private source intake, isolated ephemeral builds with network/secret restrictions, dependency lockfile, signed immutable release assets, staged approval.
- Creator portal: profile, themes, drafts, release diagnostics, review cases, listing, unlisting, sales/earnings.
- Publishing state machine: LOCAL -> UPLOADED -> VALIDATING -> BUILD_FAILED/NEEDS_CHANGES -> IN_REVIEW -> APPROVED -> LISTED; version immutable. Unlisting does not automatically break installations.
**Exit:** first third-party theme reviewed, traceable release and creator UX.

## Phase 3 — merchant portal and public storefront operations

- Merchant portal: listings/recommendations, purchase/install, per-store settings forms, asset uploader, preview/publish, theme updates, domains/certificates, store status, rollback, usage.
- Link from Atlas Web theme suggestions; full flow owned by portal. SSO and organization selector enforce Java membership/RBAC.
- Domain verification and TLS, production Worker deployments and dispatch mapping, published/draft config snapshots, backups and cleanup.
- Licensing and theme sales through Atlas Hub-authoritative billing; creator payouts only via approved provider arrangements.
**Exit:** merchant can go from signed-in Atlas Web recommendation to a purchased live store without second login.

## Phase 4 — hardened global scale

- Multiple storefronts/outlets, markets, locales, currency and region-legal availability constraints; accessibility and SEO testing.
- Long-term theme SDK compatibility and migrations, immutable release diffs, staged upgrade policies, instant rollback.
- Product/inventory invalidation, cache observability, custom domain renewals, outage fallbacks, security incident handling.
- Storage and Worker request/script/CPU cost dashboards, creator fraud/abuse and moderation, rights management.
**Exit:** production threat tests, billing reconciliation, SLOs, load and catastrophe recovery drills all passed.

## Test inventory (mandatory)

- Identity: no login-token redirect; session rotation, logout, role switch, creator-without-business, merchant-without-creator, org switch and expired sessions.
- Tenant: manipulated Host, DNS reclaim, false asset IDs, cross-store cache, cross-org direct API, injected storefront ID, reused preview URL.
- Theme: malicious NPM hooks, external scripts, encoded XSS, browser exfiltration, CORS, untrusted SSR, timeout, memory and dependency supply chain.
- Data: public API contract, stale price and inventory, duplicate orders, cart race conditions, payment processor eligibility and trusted checkout, refund status reconciliation.
- Publication: duplicate submissions, review race, deploy failure, domain pending, rollback, unlist, creator edits, merchant override retention and update migration.
- Accessibility/SEO: responsive mobile, keyboard, headings/structured product metadata, crawler HTML, Core Web Vitals, broken images and localized URLs.
- Operations: Worker script counts/CPU/subrequests, R2 storage/read/write cost, CDN hit ratio, egress blocks, domain certificate expiry, asset orphan cleanup, audit retention.

**Nothing in this document is an implementation-complete claim.**
