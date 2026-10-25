# Theme marketplace, creator publication, merchant installation and lifecycle

**Status:** Design contract; not implemented.

## Actor roles

Creator owns source/version/listing rights for their theme; Atlas reviewer approves/rejects public releases; merchant buys/installs/chooses theme for own org/store; trusted billing service owns charges/entitlements, not arbitrary theme JavaScript. Platform staff controls abuse/safety takedowns under audit.

## Canonical data and states

- `CreatorProfile`: AtlasHub user ID, bio, status, available payouts/terms (financial qualification separately).
- `Theme`: canonical identity/slug, author, ownership and general categories.
- `ThemeVersion`: semantic version, immutable source archive reference, lockfile/manifest digest, build ID, compatible SDK/runtime.
- `BuildJob`: QUEUED/RUNNING/FAILED/PASSED, isolated worker logs redacted, outputs and provenance.
- `ReviewCase`: NOT_SUBMITTED/IN_REVIEW/CHANGES_REQUESTED/APPROVED/REJECTED with review evidence.
- `ThemeRelease`: immutable artifact references, signature/hash, capabilities/CSP policy; APPROVED/DEPRECATED/REVOKED.
- `ThemeListing`: DRAFT/LISTED/UNLISTED/REMOVED with price, supported markets/categories and approval link.
- `ThemeLicense`: merchant organization, license/receipt/entitlement, source billing transaction ID, permitted installs, refunds/expiry policy.
- `ThemeInstallation`: merchant storefront, pinned release, installation and target environment.
- `StorefrontPublication`: active release, config revision, deployment ID, verified domain(s), publishedAt, rollback parent.
- `CreatorEarning`: event-based projection from platform-authoritative sales/refunds/fees, not a second independent financial ledger.

## Creator flow

1. Register or sign in as individual via **separate Auth Web** and Java identity, optionally with **no organization**. Create creator profile in **independent Storefront Portal frontend**, using backend creator API. Do not require bank/KYB simply to author a template.
2. Scaffold React theme via CLI, implement route contract/SDK, supply valid default settings and preview media.
3. CLI authenticated submit with unique version/idempotency key; server issues scoped short-lived upload instructions; archive arrives **private**.
4. System quarantines source, validates manifest and dependencies, isolated builds SSR/client assets, malware/supply-chain checks, computes immutable release digest, stores build diagnostics.
5. Creator previews non-indexable candidate and submits for review; reviewer or automated controls approve, request changes or reject.
6. Only approved release/listing becomes visible to marketplace. Sale price, payouts, category taxonomy and account state are platform-enforced.
7. Creator new version creates another immutable review/release. No replacement of bytes already installed on merchants.
8. `unlist` hides theme from new purchases; existing licensed installs generally continue. Security/legal revocation uses explicit incident policy and merchant migration/notification.

## Merchant purchase/install/publish

1. Atlas Web recommendation opens **separate** [Storefront Portal frontend](https://github.com/OlamidotunIY/atlashub_storefront_portal) listing using safe deep link, completing central SSO via [Auth Web](https://github.com/OlamidotunIY/atlashub_auth_web) + Java/IdP, not duplicated login UI.
2. Marketplace displays genuine preview/demo, features, region/SDK compatibility, version, price, creator, update policy.
3. Buy via **trusted Atlas Hub billing** or claim free theme; after server-confirmed entitlement, install approved release.
4. Create configuration revision using developer defaults + validated merchant overrides; upload/choose tenant-owned logo/favicon/banners/fonts etc. Theme creator cannot edit installed merchant settings.
5. Preview draft under isolated no-index URL (not same origin as management app), test real product/price sample and responsive pages.
6. Verify custom domain, TLS and store eligibility; publisher chooses active release + config revision, request deployment.
7. Publishing saga: acquire deployment lease, deploy/verify Worker, atomically update verified host mapping/published pointer, purge/warm CDN caches, emit outcome/audit; failed stage leaves previous live release active or clearly degraded, never fictitiously successful.
8. Theme version upgrade requires explicit authorization; show config compatibility/migrations; save merchant overrides and rollback points.
9. Merchant can change/cancel listing/license and choose alternative theme according to explicit rules; preserve operational orders and catalog data independently.

## Billing and creator revenue

Use AtlasHub Java platform billing service for the **AtlasHub theme marketplace's own purchases**, subscriptions, entitlements and accounting. Merchant business creation is independent of payment-provider setup; provider onboarding/KYC only gates provider-dependent customer checkout. Merchant external Paystack/Stripe connections for **their storefront customer checkouts** are different from payment of themes to AtlasHub. Creator revenue share and payout require a contractual payout policy and provider integration, refunds/chargebacks, tax/KYC rules by country; do not assume automatic self-service creator payouts are legal/implemented. Avoid two sources of truth for revenue.

## Rollbacks and deprecation

- A merchant release/version is pinned until approved update (unless explicit separately consented automatic patch policy).
- Theme version cannot be mutated, only superseded/revoked.
- Storefront config revision is separate and backed up.
- `unlist` is not `delete`; existing installs remain unless abusive/copyright/security takedown.
- Deleting a creator profile does not immediately delete purchased binaries/data or other merchants' legally licensed storefronts; retention/legal/takedown rules documented.
- Merchants may export their own assets/config data according to terms and data protection policy; cannot automatically obtain licensed creator source code.

## Marketplace moderation and ratings

Preview screenshots and demos hosted from immutable release, not user-submitted phishing URLs. Review intellectual property ownership, scripts, trackers, accessibility, quality, missing settings defaults, performance and misleading payment flows. Feedback/rating/report-abuse moderation, takedowns and appeal paths are later product features; require explicit scope in implementation.

## Failure and race invariants

Idempotent publish, purchase, retry and upload; version uniqueness enforced; revoked release cannot be newly installed; cancelled purchase not granted entitlement; domain never binds to two live merchants; no theme upload can execute without completed quarantine checks; no webhook duplicate causes double charging or doubled creator credits.
