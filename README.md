# Atlas Hub Storefront Platform

**Status:** Architecture selected for prototyping; documentation only. No portal, hosting runtime, SDK, CLI or marketplace is deployed.

Atlas Storefront is the **independent TypeScript management backend and hosting service** for full React themes, immutable uploads and releases, creator marketplace metadata, asset storage, domains and isolated storefront rendering. Merchants and creators use a **separate frontend repository**, [atlashub_storefront_portal](https://github.com/OlamidotunIY/atlashub_storefront_portal), authenticated through the separate [atlashub_auth_web](https://github.com/OlamidotunIY/atlashub_auth_web) frontend and Java-managed central identity. Atlas Web only surfaces merchant storefront recommendations/deep links. Individual registration does not create or require an organization or compulsory Anchor/KYB onboarding.

## Repository boundaries — standalone repositories, no monorepo tools

| Repository | Ownership |
|---|---|
| [atlashub_backend](https://github.com/OlamidotunIY/atlashub_backend) | Existing Java business system of record, IAM/auth, commerce, orders, inventory, checkout, billing and payment/shipping capability policy |
| [atlashub_web](https://github.com/OlamidotunIY/atlashub_web) | Existing merchant operations dashboard; storefront recommendations and SSO deep links, **not** duplicate template management |
| **atlashub_storefront (this repository)** | Node/TypeScript Storefront management backend, theme build/review/publishing jobs, tenant deployment/runtime and domains, storage adapters and API contracts |
| [atlashub_storefront_portal](https://github.com/OlamidotunIY/atlashub_storefront_portal) | Independent React/Vite Merchant+Creator Portal frontend, marketplace UI and storefront management pages |
| [atlashub_auth_web](https://github.com/OlamidotunIY/atlashub_auth_web) | Independent React/Vite central login, account signup, recovery/MFA user interface; Java/IdP owns authentication backend |
| [atlashub_theme_sdk](https://github.com/OlamidotunIY) *(proposed, repo not verified)* | Independently versioned Vite React scaffolder, public Storefront SDK, theme CLI, contracts and examples |

**Do not use a monorepo or Turborepo/Nx.** Repositories are independent. This repository groups the related backend and hosting components; portal/auth frontend and developer SDK/CLI belong in separate repositories. There is no new standalone authentication backend: existing Java `auth`/IAM (or integrated OIDC provider) remains authoritative.

## Decisions and guardrails

- **Theme creation:** Vite + React + TypeScript; React Router Framework Mode for SSR/pre-rendering. Creator owns layout, CSS, animation and approved client-side interactivity; Atlas supplies typed route contracts, storefront SDK and theme manifest/settings schema.
- **Merchant customization:** Developer must define valid defaults for configurable settings (logo, colors, banners, fonts, homepage copy, featured collections etc). Merchant overrides are tenant/storefront-scoped, validated and retained on upgrades. No drag-and-drop editor in initial scope.
- **Dashboard:** Separate `atlashub_storefront_portal` repository for merchant and creator role-aware workspaces. A user may hold both. Atlas Web displays recommendations/deep links; Portal owns all storefront/marketplace management. Central login/signup UI in separate `atlashub_auth_web`.
- **Authentication:** central Atlas Hub identity, OIDC Authorization Code + PKCE or secure BFF; existing frontend login must be extended to support SSO. Same identity, independent app sessions and authorization; never redirect with tokens.
- **SDK (separate planned repository):** approved public storefront data access (catalog, collections, product details, cart, checkout initiation), scaffold and theme CLI published via versioned packages. No admin/HR/ledger access or merchant/provider secrets. Data sourced through Java-authoritative backend.
- **Publishing:** developer CLI -> private source quarantine -> locked-down build/review -> immutable release -> marketplace approval -> license/install -> tenant preview/publish/rollback.
- **Chosen deployment direction:** Cloudflare Workers for Platforms **untrusted mode**, per-merchant-installation isolated Worker as initial unit of execution, dynamic dispatch Worker for verified host routing and limits; object storage/CDN for immutable built assets. This choice remains **conditional on prototype proving React Router SSR, isolation, performance and unit economics**.
- **Storage:** separate source archives, approved release bundles and merchant public assets in R2/object storage. PostgreSQL metadata/settings; existing Java-backed storage authoritative for inventory, products, orders, payments and business data.
- **Domains:** Cloudflare for SaaS with ownership verification, custom hostnames, TLS state and fail-closed unknown-host routing.
- **Security:** strict capability boundary, isolated builds, no arbitrary privileged server code, restricted egress, CSP/content reviews, trusted checkout and step-up where relevant. Sandboxing server execution does **not** make arbitrary browser scripts safe.
- **Operations:** measurable p95 page latency, SEO, accessibility, caching/invalidation, per-tenant quota/billing, replay-safe publication, domain and rollback runbooks.
- **Theme lifecycle:** immutable versioning, review, release channels, creator unlisting without automatically breaking purchased installations, compatibility and preservation of merchant overrides.

## Documentation map (authoritative proposals)

- [Architecture and repo/service boundaries](docs/architecture/system.md)
- [Portal workspaces and central SSO](docs/architecture/portal-identity.md)
- [Creator scaffolding, SDK and CLI](docs/developer/sdk-cli.md)
- [Theme metadata and SDK contracts](docs/contracts/theme-manifest.md)
- [Cross-repository API/event contract](docs/contracts/platform-apis.md)
- [Publishing, review, licensing and upgrades](docs/architecture/marketplace-lifecycle.md)
- [Object storage, merchant data and lifecycle](docs/architecture/storage-data.md)
- [Workers isolation, dispatch, SEO and hosting](docs/architecture/runtime-hosting.md)
- [Custom domains, customer checkout and caching](docs/architecture/domains-checkout-seo.md)
- [Creator-code and platform security](docs/security/theme-sandbox.md)
- [Operations, scaling, pricing and observability](docs/architecture/operations-costs.md)
- [Module-by-module backlog, sequence and test gates](docs/modules/index.md)
- [ADR 0001 — selected runtime and alternative](docs/decisions/0001-runtime-options.md)
- [ADR 0002 — repository, portal and identity](docs/decisions/0002-portal-and-backend-boundaries.md)

## Proposed backend/hosting code structure (not implemented yet)

```text
src/
  domain/                 # Theme/Creator/Installation/Publication/Domain aggregates
  application/            # Commands/queries, orchestration, services
  infrastructure/         # PostgreSQL, R2, Cloudflare, queues, Java API adapters
  presentation/           # Backend HTTP/BFF-facing APIs
workers/
  build/                  # Isolated build worker process/deployment
  publish/                # Deployment/review/cache/domain jobs
  dispatch/               # Cloudflare trusted dispatch Worker
  template-runtime/       # Trusted adapter; third-party code in isolated user Workers
contracts/                # Versioned API/proto/JSON schema for other repos
docs/
```

No code structure above implies implementation is complete. Portal frontend lives in a different repository. The Java backend migration removes mandatory Atlas-managed banking/KYB and separates individual User creation from optional Organization creation. We first implement and evaluate an end-to-end proof of concept with two independently authored themes and two unrelated merchants.

## Reference services (architecture, not deployment claims)

[Workers for Platforms](https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/), [worker isolation](https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/worker-isolation/), [Cloudflare for SaaS](https://developers.cloudflare.com/cloudflare-for-platforms/cloudflare-for-saas/start/getting-started/), [R2](https://developers.cloudflare.com/r2/), [React Router](https://reactrouter.com/start/framework/installation).
