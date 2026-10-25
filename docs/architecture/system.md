# Storefront Platform — end-to-end system architecture

**Status:** Agreed architectural direction, **implementation pending prototype**. Everything below describes planned capability, not an existing deployment.

## Product experience and frontends

The existing **Atlas Web** remains merchant business operations: POS, finance, orders, HR, accounting, inventory, general settings and storefront **recommendation cards**. Clicking a recommendation or Storefront link opens the **dedicated Storefront Portal** (new `apps/portal` in this repository) at a deep link to that marketplace listing, with central Atlas Hub SSO.

The Storefront Portal has:
- **Merchant workspace:** search marketplace, compare prices/previews, buy/claim licensed theme, install, choose active version, configure theme settings/branding/media, preview draft, connect domain, request TLS, publish/rollback/update, manage stores/assets/licenses/billing.
- **Creator workspace:** register developer profile independently of merchant organization, issue scoped developer CLI credentials, upload versions, build/review diagnostics, submit to marketplace, drafts/unlisting/deprecation, edit listing/pricing, see installed-version and sales/earnings reports.
- **Shared account and portal session:** one Atlas Hub user can be both creator and member of multiple organizations; workspace/organization explicitly selected. No duplicate merchant dashboard in the portal for commerce, HR or accounting, and no independent creator authentication directory.

No drag-and-drop theme construction in MVP. Merchant customization is through form fields derived from developer-owned versioned JSON/schema and valid defaults.

## Technology and services

- **AtlasHub Java backend**: authoritative organization identity, membership/RBAC, product catalog, published price, inventory, fulfillment, orders, merchant-connected payments, checkout, subscription/theme entitlements and platform accounting.
- **Storefront TypeScript management API** (Node.js): DDD/Clean/ports-adapters business functions for creators, theme registry/version/review, source uploads, assets, installations, storefront settings, domain binding, publisher jobs and access enforcement. It uses **its own PostgreSQL** metadata store; it is a backend but does **not** need a new GitHub repo.
- **Portal**: React+Vite TypeScript app serving merchant and creator workspaces with SSO and BFF/session strategy.
- **Theme SDK and CLI**: Vite scaffolding, React Router route contracts, typed safe storefront queries/hooks and local tests/mocks, signed-in `theme` CLI.
- **Build worker**: receive source archive via private signed upload; isolate dependencies/npm lifecycle scripts, scan/validate/audit, enforce route/config/asset limits, produce immutable assets plus edge-compatible SSR module.
- **Theme hosting**: Cloudflare Workers for Platforms in **untrusted** dispatch namespace; initial **separate user Worker per merchant storefront installation**, all behind a shared dispatch Worker with trusted hostname verification, environment/quota enforcement and optional outbound Worker for constrained egress. Production bundle references the approved immutable theme release; do not run the developer's uploaded dev server. This is the selected candidate only after benchmark/security feasibility gates.
- **R2/CDN**: source quarantine (private), approved releases (immutable, controlled public asset delivery), merchant storefront branding/uploads (tenant-scoped). **Not** Java business OLTP database.
- **Cloudflare for SaaS**: approved merchant domains, verification, TLS management, mapping to storefront, automatic renewal monitoring. A domain claim without verified ownership and certificate readiness cannot activate.
- **Jobs and events**: build queues, release validation, review, deployment, domain activation, asset sanitation, cache invalidation, license/accounting sync. Required idempotency, status and retries.

## Authoritative boundaries

Atlas Hub Java: User, Organization, Membership, Role, product, prices, inventory, order, tax, payment choices, checkout/refund, platform subscription/billing/entitlement. Storefront platform: CreatorProfile, Theme, ThemeVersion, ThemeRelease, ThemeListing, Review, Installation, Storefront, StorefrontConfiguration, DomainBinding, Asset, Deployment, PortalSession, ThemeSaleReport (Java billing/financial events remain authoritative). Theme render code: display only its allowed storefront scope.

