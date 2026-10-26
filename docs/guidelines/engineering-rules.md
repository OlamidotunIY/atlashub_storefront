# Storefront Backend/Hosting — engineering, DDD and security rules

**Normative for future code.** Source pending.

## Code conventions
- TypeScript strict, validated environment/inputs, explicit error types, structured logging and request correlation. Node LTS and current Cloudflare Worker-supported runtime determined via compatibility tests; server-only Node libraries must not leak into edge bundles.
- Domain entities/value objects may raise domain events but import no DB, ORM, HTTP, React, queue, cloud SDK, logging implementation or framework exceptions. Application commands/queries use ports and policies; adapters translate external errors. Service DI only at composition/bootstrap.
- Use bounded context module public entrypoints; no cross-context direct ORM joins or business logic in controllers. Every mutation requires tenant/creator IAM and entitlement verification as applicable, DB transaction and idempotency.
- Use PostgreSQL for theme/install/storefront metadata; use R2/source/archive and merchant assets; Java remains financial/catalog authority. Do not duplicate authoritative prices, stock, orders, provider status or billing.

## Publishing lifecycle
Upload candidate archive through single-purpose short-lived signed URL into private quarantine; validate ownership, size, malware, traversal and manifest/defaults. Build locked dependency tree in disposable restricted compute with **no privileged production secrets**, audit/review/attest, generate immutable signed release. Only approved release can be installed. Merchant preview/draft separate from public; publication updates healthy Worker+verified domain mapping atomically. Rollback is a supported command; unlisting not arbitrary shutdown of licensed merchant stores.

## Runtime trust and commerce
- Workers for Platforms **untrusted mode** initially per-store installation; trusted dispatcher for verified hostname and exact tenant mapping, bounded CPU/request/egress, Cloudflare plan limits measured.
- Server SSR isolation does not protect browser from malicious theme JS; CSP, restrictions and separate trusted checkout/customer account origins remain mandatory.
- Customer public SDK no privileged Java bearer token, payment processor secret, HR/ledger/settlement, or generic proxy to Java administrative paths.
- Java checkout recomputes price/stock/tax/provider eligibility. Provider onboarding KYB when required is merchant provider-specific, not Atlas-operated compulsory Anchor banking.
- Cloudflare credentials/DB/R2 keys are back-end only; staging/prod separated, rotate keys, never log secrets.

## Identity and permissions
Atlas Hub global User can sign in without Organization; creator account references User, while store/merchant changes require verified selected Organization IAM roles. Central Auth Web is separate UI; Java/IdP owns OIDC/SSO; Storefront Portal is independent frontend. Service tokens short-lived/audience-bound. No URL-provided org/store IDs as trust authority. Staff review privileges separate from creator and merchant.

## Reliability
Outbox/inbox for cross-service state changes, idempotent uploads/publish/refunds/events, explicit `PENDING/FAILED/UNKNOWN` no fictitious success, timeouts and backoff. No distributed ACID assumption across Java/R2/Cloudflare. Audit all version, domain, review, permission and active mapping mutations; rollback runbook and data retention before deletion.

## Pull requests
Document changed bounded context/invariants, ports+adapters, migration/data impact, contract/API compatibility, tenancy/security and Cloudflare resource cost; tests and docs required. No Turborepo/Nx or duplicate frontend auth/source.
