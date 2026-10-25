# Storefront Platform — Module-by-module implementation plan

**Status:** Proposed. No module below is implemented merely because it is documented.

| Bounded context/module | Responsibilities | Acceptance criteria |
|---|---|---|
| `theme-schema` | Manifest, page/section/block schemas, version compatibility, locale/settings validation | Strict validations; no executable untrusted server code |
| `theme-catalog` | Discover approved themes, versions, metadata, previews, publishers | Tenant-independent curated catalog and consistent pagination |
| `theme-authoring` | Drafts, preview, versioned edits, assets, creator permissions | Preview isolation and durable history |
| `theme-engine` | Server rendering AST/config evaluation and safe commerce data bindings | Escaping, schema checks, deterministic rendering, performance limits |
| `storefront-runtime` | Verified domain routing, SSR, edge caching, SEO and public pages | Two stores served with zero tenant leakage; market-specific caching |
| `storefront-publication` | Immutable releases, publish rollback, invalidation and audit | Atomic publication and reversible deploy |
| `domain-provisioning` | DNS challenge, TXT/CNAME verification, TLS, aliases, redirects | No domain takeover/host header confusion |
| `commerce-client` | Typed Java catalog/product/cart/order and checkout API | Versioned auth, no direct database access, error isolation |
| `theme-editor` | Drag/drop or section editor, theme settings and preview | Accessible editing, drafts and approval flow |
| `asset-pipeline` | Upload/storage, resize/optimization, signatures, malware checks | Limits, content validation and CDN integration |
| `licenses` | Merchant licenses, purchases and entitlement resolution | Paid themes not publishable without verified entitlement |
| `creator-marketplace` | Creator onboarding, submissions, reviews, releases and payouts | Abuse prevention, policy controls, accounting/billing integration |
| `observability` | Multi-tenant metrics, rendering traces, event ingestion | Domain/page health, performance budgets and alerts |

## Phase 1 — merchant storefront core
1. Define Java public commerce contract: store configuration, products/variants, pricing per market, availability, cart, checkout and order status.
2. Define signed immutable theme manifest and approved component schemas.
3. Build tenant hostname resolution and render one internal starter theme.
4. Add verified custom domain, HTTPS, cache invalidation, SEO and merchant publication flow.
5. Test two tenants with different catalog/price/currency/domain and no cross-tenant leaks.

## Phase 2 — theme platform
1. Theme catalog, preview, draft settings, assets and theming editor.
2. Theme releases, schema compatibility, atomic publication and rollback.
3. Expand curated theme variants while retaining one shared rendering contract.
4. Add market/localization/accessibility and catalog discovery APIs.

## Phase 3 — creator marketplace
1. Author credentials/creator identities and publishing rights.
2. Third-party theme submission and comprehensive sandbox/review workflow.
3. Paid licensing, entitlements, creator revenue reporting and refunds.
4. Upgrade/migration guarantees, reporting and takedown policies.

## Test checklist
Contract tests Java vs TypeScript; host routing, domain takeover prevention, SSR escaping/XSS, authorized publication, paid license checks, audit/rollback, price+inventory authority, cross-border locale/currency, accessibility, performance, origin failover and resource isolation.

## Deferred
One build/server per merchant, merchant-uploaded arbitrary backend JS, unrestricted custom runtime extensions, creator payment processing outside approved billing providers, country availability promises without local eligibility.
