# Cross-repository APIs, event contracts and ownership

**Status:** Proposed API contracts, not implemented endpoints. Naming and schemas must be aligned with actual Java OpenAPI before code generation.

## Repository ownership

- `atlashub_backend` Java identity, member/organization roles, products/variants, published prices, tax/availability, catalog media IDs, orders/cart, payments/providers, billing/entitlements, inventory, fulfillment.
- `atlashub_storefront` TypeScript management: independent creator identity **extension** (profile linked to Java User), theme releases, portal session, storefront installation/configuration, domain and storage/deploy lifecycle. Calls Java-authorized APIs; own PostgreSQL metadata.
- `atlashub_web` business operations and theme recommendations/deep links; no second implementation of theme management.
- **Storefront Portal**: dedicated UI in storefront monorepo consuming management APIs via BFF/session layer.

## Contract groups (illustrative paths; finalize during implementation)

### Java identity and entitlements for portal
- OIDC Discovery/authorize/token/userinfo/end-session or equivalent conformant identity service: centralized AtlasHub user identity, code/PKCE and session.
- `GET /api/v1/me` resolves current user/allowed organizations; only backend, not URL, asserts membership.
- `GET /api/v1/organizations/{id}/permissions` resolves user-scoped merchant abilities.
- `GET /api/v1/organizations/{id}/entitlements` verifies theme purchase/plan and license usage; the Java billing service remains purchase source of truth.
- `POST /api/v1/platform/theme-purchases` initiates trusted theme purchase; confirms entitlements via verified payment, server event, refund lifecycle. Names provisional.

### Storefront management portal/API
- `GET /v1/themes`, `GET /v1/themes/{id}`: browse curated marketplace listing, filters and pricing view derived from authoritative billing.
- `POST /v1/creators/profile`, `GET /v1/creators/me`: register creator linked to Atlas identity without business registration.
- `POST /v1/creator/themes`, `POST /v1/creator/themes/{id}/uploads`, `POST /v1/creator/themes/{id}/versions`: authorize creator and source upload submission.
- `GET /v1/build-jobs/{id}`, `POST /v1/creator/themes/{id}/submit-review`, `POST /v1/reviewer/cases/{id}/decision`: reviewed version lifecycle.
- `POST /v1/organizations/{orgId}/stores`, `GET /v1/organizations/{orgId}/stores`: create/manage stores with Java organization authorization on every call.
- `POST /v1/stores/{id}/installations`, `POST /v1/stores/{id}/config-drafts`, `POST /v1/stores/{id}/preview`, `POST /v1/stores/{id}/publish`, `POST /v1/stores/{id}/rollback`: merchant-only state transitions.
- `POST /v1/stores/{id}/asset-uploads`, `POST /v1/stores/{id}/assets/{assetId}/finalize`, `DELETE /v1/stores/{id}/assets/{assetId}`: tenant-owned asset security.
- `POST /v1/stores/{id}/domains`, `POST /v1/domains/{id}/verify`, `GET /v1/domains/{id}`: verify DNS, SSL and dispatch status.
- `GET /v1/creators/me/sales`, `GET /v1/creators/me/earnings`: projection from Java billing events plus listing logic.

### Public commerce facade for storefront SDK
- `GET /storefront/v1/context`: return public brand, locale, market, approved theme settings; trusted host association server-side.
- `GET /storefront/v1/products`, `GET /storefront/v1/products/{handle}`, `GET /storefront/v1/collections`, `GET /storefront/v1/collections/{handle}`: published public catalog only, scoped to verified storefront.
- `POST /storefront/v1/carts`, `GET /storefront/v1/carts/{id}`, `POST /storefront/v1/carts/{id}/items`: cart tied to verified storefront, server revalidation and reasonable rate limits.
- `POST /storefront/v1/checkout/sessions`: create trusted checkout session/return URL with authoritative Java price/stock/provider policy, no theme ability to choose arbitrary bank account or override price.

Browser SDK does NOT expose merchant admin APIs, billing/creator payout controls, raw provider keys, private customer records or generic Java endpoint proxy.

## Context/invariants

`AuthenticatedPrincipal`: server-verified user ID, allowed org memberships, scopes, session expiry; `MerchantContext`: selected authorized org/storefront; `PublicStorefrontContext`: dispatch-attested verified hostname, store ID, market/locale, publication version and bounded capabilities. Public browser-provided org/store IDs are never independently trusted; use server-to-server proof from dispatch/gateway or authoritative mapping.

Mutation envelopes: request/correlation ID, idempotency key, actor, resource version/ETag, environment (live/test), timestamp and policy result. Events: stable ID, org/store ID, version and occurredAt; idempotent outbox/inbox consumption. API versions additive and backward-compatible where possible. Use generated Java OpenAPI TypeScript client for contracts, limit theme SDK methods to allowlisted public endpoints.

## Cross-service events

Java -> Storefront: `OrganizationMembershipChanged`, `UserSuspended`, `EntitlementGranted/Revoked`, `ThemePurchaseSettled/Refunded`, `ProductPublished/Updated`, `PriceChanged`, `InventoryAvailabilityChanged`, `CollectionUpdated`; all names provisional. Storefront -> Java: `StorefrontPublished`, `DomainVerified`, `ThemeInstallationChanged`, `CreatorEarningStatementGenerated` as needed. Never duplicate authoritative order/payment posting in storefront.

Sync/async: use HTTPS service APIs for immediate authorization/checkout and outbox-backed events for state propagation/cache invalidation. No direct cross-repository database joins or permission checks implemented in React client alone. Failure/stale entitlement policy must be defined (block new purchase/install where proof unavailable; retain live store with safe degradation according to policy).

## Eventual consistency/transactions

Publishing or purchase across Java and Node is **not** one distributed ACID transaction. Use idempotent sagas, pending state, retries and compensations. Charge success must be confirmed by Java before granting paid license. A domain mapping only becomes active after a healthy Worker deployment and TLS verification.

## Security boundaries

SSO browser tokens only for portal scoped session, OAuth code/PKCE; secure server-to-server audience-scoped identity for management Java calls. No Java service/admin credential reaches theme bundle or untrusted user Worker. Whitelist external theme SDK resources by route, method, scope and rate limit. Staff reviewer actions audited/maker-checker as policy dictates.

## Contract acceptance

Proof with generated TypeScript client and Java test fixture; reject forged org, store, Host and SDK context; cancel/duplicate checkout correctly, reconcile refund/license, schema compatibility and custom domain readiness; public product with localized currency and price source verified.
