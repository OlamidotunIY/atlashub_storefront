# Atlas Hub Storefront & Theme Platform

> **Status:** Documentation-first architecture. No deployable storefront or theme engine is implemented in this repository yet.

A multi-tenant commerce storefront platform with merchant-managed custom domains, versioned and paid themes, a secure theme authoring/creator ecosystem, and public storefront rendering. Hundreds of themes share one compatible rendering contract, not one application or server per theme.

## Clear ownership

- **Atlas Hub Java backend:** Products, inventory, prices, outlets, customers, order checkout, provider eligibility/routing, payments reconciliation, accounting, merchant entitlements and authorization.
- **Storefront management backend (TypeScript):** Theme catalog, versions, creator submissions, editor drafts, publishing, domain provisioning, release validation, assets and jobs.
- **Theme engine:** Validated declarative sections, blocks, schemas, settings and commerce data binding; never runs arbitrary creator code with server privileges.
- **Public storefront runtime:** Resolves verified hostname -> merchant storefront -> published theme release. Next.js is one *candidate* SSR/streaming runtime, not the entire template engine.
- **Theme editor:** Safe preview and customization without bypassing publishing and license checks.

## Architecture

Use DDD bounded contexts, Clean/Hexagonal Architecture and CQRS where workflows are stateful. Start with a TypeScript monorepo for management API, theme-engine packages, renderer, editor and background jobs. Compare Next.js with a custom Node.js/Liquid-like renderer using a measured prototype before selecting the final runtime.

## Documents

- [System architecture and deployment](docs/architecture/system.md)
- [Module-by-module implementation plan](docs/modules/index.md)
- [Theme manifest contract](docs/contracts/theme-manifest.md)
- [Third-party theme security](docs/security/theme-sandbox.md)
- [ADR 0001: Rendering-runtime options](docs/decisions/0001-runtime-options.md)

## Guiding rules

Every domain must resolve to an authorized single tenant. Public theme assets never contain integration credentials. Product availability and prices come from backend-authoritative commerce APIs at checkout. Domain ownership verification, TLS and rollback are mandatory. Merchant-paid themes use versioned entitlements; third-party creator code is untrusted.

**Planning only.** No theme, marketplace, merchant deployment or domain automation is live merely because it appears in these documents.
