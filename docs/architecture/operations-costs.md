# Operations, costs, quotas, deployment reliability and quality targets

**Status:** Operations plan, not provider subscription or spend authorization. Pricing facts must be rechecked at provisioning.

## Production environments

- Independently deployed projects: `atlashub_auth_web` central identity UI, `atlashub_web` operations, `atlashub_storefront_portal` merchant/creator frontend, `atlashub_storefront` API/build/Workers backend, Java identity/commerce and a separately planned `atlashub_theme_sdk`. No Turborepo/Nx. Independent `dev`, `staging` and `production` storage, database, Worker namespaces, secret stores and IdP client IDs.
- CI builds portal and management services from reviewed code; creator uploads build **only** in quarantined ephemeral workers.
- Promotion requires immutable source/build hash and signed release; production worker scripts deployed from approved artifacts, not raw user upload.
- Feature flags for region, merchant plan and rollout; staging custom domains and previews must not index or leak production customer data.
- Backups: metadata PostgreSQL PITR, versioned R2 objects/retention, domain/worker mapping export, immutable audit record and disaster recovery runbook.
- Deploy rollback: previous ThemeRelease + StorefrontConfigRevision + active Worker deployment and dispatch mapping can be restored atomically.

## Failure domains and expectations

- **Java unavailable:** public cached catalog may remain within bounded TTL, but do not accept final checkout/order/payment claim without Java confirmation. Central identity and IAM availability also affects new Portal sign-ins/permissions; existing verified sessions must follow defined expiry/revocation policy.
- **Storefront management API unavailable:** active published stores may continue via existing mappings/CDN; prevent new installs/config publishes.
- **Cloudflare dispatch unavailable:** serve controlled 5xx, observe all stores; do not silently route to wrong merchant.
- **Tenant Worker faulty:** isolate store, alert, roll back previous release, avoid cross-tenant blast radius.
- **R2/CDN issue:** immutable assets may be cached; preserve object/version backup strategy and outage messages.
- **Domain validation failed:** remain pending, do not bind traffic or claim TLS live.
- **Source build queue failed:** retain upload and retry bounded idempotently; report creator review/build status accurately.

## Example service indicators (targets must be calibrated)

- p95 SSR time-to-first-byte by geography, cache hits vs SSR misses.
- Core Web Vitals, JS bundle size, HTML validity, search indexing outcomes.
- Worker CPU milliseconds/request, subrequests, memory, script count, errors per release/store.
- CDN hit ratio, Java catalog API latency/availability and invalidation lag.
- DNS/TLS activation time, cert renewal status, domain routing misses.
- Publish queue latency, deployment success, rollback duration, build worker scan failures.
- Cross-tenant security events, customer checkout abandon/failures, payment verification discrepancy.

Avoid invented guaranteed latency targets; set SLOs with benchmark telemetry and business requirements.

## Cloudflare pricing (reference as of docs checked 2026-10)

Official Workers for Platforms pricing **currently** states $25/month plan, includes 20M requests/month, 60M CPU milliseconds/month, 1000 scripts and usage-priced overages. Published overage examples: $0.30/extra million requests; $0.02/extra million CPU milliseconds; $0.02/extra script. Check exact current plan/pricing/terms before budget approval:
https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/pricing/

**Not free**, and per-installation Worker deployments contribute to script usage. Example rough script count 5,000 merchant storefronts -> ~5,000 tenant scripts (plus dispatch/staging), not 5,000 always-on servers. This is not cost estimate without request/CPU/cache and account plan data. CPU/subrequest limits may make some themes incompatible even with 'unlimited' namespaces.

R2 pricing (official snapshot 2026-10) differentiates object GB-month, Class A write/list operations, Class B read operations and potentially infrequent-access retrieval. Standard class published reference $0.015/GB-month, Class A $4.50/million, Class B $0.36/million, zero R2 egress bandwidth subject to terms. Recheck:
https://developers.cloudflare.com/r2/pricing/

Additional potential costs: CDN/Workers edge traffic beyond included, custom domain/SSL-for-SaaS plan/hostname charges, external PostgreSQL, build compute isolation, source/security scans, observability, support, legal/compliance, payment processing, theme refunds/revenue shares. Provider rates and billing units may change.

## Unit economics and rate limits

Track cost dimensions by store/plan, theme release, creator, month, traffic country and environment. Count Worker request/CPU/subrequest, R2 GB/object request, CDN miss, metadata DB storage/query, build CPU/storage, domain/cert, logs and commerce API throughput. Charge/suspend abusive source submissions and quota-heavy rendering; merchant bill plan should cover true marginal costs.

Set enforceable per-store request rate, CPU, subrequest, build duration/size, assets size, SDK calls, preview TTL, domain count and source archive count. Blocks should not rely only on JavaScript portal UI. Quotas/denial-of-wallet safeguards in trusted dispatch.

## Observability and privacy

Correlate portal action -> backend command -> build/review -> worker publish -> HTTP dispatch -> Java storefront API with tenant-safe IDs, not PII or secrets. Redact customer personal data; do not collect entire checkout inputs/theme source logs by default. Alert on egress violations, suspicious theme submissions, cross-tenant mapping attempts, 5xx spikes, abusive worker loops and certificate expiration.

## Developer and merchant support procedures

Developer build failures show reproducible safe diagnostics, denied dependencies/capabilities, line numbers where possible, SDK compatibility warnings; merchants get draft/active health, domain progress, expected DNS records, update and rollback choice, listing/entitlement state, and communication for takedown. Support agents require audited scoped assistance, never blanket creator access to merchant data.

## Planned load-test matrix

- 2 merchants / 2 themes PoC including same theme on two stores, different currency and verified host.
- 200 themes / 5,000 installations simulated provisioning and mapping scale with variable traffic, not assertion of existing traffic.
- Product surge, cache invalidation storms, SSR cold starts, high-size JS, malicious CPU spike and CDN outage.
- Domain churn/reclaim and Worker deployment failures while previous merchant store remains live.
- Checkout/cart protection during stale price, inventory changes, provider disabled, webhook duplication.