Cross-repository links:
- https://github.com/OlamidotunIY/atlashub_backend
- https://github.com/OlamidotunIY/atlashub_web
- https://github.com/OlamidotunIY/atlashub_storefront

## Domain-Driven Design and Clean Architecture

Proposed bounded contexts: CreatorIdentityAdapter (not its own identity provider), ThemeCatalog, ThemeAuthoring, ThemeBuildValidation, ThemeReview, ThemeMarketplace, ThemeLicense, ThemeInstallation, StorefrontConfiguration, StorefrontPublication, DomainProvisioning, AssetManagement, CommerceIntegration, DeploymentControl and Audit.

Domain: aggregates, immutable release/value objects, transitions/eligibility. Application: typed use cases, commands, queries, policies. Infrastructure: PostgreSQL, R2, Cloudflare adapters, jobs, SSO and Java API clients. Presentation: portal, management REST endpoints, CLI auth/upload/preview endpoints, customer SSR gateway.

CQRS for stateful author/release/publication and catalog/search views. Not every raw HTTP request, audio, render chunk or product fetch needs a command bus. Services are logical boundaries within one repository, not mandatory premature microservices.

## Typical merchant journey

1. Business sees recommendation in Atlas Web and opens Storefront Portal listing via canonical deep link.
2. Browser completes central SSO; portal receives *its own* scoped session; organization authorization resolved from Java, not URL alone.
3. Merchant purchases/claims a theme via Atlas Hub controlled billing, receives entitlement; installs a specific approved release.
4. Creates storefront draft, assigns logo/banner/colors/text/etc (schema-validated merchant overrides, developer defaults as fallback), selects market/locale and verified asset IDs.
5. Preview only in an isolated, non-indexable environment; connect and verify domain with TLS.
6. Publish atomically: storefront -> immutable ThemeRelease + specific validated config version + verified domain + deployed Worker. Preview and published URLs never reveal private merchant data.
7. Customer requests domain; dispatch resolves verified domain to storefront, calls isolated Worker, renders published theme using scoped public commerce API/data (may SSR/pre-render), returns HTML and static assets from CDN.
8. Checkout sends cart to Java for authoritative price/tax/inventory/payment/shipping validation; present trusted checkout; process provider webhooks at authoritative Java service.
9. Merchant edits assets/config, previews/publishes again or rollbacks; code-release upgrades do not silently override their stored values.

## Creator journey

Create Atlas account or sign in -> register creator identity (no required merchant business) -> CLI login -> scaffold React theme -> local preview with fixture/mocks -> validate/build -> upload version into private quarantine -> isolated build+contract/security scan -> creator dashboard preview/errors -> submit for approval -> approved listing visible to eligible merchants -> versioned sales/licensing/updates/unlisting.

## No duplication and no unrestricted access

Atlas Web has recommendations/deep links only. Portal hosts full theme/store management. Java is commerce source of truth; theme runtime is not the provider payment processor, ledger or identity authority. Merchants must not re-enter their products merely because they installed a theme. The same theme release may be installed by many stores without copying its source for each one.

## Architectural risks to verify

1. React Router framework SSR/prerender adapter successfully compiles and deploys within Workers for Platforms user Worker and CPU/memory restrictions.
2. One Worker per store provides intended isolation, but deployment count, lifecycle, compatibility and costs are verified (not equated to 1 VM/store).
3. Browser-executed third-party scripts retain browser power; require CSP, separate origin for checkout/customer account, security review, asset isolation, egress policy, dependency policy.
4. Identity SSO to existing Java auth is **not built** just by placing a link in Atlas Web.
5. Tenant assets/cache, domain takeover and cross-tenant leakage are tested.
6. Price/stock reconciliation, SEO structured data, a11y and storefront latency meet defined gates.

Full contracts, security and operational rules are in adjacent files. No current feature is represented as production-ready.
