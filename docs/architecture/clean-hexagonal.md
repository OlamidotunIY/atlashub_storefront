# Storefront Management/Hosting — DDD, Clean and Hexagonal Architecture

**Status:** Normative future source architecture. This repo is a standalone **Node.js/TypeScript Storefront backend and hosting platform**, not a frontend monorepo, Auth UI or SDK CLI repository. No runtime code is implemented yet.

## Business bounded contexts
- `CreatorRegistry`: CreatorProfile (linked to Java User), theme ownership, creator listing rights and sanctions.
- `ThemeCatalog`: Theme, ThemeVersion, immutable ThemeRelease, compatibility and listing metadata.
- `ThemeSubmissions`: SourceArchive, BuildJob, validation findings, review case and approval; creator upload does **not** equal marketplace availability.
- `ThemeMarketplace`: listing eligibility and presentation; Java platform billing is authority for paid theme license and refunds.
- `ThemeInstallations`: per-merchant entitlement reference, pinned approved release, compatibility and upgrades.
- `StorefrontConfiguration`: Storefront, DraftConfigRevision, PublishedConfig, validated developer defaults, merchant overrides.
- `AssetLibrary`: merchant-owned uploads/media and release assets, ownership/retention and visibility.
- `DomainProvisioning`: DomainBinding, verified hostname ownership, TLS state and handover.
- `StorefrontPublication`: Publication, WorkerDeployment, atomic activation/rollback and recovery.
- `CommerceFacade`: authorized published/catalog read views and trusted cart/checkout gateway to **Java**; not independent price/stock/order truth.
- `Observability/Usage`: meter quotas, rendering/build cost and incident evidence.

## Clean/hexagonal dependency direction
```text
HTTP Controllers / Job Consumers / Cloudflare Adapters
             |                 |
             v                 v
        Application use cases + inbound ports
                     |
                     v
       Domain aggregates, policies, events
                     ^
                     |
   Interfaces (outbound ports defined inward)
                     ^
                     |
 PostgreSQL / R2 / Cloudflare / Java / Queue adapters
```
Infra adapters implement application/domain-defined outbound ports, never imported by domain. Application orchestrates transactions/retries/authorization checks through ports. Domain models are pure TS without Express/Fastify/Nest, Prisma/TypeORM, Cloudflare Workers, fetch/Node process, R2 or queue package dependencies.

## Ports and adapters examples
- `ThemeRepository`, `ThemeReleaseRepository`, `StorefrontRepository`, `PublicationRepository`, `UnitOfWork`; PostgreSQL adapters.
- `ObjectStoragePort`: quarantined source, content-addressed approved releases and merchant files; R2 adapter.
- `BuildSubmissionPort`: isolated build/canary/scan; disposable worker adapter.
- `DeploymentPort` and `DomainProviderPort`: Workers for Platforms, Cloudflare for SaaS, verified SSL/host routing; Cloudflare adapters.
- `MerchantAuthorizationPort`, `PlatformBillingEntitlementPort`, `PublicCommercePort`: Java/IdP adapters; never independently fabricate permissions, licenses, prices or provider statuses.
- `Clock`, `IdGenerator`, `EventPublisher`, `Telemetry`: deterministic tests and outbox-driven events.
- Inbound adapters: management HTTP/BFF API, trusted dispatch Worker, jobs/listeners, creator upload/CLI API.

## DDD invariants
Creator owns Theme and may submit new immutable versions; only approved releases may be offered. Licensed merchant owns Storefront config and asset references, **not developer source**. A publication points to an immutable release, validated settings revision, verified hostname and healthy isolated Worker. A merchant's installed version is not changed silently. Portal user and org permissions are Java-issued; management API revalidates on every mutation. Build+deploy jobs use idempotency/saga states and compensation; no distributed database transaction with Java or Cloudflare.

Payment-provider direct merchant settlement and optional provider connection are Java responsibilities, not Storefront-held funds. User signup is independent from organization creation; creator may have no organization. **Retire mandatory Anchor banking/KYB** as Java legacy migration, while provider-specific requirements remain for payment capabilities.

## CQRS and events
Use CQRS where writes are complex and auditable: `SubmitTheme`, `ReviewTheme`, `InstallTheme`, `PublishStorefront`, `VerifyDomain`, `RollbackPublication`; read projections for theme discovery, creator dashboard and deployment status. Do not apply a separate command handler to every cache lookup. Outbox/inbox, retry-safe idempotency, deterministic status and audit. No HTTP handler should directly mutate PostgreSQL without passing application policy and authorization.

## Isolation boundary
Third-party React code is **untrusted**: build isolated; production SSR in Workers for Platforms **untrusted mode**, initially one Worker per storefront installation and trusted dispatch routing. Browser script can still act maliciously; CSP/review/trusted checkout are mandatory. Treat provider-specific Cloudflare ability/limits as PoC gate, not assumed production fact.
