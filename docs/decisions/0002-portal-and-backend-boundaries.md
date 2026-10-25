# ADR 0002 — Dedicated Storefront Portal within existing storefront monorepo

**Status:** Accepted as design direction (not implemented).
**Date:** 2026-10-25.

## Context

Atlas Web already owns merchant business operations, and both merchants and independent theme developers require full theme management. We do not want to duplicate Atlas Web commerce/payroll/accounting UI. We do want theme marketplace and creator dashboards under one dedicated product surface with seamless sign-in.

## Decision

- Use existing `OlamidotunIY/atlashub_storefront` TypeScript monorepo for **new Storefront Portal frontend** and independent **Storefront management backend**, SDK, CLI, build/review workers, edge runtime packages and contracts.
- No fourth GitHub backend repository required. Individual apps are independently deployable within the repo.
- Existing `OlamidotunIY/atlashub_web` displays storefront **recommendations/deep links**, not duplicated full marketplace or merchant/storefront management.
- Merchants and independent creators both use Storefront Portal, with role-aware workspaces, cross-org selection and same AtlasHub user identity; a person can be both.
- Central AtlasHub sign-in with OIDC Authorization Code + PKCE or secure BFF, app-specific sessions; no URL token sharing. Existing Java IAM/Auth must actually implement/reuse conformant provider/SSO integration before automatic login exists.
- Creator registration does **not** require registering a merchant organization. Product subscription and role/permission rules remain Java authoritative.
- No drag-and-drop designer at launch; developers create React themes locally, merchants configure manifest-declared settings with developer defaults.

## Tradeoffs

Monorepo benefits: atomic SDK/portal/API contract changes, shared TypeScript schemas, easier CI/developer workflows. Tradeoffs: build isolation, deployment ownership, package boundaries and CI test cost. A separate portal repository is unnecessary initially and should only be considered after independent teams/release lifecycle genuinely require it.

## Deferred

Central social login choices, exact public domains, designer visual canvas, native mobile portal, paid theme creator payout jurisdiction policy. Domain/route examples in docs are placeholders, not live URLs.

## Related
[Portal and SSO](../architecture/portal-identity.md); [Architecture](../architecture/system.md); [Contract](../contracts/platform-apis.md).
