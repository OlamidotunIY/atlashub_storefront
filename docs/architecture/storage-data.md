# Storage, objects, metadata, tenant assets and business data boundaries

**Status:** Planned design and lifecycle; no R2 buckets or database deployed yet.

## Four distinct storage/data categories

| Category | Logical owner | Technology candidate | Data and access |
|---|---|---|---|
| Creator **source archive** | Storefront platform | Private R2/object storage bucket | Exact submitted project archive + manifest/lockfile; only creator/approved security pipeline, not public or merchant-readable |
| Immutable **approved release artifacts** | Storefront platform | Private R2 origin plus CDN public immutable *approved* assets | Versioned hashed JS/CSS/fonts/images, SSR module deployment package, release manifest/checksum; one release reused by many licensed installs |
| **Merchant storefront assets and theme overrides** | Merchant organization/storefront | Tenant-scoped R2 objects, CDN, PostgreSQL metadata + configuration revisions | Merchant logo/favicon/banners/fonts/marketing media/brand colors/copy, only when merchant explicitly configures; no shared creator write permission |
| **Live business data** | Atlas Hub Java commerce/financial modules | Existing Java-backed OLTP/data services | Products/variants/prices/inventory/outlet stock/orders/checkout/customers/payout states, referenced by scoped storefront SDK, **not replicated as authoritative storefront DB tables** |

Public product images and files belong to Java Commerce/Catalog's asset ownership even if backed by the same physical storage provider. Storefront may receive authorized public media URLs/IDs, but not assume ownership or deletion rights.

## Developer upload/build/release object lifecycle

