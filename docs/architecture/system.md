# Atlas Hub Storefront — System Architecture

**Status:** Proposed; no application code yet.
**Stack:** TypeScript monorepo proposed; select renderer via performance/security prototype. Java remains commerce source of truth.

## Boundaries
1. **Java commerce core:** identity, org/outlet context, catalog, pricing, promotions, tax calculations, inventory/reservation, order state, merchant-connected payments, shipping capability routing, billing entitlements.
2. **Theme management API:** author registry, themes, immutable versions, validation, asset storage references, storefront drafts, publication jobs, licenses, verified domains.
3. **Theme engine:** safe, deterministic resolution of page templates, sections/blocks, setting schemas, localization, approved components, public commerce bindings.
4. **Storefront runtime:** shared multi-tenant renderer; resolves verified Host -> Storefront -> PublishedThemeRelease -> Config -> Commerce API -> HTML; supports SSR, cache variation and checkout redirects.
5. **Theme editor:** previews drafts, edits sanctioned settings/content, saves revisions, publishes only with permissions/entitlements.
6. **Creator marketplace:** submission/review, policy checks, version compatibility, licensing, payouts and revocation.

## DDD bounded contexts
`ThemeCatalog`, `ThemeAuthoring`, `ThemeRelease`, `StorefrontPublication`, `DomainProvisioning`, `CreatorMarketplace`, `CommerceIntegration`.

Domain models `Theme`, `ThemeVersion`, `ThemeRelease`, `Storefront`, `DomainBinding`, `ThemeLicense`, `ReviewCase`. Domain rules are independent of HTTP/SQL/React. Application workflows use explicit commands and queries. Infrastructure implements storage, CDN, DNS/TLS provider, background queues, database and Java API adapters. Presentation exposes admin/editor APIs and public rendering.

## Rendering path
- Request Host is canonicalized, checked against a VERIFIED domain registry and resolved to a tenant. Unknown/unverified hosts are rejected; never use unchecked host headers to grant tenant access.
- Select published immutable theme version and store market/locale.
- Resolve the requested page type (`HOME`, `COLLECTION`, `PRODUCT`, `CART`, `CONTENT`), settings/sections and approved data dependencies.
- Request public (tenant/market-safe) product/catalog data from Atlas Hub Java.
- Render SSR/HTML with cache keys including tenant, locale, market, release version and relevant data versions; invalidate on authoritative commerce events.
- Checkout uses Atlas Hub API to recalculate pricing, inventory, tax and available payment/shipping methods. Theme does not determine authoritative payment status.
- Record performance/errors. Promote/rollback publication atomically.

## Why 200+ themes are compatible
Store themes as separate immutable packages/manifests and static assets referencing a shared versioned theme schema and trusted UI primitive library. Tenant storefront configs bind to a release. This avoids 200 Next.js applications or deployments. Creator themes cannot access privileged server runtime.

## Domain management
A merchant can connect an existing domain; automated DNS ownership verification, custom hostname registration and TLS issuance are required before publishing. Domain registration/resale is separate optional registrar integration.

## Creator protection
Initially support declarative manifests plus reviewed components; do not accept arbitrary server-side theme JS. Use sandboxed validation, resource budgets, CSP, external asset controls, moderation and version rollback.

## Proposed monorepo topology (future; not created yet)
```
apps/management-api/
apps/storefront-runtime/
apps/theme-editor/
apps/worker/
packages/theme-schema/
packages/theme-engine/
packages/commerce-api-client/
packages/components/
packages/theme-sdk/
themes/starter/
```
Next.js is one candidate **renderer**, not the whole platform. Alternative: Fastify/NestJS management and a purpose-built Liquid-like safe rendering runtime.

## Source alignment
Java repo: https://github.com/OlamidotunIY/atlashub_backend
Backend design proposal: https://github.com/OlamidotunIY/atlashub_backend/blob/docs/global-platform-ai-storefront-architecture/docs/architecture/storefront-theme-platform-proposal.md
