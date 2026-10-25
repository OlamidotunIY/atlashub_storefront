# ADR 0001 — Vite/React creator SDK and isolated Workers storefront hosting

**Status:** Accepted as architecture direction; **conditional on prototype security, compatibility, cost and performance approval**.
**Date:** 2026-10-25.

## Context

Third-party creators must be able to develop real React storefront applications with custom CSS, layouts, animations and browser interactions; merchants can safely customize schema-defined settings without coding. The catalog can hold hundreds of themes and serve thousands of merchants. Search-visible product pages and excellent first-load performance are essential; arbitrary user code must not receive privileged access.

Earlier documentation proposed strictly declarative/Liquid-like themes and considered Next.js as general runtime. That is **superseded** by this React SDK/CLI direction. No initial drag-and-drop editor.

## Decision

1. Create-theme scaffolding: **Vite + React + TypeScript + React Router Framework Mode**, required page contract and standardized Atlas Hub SDK/mock adapters. Independent creators own theme visual behavior.
2. React Router supports SSR and pre-rendering. Choose **SSR/prerender for public discoverable pages**, cache appropriately; personalized cart and checkout are separate browser/Java trusted flows.
3. The initial hosting design: **Cloudflare Workers for Platforms**, **untrusted-mode** dispatch namespace; **one isolated user Worker per merchant storefront installation** with an approved theme release, trusted dynamic dispatch Worker for verified hostname->store resolution, authorization and per-store resource limits. **Do not use the default shared trusted namespace** for untrusted code.
4. Deploy immutable approved theme static assets via **R2 + CDN**. Keep uploaded source private, build in isolated ephemeral workers, enforce signed release manifest and validated merchant config.
5. One storefront deployment != one dedicated VM, and one template design != one project per merchant. Underlying immutable theme assets are reused.
6. Next.js is **not required**. We should not freeze a specialized template language or custom Node interpreter for MVP. React Router/Vite may be replaced only after a documented benchmark/ADR.
7. Third-party theme server and browser code remains risky: no privileged service bindings, no secrets in bundles, restricted outbound fetch, CSP, reviewed dependencies, separate trusted checkout/customer account.

## Alternatives

- **Next.js per template:** strong integrated SSR but restrictive hosting assumptions, more coupling and per-theme app complexity. Not selected for creator scaffold.
- **Custom Liquid-like renderer:** much easier restriction of theme expressiveness and review, but sacrifices complete React customization. Revisit only for a separate safer theme format.
- **Vite SPA only:** easy to host but less ideal for SEO/first HTML performance; keep as fallback for nonindexable or low-complexity use cases.
- **Shared privileged Node SSR:** rejected for running untrusted creator applications.
- **One Worker per theme release only:** best code reuse but would share server execution among merchants; initial preference is per-installation Worker; benchmark operational/economic tradeoff and preserve strict tenant isolation.

## Prototype gates before permanent commitment

- Prove actual output of React Router SSR app can be compiled to and deployed as untrusted user Worker through Cloudflare dispatcher.
- Demonstrate multi-domain SSO portal and verified hostname dispatch for two merchants using both same and different release.
- Show no cross-tenant data, state, cache, config or credential leaks and no egress to prohibited endpoints.
- Reproduce clean builds, minimal dependency lifecycle risks, immutable release rollback and preview isolation.
- Measure p95 TTFB, product-page HTML SEO, Core Web Vitals, bundle size, CPU/subrequest usage, caching and economic scenario 200 themes/5,000 stores.
- Verify current Cloudflare WfP script, storage and domain pricing/limits and acceptable fallback if account features/policy or price change.

## Source verification (2026-10)

- https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/
- https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/worker-isolation/
- https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/how-workers-for-platforms-works/
- https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/reference/pricing/
- https://developers.cloudflare.com/cloudflare-for-platforms/workers-for-platforms/configuration/hostname-routing/
- https://developers.cloudflare.com/cloudflare-for-platforms/cloudflare-for-saas/start/getting-started/
- https://reactrouter.com/
