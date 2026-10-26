# ADR 0002 — Independent frontends; standalone Storefront backend and SDK tooling

**Status:** Accepted architectural direction, documentation only. **Date:** 2026-10-25. **Supersedes** earlier draft proposing one TypeScript monorepo for portal + SDK + backend.

## Context
Atlas Web is the existing business operations dashboard. Independent merchant theme management and theme creator workflows need a separate frontend. All Atlas apps share identity and the same User must be able to join/create organizations separately from registration. Developer React SDK, CLI and Vite scaffolding have separate release cadence and npm distribution. The user explicitly prefers **separate repositories** and **no Turborepo, Nx or monorepo management tools**.

## Decision and ownership

| Repository | Scope | Status |
|---|---|---|
| [atlashub_auth_web](https://github.com/OlamidotunIY/atlashub_auth_web) | Central React/Vite login, registration, verification, recovery, MFA UI, SSO challenge UI | Created; documentation added |
| [atlashub_backend](https://github.com/OlamidotunIY/atlashub_backend) | Java central User identity, credentials/IdP authority, IAM, independent organization creation, merchant data, platform billing and provider integrations | Existing; target migration proposed |
| [atlashub_web](https://github.com/OlamidotunIY/atlashub_web) | Business operations frontend; storefront recommendations and portal deep links; no duplicate theme dashboard | Existing |
| [atlashub_storefront_portal](https://github.com/OlamidotunIY/atlashub_storefront_portal) | Independent React/Vite Merchant+Creator Portal frontend, settings, marketplace, domains and listings | Created; documentation added |
| **atlashub_storefront (here)** | Node/TypeScript Storefront management backend, themes/versions/licensing references, build/review/publishing, R2, Cloudflare Workers/dispatch/domains, public storefront runtime | Existing; no Portal frontend here |
| [atlashub_intelligence](https://github.com/OlamidotunIY/atlashub_intelligence) | Python AI/voice agent; Java business tool authority | Existing |
| [atlashub_theme_sdk](https://github.com/OlamidotunIY/atlashub_theme_sdk) | Public browser theme SDK, Vite/React Router scaffold, creator CLI, theme contracts, versioned npm packages | **Repository created; architecture docs initialized; no published SDK/CLI yet** |

## Identity and onboarding
- User registration/login **not** equal to Organization creation. Register a global individual User with zero organizations allowed. Business creation/invitations are optional later workflows.
- `atlashub_auth_web` is UI **only**; credentials, OIDC Authorization Code+PKCE/BFF authorization, ID tokens, sessions and IAM are controlled by Java backend/approved IdP. Separate apps establish individual sessions using central SSO, not shared localStorage.
- Creator can register profile in Portal without business/KYB. Merchant business creation is not blocked on Anchor/KYB and payment processor connection is optional; only provider-dependent payment functionality requires verified eligible connection.
- Merchant and Creator portal workspaces share identity but server-side role/ownership checks remain separate.

## Source and dependency management
- Standard individual repositories and GitHub Actions CI/CD; no workspace/monorepo tools.
- Versioned OpenAPI/JSON Schema, published npm SDK/contracts and compatible semantic version ranges, code-generation and cross-repo contract tests. No runtime imports across Git repositories and no copy-paste authorization logic.
- Backend services may contain several modules/processes (management API, build/publish workers, Cloudflare dispatch) without needing per-module GitHub repos; split only when deployment/security boundaries justify.
- Portal independently deployable; Auth UI independently deployable; SDK npm release and Storefront service release can occur separately.
- Continue Cloudflare Workers for Platforms untrusted-mode SSR candidate, R2/CDN, Cloudflare for SaaS, trusted checkout, no duplicate merchant business data.

## Consequences
Positive: clear identity and creator experience, independent releases/deployments and no toolchain forced monorepo. Tradeoffs: explicit API contract versioning, release compatibility, cross-repo CI, operational ownership and separately hosted frontends. Accepted.

## Related
[Storefront architecture](../architecture/system.md), [portal/SSO integration](../architecture/portal-identity.md), [API contracts](../contracts/platform-apis.md), [SDK contract](../developer/sdk-cli.md), [storage](../architecture/storage-data.md), [runtime](../architecture/runtime-hosting.md).
