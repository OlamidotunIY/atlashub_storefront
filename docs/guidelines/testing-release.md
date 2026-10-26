# Storefront Management/Hosting — testing and release engineering

**Status:** CI blueprint. No application code deployed.

## Core test pyramid
- **Domain unit:** Theme/ThemeVersion immutable, required settings defaults, creator ownership, listing approved state, config revisions, license eligibility, domain uniqueness, publication state transitions.
- **Application unit (mocked ports):** authenticated store creation, upload, review, theme purchase confirmation, install, asset allocation, domain verification and safe atomic publish/rollback.
- **Adapter/integration:** PostgreSQL transaction/outbox, R2 scoped uploads/quarantine/object lifecycle, Java IAM + platform billing + public commerce ports, Cloudflare API dispatch/Worker provisioning and custom hostname/TLS, queue retry/idempotency.
- **Contract:** API compatibility with separate Portal and SDK/CLI; Java public storefront and identity/entitlement contracts, webhooks/event schemas, version migrations.
- **E2E:** two unrelated orgs and two authored React themes (including shared release on different orgs), SSO from existing Atlas Web to independent portal, theme license/install, settings/logo, custom DNS/TLS, SSR with SEO HTML, cart to Java trusted checkout, release rollback, creator unlist.
- **Adversarial:** malicious dependency/install script, archive bombs/traversal, arbitrary Worker SSR network, browser script exfiltration, cross-tenant Host/cache/assets/checkout, fake license, revoked role, reused signed upload URLs, compromised creator, unverified custom domain, domain takeover, failed deploy.
- **Operations:** performance under simulated 200 theme designs/5000 installations; Worker script/CPU/subrequests cost, CDN hit rate, R2 reads/writes, build duration/size, Web Vitals, route certificates/expiration.

## CI/CD
Lint/typecheck/unit/contract, security and dependency scan, SQL migration test, isolated build smoke test with fake providers and staging Cloudflare features. CI secrets only available to trusted deploy jobs (not creator builds). Audit release provenance and code hashes, exact migration version, atomic rollback controls, feature flags for region/merchant plan.

## Go/no-go for Cloudflare runtime
Workers for Platforms untrusted isolation and actual React Router SSR/prerender compatible build **demonstrated**, independent store cache/egress, public SEO HTML, safe 2-org checkout, measured request/CPU/unit costs and domain lifecycle; otherwise choose a safe alternative via ADR. Never silently substitute privileged shared Node runtime.

## Release and definition of done
Every module has business invariant tests, dependency architecture check, integration fixture, error and replay behavior, tenant authorization test, user-visible state semantics, telemetry/runbook, docs and safe DB migration. No feature marked shipped from documentation or fake adapter alone.
