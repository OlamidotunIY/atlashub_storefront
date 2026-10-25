# ADR 0001 — Separate theme-management backend from storefront renderer

**Status:** Proposed; renderer technology not yet selected.

## Context
Atlas Hub's merchant storefront platform should support hundreds of theme packages, third-party creators, safe data binding, visual editing, domains and managed publication. It should not deploy a separate Next.js app or server for every merchant or theme.

## Decision
Start with a TypeScript monorepo containing distinct theme-management API, schema/engine, public runtime, editor and worker application boundaries. Use declarative manifests and approved components as the default third-party theme interface. Deploy one horizontally scalable multi-tenant runtime pool where possible.

Do **not** select Next.js solely because the storefront uses React. The rendering implementation is an explicit benchmark decision:
1. **Next.js SSR/runtime** — ecosystem, routing, rendering and caching; design strict theme/component isolation.
2. **Custom Node renderer with safe Liquid-like language** — a more template-centric author experience but requires building and securing rendering/caching/SEO systems.

Neither path requires 200 applications. One theme is a release package/configuration consumed by shared runtime.

## Acceptance benchmark before final selection
Render 20 representative themes across two tenants and several locales/markets. Evaluate CPU/memory per request, p95 TTFB, Core Web Vitals, accessibility, security under malicious template content, warm/cold caching, preview/publish/rollback, DX for creators, per-tenant isolation and deployment operations. Choose based on findings, not theoretical performance.

## Consequences
Separate backend enables creator marketplace, versioning and domain/asset workflows without coupling to any SSR library. Extra boundaries require contract/version discipline. Arbitrary third-party server-side code is excluded initially.

## Related
- System: ../architecture/system.md
- Theme contract: ../contracts/theme-manifest.md
- Security: ../security/theme-sandbox.md