1. Creator is authorized for Theme ID/version; platform reserves source submission ID + size/hash and creates **short-lived single-prefix signed upload URL** for a private quarantine bucket.
2. CLI sends **source archive** excluding \`node_modules\`, secrets, \`.env\`, local keys and generated outputs (policy rejects accidental secrets), plus lockfile/manifest. File format size and decompression bounds enforced.
3. Builder validates file/metadata ownership, archive traversal and bombs; isolated reproducible build (no privileged network/credentials); test SSR Worker compatibility; outputs separate **client static assets** and **server SSR module**.
4. Approved build stored in immutable release object prefix keyed by Theme ID/version/content digest. Object/version metadata includes checksum, source provenance, SDK/SSR compatibility and reviewed capabilities. Never publish arbitrary prebuilt upload without rebuild/verification.
5. Release static assets are served under stable content-addressed URLs through CDN. SSR code is deployed to untrusted Workers for Platforms Worker for each installation, not executed directly out of R2 by portal.
6. Unapproved releases, source archives and private source maps stay private. Deprecated releases remain available while licensed stores depend on them, subject to legal/security emergency policy.
7. Garbage collect only unreferenced, past-retention drafts/builds/orphaned release files; never delete assets used by live publications merely because listing is unlisted.

Illustrative bucket/prefix layout:

\`\`\`text
r2://theme-source-private/creators/{creatorId}/themes/{themeId}/submissions/{submissionId}/source.tar.gz
r2://theme-build-private/jobs/{jobId}/logs.json
r2://theme-releases-private/themes/{themeId}/{version}/{digest}/server/entry.mjs
r2://theme-releases-private/themes/{themeId}/{version}/{digest}/client/assets/index-{hash}.js
r2://merchant-assets/organizations/{orgId}/storefronts/{storefrontId}/branding/{assetId}/{digest}.webp
r2://merchant-assets/organizations/{orgId}/storefronts/{storefrontId}/banners/{assetId}/{digest}.webp
\`\`\`

Object *prefixes* are organizational conventions, not authorization controls. Every upload, get, delete and mutation must validate tenant ownership against server-side asset metadata. Public CDN access should expose only transformed/approved objects; never expose signed internal source paths. Use custom CDN domain, cache keys and origin access rules.

## Relational metadata (PostgreSQL proposed)

Entity candidates:
- \`CreatorProfile(id,userId,status)\`
- \`Theme(id,creatorId,slug,status)\`
- \`ThemeVersion(id,themeId,semver,submissionId,schemaVersion,sourceKey,sourceDigest,reviewId)\`
- \`BuildJob(id,versionId,status,logsKey,attemptCount,createdAt)\`
- \`ThemeRelease(id,versionId,assetManifestKey,artifactDigest,clientPrefix,workerArtifactRef,approvedAt)\`
- \`ThemeListing(id,themeId,activeVersionId,price,billingProductRef,listed)\`
- \`ThemeLicense(id,organizationId,themeId,billingEntitlementRef,status)\`
- \`ThemeInstallation(id,storefrontId,releaseId,licenseId,status)\`
- \`Storefront(id,organizationId,market,locale,status,activePublicationId)\`
- \`StorefrontConfigRevision(id,storefrontId,releaseId,schemaVersion,settingsJSON,status)\`
- \`DomainBinding(id,storefrontId,canonicalHost,ownershipProofStatus,sslStatus,active)\`
- \`Asset(id,ownerOrganizationId,storefrontId,key,digest,mime,size,visibility,status,deletedAt)\`
- \`WorkerDeployment(id,storefrontId,releaseId,workerName,environment,status,artifactDigest)\`
- \`Publication(id,storefrontId,releaseId,configRevisionId,deploymentId,previousPublicationId,status)\`
- \`AuditEvent(id,actorId,orgId,objectId,action,requestId,timestamp)\`

Identifiers and unique keys are illustrative; migrations, indexes, schema and isolation model must be designed and reviewed. PostgreSQL is **storefront platform metadata only**, not a second copy of Java authoritative business state. Feature read models may cache non-sensitive public catalog projections with version/expiry/invalidation.

## Tenant asset upload workflow

1. Authorized merchant opens Storefront Portal's settings/asset library for an explicitly selected organization/storefront.
2. Management backend verifies Java membership/publisher authorization and storage quota; creates \`UploadSession\` with max bytes, MIME and allowed object prefix, and issues short-lived signed upload authorization.
3. Browser uploads into a quarantine/staging prefix directly to object storage. Server validates actual content type, file signatures, SVG safety, malware, resolution and quotas; converts images into optimized variants in a sandbox.
4. Create Asset metadata and approved public variants; merchant settings reference Asset **ID**, not arbitrary raw object key/URL.
5. Merchant picks logo/favicons/banner etc; a new DRAFT config revision is saved. Preview references the draft; publish selects a known validated config revision atomically.
6. CDN URLs for public images may remain stable versioned/hashed paths; old configuration asset references remain for rollback until no live/retained config depends on them.
7. Deletion: remove reference from draft/published revisions through explicit workflow; garbage collect only when unreferenced and retention expires, with backups/log/legal holds as needed.

Different merchants can use the **same developer default asset** stored with theme release, but cannot access another merchant's override assets without explicit publication. Assets intended to be private must never receive public CDN URLs.

## Default values versus overrides

A theme's manifest supplies defaults for logo, color, hero title, example imagery, featured products fallback behavior etc. Defaults are part of the immutable theme release. Merchants' validated overrides and asset references live in StorefrontConfigRevision.

Published config = defaults of pinned ThemeRelease + merchant override JSON + authorized asset URL resolver, under a fixed schemaVersion. A changed default in future release cannot silently replace merchant override. No developer can mutate merchant config simply by changing source release.

For merchant-specific references such as collection IDs, use optional null default with safe rendering fallback or configurable default collection strategy; do not hardcode another merchant's product IDs.

## Commerce data access, caching and PII

- Storefront request verified hostname -> attested StorefrontContext -> Java public-commerce gateway -> public view models. No theme can pick arbitrary organization ID.
- SDK exposes published catalog, variant, availability, published price and media, plus cart/checkout initiation through scoped API. Inventory/order state and payment ledger live in Java.
- Public product/collection data can be cached by org/storefront/market/locale/currency/catalog version, with TTL and Java events to invalidate. Never mix personalized carts/customers or auth tokens into shared public edge cache.
- Java verifies price, tax, shipping/payment method eligibility, inventory reservation and final payment when customer checks out; cached theme page is not a financial authority.
- Respect data minimization, PII export/deletion/retention, creator IP licensing, region legal needs and cross-border hosting review.

## Cost and lifecycle policies

Account for source archive retention, immutable releases, installed but unlisted releases, upload/build temporary data, merchant media renditions, Class A/B R2 operations, CDN cache misses and old releases. Apply quotas per developer, merchant, plan and file type. Deduplicate hashed immutable assets only when safe and IP policy permits; don't merge unrelated merchant private assets by digest without authorization.

R2 reference: https://developers.cloudflare.com/r2/pricing/
CDN caching reference: https://developers.cloudflare.com/cache/interaction-cloudflare-products/r2/
