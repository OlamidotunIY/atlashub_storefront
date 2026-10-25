# Domains, published storefront URLs, checkout and customer security

**Status:** Proposed; separate from creator dashboard and developer tooling.

## Domain and TLS lifecycle

Every storefront may initially have a platform-hosted subdomain. Merchant may connect a custom domain from existing registrar using DNS record instructions in **Storefront Portal**. Cloudflare for SaaS is selected for custom hostname management and certificate issuance/renewal subject to deployment account availability.

State machine candidate: \`REQUESTED -> CHALLENGE_ISSUED -> OWNERSHIP_VERIFIED -> CERTIFICATE_PENDING -> READY -> ACTIVE\`; alternate paths \`FAILED\`, \`SUSPENDED\`, \`REVOKED\`, \`REMOVING\`.

1. Merchant authorized for storefront requests \`shop.example.com\`; backend canonicalizes hostname and checks global uniqueness/ownership.
2. Backend generates DNS TXT/HTTP proof as applicable; registers Cloudflare custom hostname/SSL validation record. Provide merchant explicit CNAME target and status UI.
3. Validate **hostname control** and **certificate** independently; do not go live before both ready. Track renewal, DNS changes, decommission.
4. Bind verified hostname to Storefront publication and active Worker deployment in trusted dispatch mapping. The merchant can choose canonical host, redirects and safe aliases.
5. Resolve unknown/unverified/stale hosts to a generic non-sensitive failure. Never infer tenant from arbitrary Host header, URL path, query org parameter or client-supplied store ID.
6. Ownership recheck and deletion must address domain reclaims, expired registrations, DNS move and customers moving between merchants.
7. Consider apex/root domain behavior, redirects, \`www\` alias, ACME/Caa and provider plan limits. Configure HSTS/TLS, observability and expiry alerts with runbooks.

Reference: https://developers.cloudflare.com/cloudflare-for-platforms/cloudflare-for-saas/start/getting-started/ and https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/configuration/hostname-routing/

## Web serving/SEO

A theme supports canonical routes, SEO metadata and open graph, product JSON-LD, sitemaps/robots (including merchant-defined visibility), accessible image alt semantics, configurable locales and hreflang. Product and collection SSR/pre-render indexes should receive public catalog metadata from Java. Dynamic stale inventory/pricing must not be treated as authoritative. Preview and draft domains use no-index and access controls.

## Shopping cart and trusted checkout

**A theme controls presentation but not financial authority.**

- Browser/cart SDK talks to Java-scoped public Storefront API, using a cart/session token securely bound to verified public storefront context and proper CSRF/abuse controls.
- Product/variant/quantity updates create server-evaluated cart; Java alone calculates price, tax, stock/reservations, shipping routes and eligible merchant-connected provider methods for merchant country/currency.
- \`checkout.start\` creates a trusted, short-lived checkout session/redirect. Theme may customize a safe button and landing experience; it may not inject bank/card collection logic or provider secrets.
- Payment collection uses a trusted Atlas-controlled checkout origin/widget with strict separation from untrusted theme JavaScript. Payment callbacks/redirects are not proof of payment. Java verifies signed provider webhook/server status, idempotency and reconciliation before marking order paid.
- Refunds, chargebacks, fulfillment, accounting and payout states stay Java-owned and originate through server-verified domain events.

Untrusted theme JS may display false UI or attempt phishing. Use marketplace manual review, CSP, form-action/connect-src restrictions, canonical checkout URLs, clear safe browser flow and fraud reporting.

## Provider-capability reality

Merchants connect external eligible payment providers through Atlas Hub integration framework (Paystack/Stripe etc when supported), choose available routing/customer options. The storefront theme does **not** get unrestricted provider credentials, direct refund API or right to claim country/currency capability. Unsupported provider/country operations remain clearly unavailable. Hosted storefront does not itself imply licensed international payment/tax functionality.

## Customer data

Public storefront pages expose only catalog-public view models and published merchant assets. Customer identity/orders/account details use separate trusted backend flows and appropriately isolated browser origin/session; never expose management app cookies or merchant tokens on customer domains. Inventory and actual order data are not copied into theme asset storage or manifest.

## Event/state handling

Authoritative Java publishes outbox-backed events for product price, inventory/collection, order state, payment/refund, entitlements and storefront availability; storefront caches invalidate and page state refreshes as needed. Idempotent event consumer and bounded staleness. Pending/failed/unknown payment events cannot be turned into success by a theme or redirect query param.

## Acceptance

Two businesses with different domains and same theme may never share cart/order/customer/cached data. Verified apex/subdomain DNS + TLS flows, rejected stale/spoofed host, no accidental indexing of private preview, checkout handles retry/out-of-stock and processor webhooks correctly, keyboard/mobile access and SEO structured data validated.
