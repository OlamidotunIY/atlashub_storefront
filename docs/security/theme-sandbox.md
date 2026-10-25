# Storefront third-party React security and trust boundaries

**Status:** Mandatory design policies; enforcement to be implemented and tested. This supersedes earlier assumption that themes would be declarative-only. React creators have broad **visual freedom**, not unrestricted system access.

## Adversaries and assets

Untrusted developer source archives/NPM dependencies, malicious review updates, compromised creator token, fraudulently claimed domains, malicious merchant/cross-tenant request, hostile product copy, browser extensions and CSRF/XSS. Protect merchant catalogs not designated public, customer PII/payment details, orders, account identities, business APIs, integration secrets, CDN/cache tenant keys, backend databases, creator IP and signed releases.

## Build time (untrusted supply chain)

- Upload to private quarantine using server-authorized short-lived object-store signed URL scoped to a single draft/version.
- Verify creator identity and ownership, manifest schema/route/settings/defaults and filename/path canonicalization, archive bombs, file type, dependency allowlist/denylists, virus patterns and licensing policy.
- Build in disposable sandbox or isolated ephemeral compute with no privileged host filesystem, no application credentials, no unrestricted network, no access to internal metadata endpoints, strict CPU/RAM/time/disk/file limits and telemetry. npm install scripts may run arbitrary commands: reject/disable or isolate based on reviewed policy.
- Generate artifacts from controlled locked dependencies; deterministic digest/release provenance, server-attested integrity. A build passing scans is not safe without review.
- Human review for public marketplace submission with emergency disable, appeal and audit; automatic changes to listed source/approved release prohibited.

## Server rendering

- Use Cloudflare Workers for Platforms dispatch namespace in **untrusted mode**, one user Worker per merchant installation initially; no privileged per-store service bindings, no shared mutable cache across tenants, no Java admin tokens.
- Trusted dispatch Worker performs verified Host -> active Storefront -> deployment mapping; no input-chosen worker ID and no permissive fallback to another tenant.
- Only vetted public commerce gateway reachable by server render code; implement outbound Worker or comparable egress proxy enforcing URL/domain/HTTP method and response size restrictions where supported. Validate exact Worker bindings available under chosen Cloudflare model before rollout.
- Bound CPU, subrequests, response size, streaming cancellation, data/cache consistency and logs.
- Separate live from preview namespaces/deployments; preview URLs require authorization, no-index and bounded expiry.

## Browser/runtime

**Important:** A browser does not automatically sandbox React JavaScript merely because its SSR runs in an isolated Worker. Browser-executed malicious theme JS can alter the page, read data exposed on that storefront origin and attempt remote transmission.

- Public storefront origin must never contain merchant management cookies, private API tokens, bank data or elevated customer authentication.
- Central trusted checkout and sensitive customer account flows operate on a separate origin / restricted architecture, with secure redirects and server-validated order/cart reference. No custom theme payment-card capture.
- CSP restrictive by default (scripts, connect-src, img-src, frame-src, form-action, base-uri); vetted approved external assets/components only. Review third-party code and optional analytics tags. Prevent malicious HTML, XSS, clickjacking, form hijack and exfiltration where technically enforceable.
- Approved SDK does **not** confer security by itself: Java public storefront APIs must enforce scopes/rate limits, bind tenant identity to verified host/attested context, reject arbitrary org ID, never expose arbitrary internal Java endpoints.
- Keep secrets outside bundles. No arbitrary creator-chosen network calls; where security policy permits external services, require reviewed, declared capabilities. Avoid CORS wildcard for privileged APIs.
- For particularly untrusted extension content, consider cross-origin isolated embeds, with known SEO/navigation/a11y tradeoffs; not the default whole-store renderer.

## Identity and authorization

- Atlas identity central for creator and merchant; OIDC Authorization Code + PKCE or BFF; never share/localStorage-copy JS bearer token between portals, no tokens in redirect URL, rotate session and CSRF protect mutations.
- Creator can publish versions for owned themes, not change merchant configurations. Merchant may only install and publish for authorized org/storefront, not alter developer release.
- Independent creator registration requires no business entity; creator earnings/payments require separate legal and KYC/contract policy as applicable.
- Signed upload/CLI tokens least privilege/expiry/revoke, no long-lived master API key in theme scaffold or committed files.

## R2/storage and domains

- Private source and draft releases; only scanned/approved static assets public through CDN-controlled paths. Tenant asset ACL/object references revalidated; short-lived scoped upload URLs and metadata ownership checks.
- Domains must complete both ownership and certificate validation; verify old domain removals/reclamation, Host spoof protection and negative caching. Dynamic dispatch must not trust arbitrary Host header without authoritative mapping.
- Public images may be served globally, but do not expose private media; purge or expire raw staging uploads and abandoned previews.

## Privacy, commerce and abuse

- Personal data minimization/retention, data deletion and authorized subject requests, EU/Nigerian/other law review as geographies expand; no unsolicited customer tracking through theme bundles.
- Java backend recomputes prices, inventory, tax, delivery and eligible payment methods at checkout; order status changes on server-verified provider events.
- License fraud, creator plagiarism, malicious template takedowns, malware incident response, dependency vulnerability notifications, audit history and customer-facing disclosure procedures need owner and runbook.

## Adversarial test plan

Malicious NPM lifecycle scripts, archive traversal, SSRF to cloud metadata/internal APIs, outbound worker bypass, XSS/DOM XSS, resource exhaustion, streaming abuse, cross-store cache/ref leaks, stolen CLI token, forged installation/license, spoofed Host, wildcard DNS takeover, malicious release rollback, stolen merchant session, external fake payment form and domain/SSL expiry.

Acceptance: a malicious theme cannot read cross-tenant secrets/data, cannot bypass trusted checkout, and cannot silently gain new capabilities after approval. Prove claims with actual tests rather than relying on isolation marketing.
