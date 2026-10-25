# Atlas Hub Storefront Platform

**Status:** Architecture selected for prototyping; documentation only. No portal, hosting runtime, SDK, CLI or marketplace is deployed.

The dedicated Atlas Hub Storefront platform lets independent creators build **complete React storefront themes** with **Vite + React + TypeScript + React Router Framework Mode**, publish reviewed immutable releases using an **Atlas Hub CLI**, and offer themes through a marketplace. Merchants and creators use a **separate shared Storefront Portal** with **central Atlas Hub single sign-on**. Merchants configure logos, colors, content, domain, theme installations and storefront publishing **there**, while the existing Atlas Web business dashboard supplies recommendations and deep links.

## Repositories: no fourth backend required

| Repository | Ownership |
|---|---|
| [atlashub_backend](https://github.com/OlamidotunIY/atlashub_backend) | Existing Java business system of record, IAM/auth, commerce, orders, inventory, checkout, billing and payment/shipping capability policy |
| [atlashub_web](https://github.com/OlamidotunIY/atlashub_web) | Existing merchant operations dashboard; storefront recommendations and SSO deep links, **not** duplicate template management |
| **atlashub_storefront (this repository)** | Storefront Portal UI, independent TypeScript management backend, SDK, CLI, theme build and review pipeline, tenant rendering runtime and hosting integrations |

This is a **TypeScript monorepo**, not a single Next.js app and not a server per theme. Each service can deploy independently without separate GitHub repositories.

## Decisions and guardrails

- **Theme creation:** Vite + React + TypeScript; React Router Framework Mode for SSR/pre-rendering. Creator owns layout, CSS, animation and approved client-side interactivity; Atlas supplies typed route contracts, storefront SDK and theme manifest/settings schema.
- **Merchant customization:** Developer must define valid defaults for configurable settings (logo, colors, banners, fonts, homepage copy, featured collections etc). Merchant overrides are tenant/storefront-scoped, validated and retained on upgrades. No drag-and-drop editor in initial scope.
- **Dashboard:** Separate Storefront Portal with merchant and creator role-aware workspaces. A user may hold both. Atlas Web may display theme suggestions that deep-link into portal; the portal owns all storefront/marketplace management.
- **Authentication:** central Atlas Hub identity, OIDC Authorization Code + PKCE or secure BFF; existing frontend login must be extended to support SSO. Same identity, independent app sessions and authorization; never redirect with tokens.
- **SDK:** approved public storefront data access (catalog, collections, product details, cart, checkout initiation). No admin/HR/ledger access or merchant/provider secrets. Data sourced through Java-authoritative backend.
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

## Proposed future monorepo layout (not implemented yet)

```text
apps/
  portal/                 # Shared merchant + creator UI
  management-api/         # Independent Node/TypeScript DDD backend
  storefront-runtime/     # Cloudflare dispatch + isolated tenant rendering
  build-worker/           # Disposable isolated source builds
  publishing-worker/      # Review/release/domain/cache jobs
packages/
  create-theme/           # Scaffold generator
  theme-cli/              # Local command-line tool
  storefront-sdk/         # Approved typed client/React hooks
  theme-contracts/        # Manifest, route/settings/schema versions
  theme-runtime/          # SSR/runtime integration
  shared-ui/              # Portal UI only (NOT privileged theme API)
examples/
  starter-theme/
docs/
```

No repository folder above implies the application has already been built. We first implement and evaluate an end-to-end proof of concept with two independently authored themes and two unrelated merchants.

## Reference services (architecture, not deployment claims)

[Workers for Platforms](https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/), [worker isolation](https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/worker-isolation/), [Cloudflare for SaaS](https://developers.cloudflare.com/cloudflare-for-platforms/cloudflare-for-saas/start/getting-started/), [R2](https://developers.cloudflare.com/r2/), [React Router](https://reactrouter.com/start/framework/installation).
