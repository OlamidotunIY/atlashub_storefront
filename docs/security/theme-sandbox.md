# Theme platform security and untrusted creator content

**Status:** Design requirements, not implementation.

## Threat model
A malicious theme or domain association could cause XSS, leaked merchant/customer data, forged checkout, malicious external scripts, cross-tenant cache poisoning, unauthorized publication, domain takeover, supply-chain code execution, or storefront phishing.

## First-release policy
Accept only **declarative schemas plus audited components**. Avoid arbitrary author-defined backend JS/Node plugins, filesystem/network permissions, private npm install hooks and unreviewed dynamic imports. Theme assets pass malware/content checks, type/size limits, digest verification and asset host restrictions.

## Rendering
- Escape all dynamic content in its HTML/URL/JS context.
- Forbid arbitrary HTML/script blocks unless separately sandboxed and reviewed.
- Enforce CSP, safe external image/CDN permissions and strict redirect allowlists.
- Treat page data and creator descriptions as untrusted.
- Keep provider secrets, Java server tokens and privileged infrastructure keys out of renderer/template data.
- Never execute arbitrary theme uploads in the shared app server.

## Tenant routing and domains
Normalize hostnames and match only verified domain records; reject unknown Hosts. Verify DNS ownership before activation; prevent stale-domain account takeover; handle TLS renewal/expiry and alias management. Vary page caches by tenant, locale, currency/market and publication version; never cache personalized customer data in shared page caches.

## Permissions and billing
Java Atlas Hub owns merchant identity and entitlements. Theme management validates tenant membership, creator ownership, purchase/license and publisher role for every write. Use maker-checker for privileged/irreversible marketplace actions if required.

## Release governance
Automated validation, isolated preview, manual review for third-party submissions, signed/hashed immutable releases, staged rollout, emergency takedown and one-click rollback. Log publisher identity and release digest. Define vulnerability disclosure and review timelines.

## Future extensibility
If arbitrary creator code is ever supported, place it in isolated capability-restricted execution environments with resource/time/network quotas and separate review/security assessment. Do not let creators run arbitrary Node processes within the multi-tenant renderer.

## Verification
XSS tests, SSRF tests, malicious archive uploads, path traversal, domain claim/takeover, CSRF, cross-tenant requests, cache key variation, untrusted asset imports, license bypass and secret exposure.
