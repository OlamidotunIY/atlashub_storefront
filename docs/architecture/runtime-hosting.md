# Storefront rendering, Workers for Platforms, deployment and SEO

**Status:** Preferred deployment architecture; subject to proof of concept. No Cloudflare product deployment has been made.

## Why Vite and React Router

Vite is the creator build/dev system; React Router Framework Mode supplies route definitions, data loading, SSR/prerender strategy and hydration integration. Next.js is **not** selected for scaffold/runtime. However Vite **alone** is not an SSR server: edge-compatible SSR bundling, dependencies and runtime adaptors must be prototyped.

### Routes and rendering selection

| Page | Desired delivery |
|---|---|
| Store homepage | Pre-render where safe or cached SSR, initial HTML and metadata |
| Collection/search pages | Cached SSR for discoverable URLs; dynamic filters client-side |
| Product detail | SSR and cache invalidation for updates, HTML product JSON-LD and canonical metadata |
| Static content/contact | Pre-render or cached SSR |
| Cart | Dynamic client UI backed by trusted server cart/session |
| Authenticated customer profile | Separate trusted surface, not arbitrarily templated if it carries sensitive account data |
| Payment checkout | AtlasHub-controlled separate origin/surface; server revalidates prices, tax, stock, shipping and processor |

SSR does not guarantee SEO by itself: proper canonical URLs, sitemaps, robots, titles, structured data, accessibility, localization and caching matter.

## Workers for Platforms architecture

1. Create **untrusted dispatch namespace** for preview/staging and another for production; no per-merchant namespace.
2. Deploy trusted **dispatch Worker** for public zone, with a typed tenant-domain mapping verified by management backend. Enforce security policy, per-tenant quotas and rate limits, request normalization, per-plan CPU/subrequest budgets and logging.
3. Build and deploy **one isolated user Worker per storefront installation** initially. Each user Worker has approved theme SSR code and a release/config reference. Immutable shared client assets reside on CDN. The same ThemeRelease can be reused across thousands of different user Workers, without duplicating developer source archives.
4. Dispatch resolves exact verified hostname -> \`storefrontId\` -> active published deployment -> \`workerName\` and fetches that Worker with restricted context. Never accept workerName/organizationId from untrusted query or arbitrary header.
5. If supported/needed, use an **outbound Worker** or dedicated controlled public-commerce gateway to restrict untrusted Worker egress (hostname, method, body, size and timeout). Enforce minimal capabilities and no privileged secrets/bindings.
6. User Worker renders SSR using released React Router module + validated store config and read-only public commerce data; returns page HTML and references CDN cacheable assets.
7. Client React hydrates using safe public storefront context and SDK (not a privileged API token). Checkout is trusted/isolated. Track release, store, host and origin in redacted telemetry.
8. Atomic publication swaps mapping only after deployment probe is healthy; previous worker can be retained for fast rollback.

A storefront Worker is not a dedicated VPS. WfP supports many scripts and dispatch calls. **Per-installation Worker** is a provisional isolation unit; benchmark deployment size, service-binding support, egress limitations, billable scripts, updates and responsiveness before declaring final production design.

## Isolation rules and caveats

Cloudflare WfP's **untrusted mode** isolates cache between user Workers and disables \`caches.default\`; consult current Cloudflare docs. Do **not** rely on in-user-Worker Cache API \`caches.default\` for storefront page caching. Perform public caching at **trusted dispatch/CDN layer** with rigorously constructed tenant+host+publication+market+locale keys or use approved external cache through controlled gateway. Test isolation and content variation with two unrelated stores. Never use WfP **trusted mode** for arbitrary third-party code just to access \`request.cf\`.

Server-side compute isolation **does not sandbox customer-browser JavaScript**. A malicious theme may still change DOM, initiate fetches permitted by browser policy or phish. Require vetted scripts, tight CSP, controlled SDK, separate trusted checkout/account surfaces and marketplace review. No privileged cookies on merchant custom domains.

## Static artifacts, Worker deployment and configuration

Build results contain client JS/CSS/fonts/media plus Worker SSR entry, all approved with digest. Keep source archive and server build artifacts private; publish only client-safe hashed assets to CDN. Deployment process installs/version-pins Worker script against \`ThemeRelease\` digest; binds public configuration reference, but no master Java backend keys. Merchant config draft and published revision are separate.

No silent code mutation: publishing new release is immutable; merchant chooses update (or authorized policy), config migration checked and reversible. Worker deployment reports \`QUEUED/DEPLOYING/HEALTHY/FAILED/RETIRED\`; fail closed on missing mapping and roll back to last healthy publication.

## Multi-tenant routing, certificates and regional concerns

Cloudflare for SaaS manages merchant custom hostnames/TLS, domain claims verified server-side. A single dispatch route can handle multiple approved custom hostnames and subdomains. Verify CNAME TXT/proof and certificate readiness before binding. Plan for apex domains, redirects, www/non-www and stale hostname reclaims; do not hardcode host extraction from string splitting.

Data residency: edge compute may occur across multiple regions; document where tenant business/personal data is processed and available legal/regulatory obligations. Do not promise all countries' payment/tax availability merely because host routing works.

## Caching and freshness

Static assets: immutable content-hash URLs with long CDN TTL; changed theme version gets new hash.
Public HTML: cache by verified store, hostname, path, query-safe canonicalization, locale, currency/market and published release/config version. Invalidation on theme publish, store-config publish, relevant Java catalog/product/price availability event; use TTL safety net. Error/missing-page TTL constrained; avoid untrusted host-to-cache poisoning.
Cart/customer-specific response: private/no-store where necessary. Never cache Set-Cookie or authenticated account content in shared CDN.

## Performance, SEO, accessibility

Measure client bundle size, JS execution, hydration costs, p95 TTFB in at least two target geographies, Web Vitals, high-traffic product listings, cache hit rate, Worker CPU/subrequests/memory, broken catalog calls and graceful degradation. SEO: product structured data, OpenGraph/Twitter images, canonical URLs, sitemap/robots, locale/hreflang, proper status codes, indexable public HTML and no-index private previews. Keyboard interaction, contrast, headings and accessible cart throughout.

## Fallback strategy

If React Router SSR runtime cannot deploy into untrusted WfP with required isolation/limits: do not ship insecure shared Node SSR. Re-evaluate SSR framework/runtime, isolated external render services or limited static SPA for non-indexed contexts through a new ADR and benchmark. Preserve Vite/SDK/manifest contracts where possible.

## Official sources

https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/
https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/how-workers-for-platforms-works/
https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/worker-isolation/
https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/configuration/hostname-routing/
https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/pricing/
https://reactrouter.com/
