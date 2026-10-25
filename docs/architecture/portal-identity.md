# Portal UX, shared Atlas Hub identity, multi-role workspaces

**State:** Architecture contract; implementation pending. The portal is a **separate web application and separate GitHub repository** at [atlashub_storefront_portal](https://github.com/OlamidotunIY/atlashub_storefront_portal), not located under this backend's source tree. Shared login/register/recovery frontend is [atlashub_auth_web](https://github.com/OlamidotunIY/atlashub_auth_web); Java/IdP owns credentials and OAuth/OIDC.

## User and business experiences

### Atlas Web (existing business dashboard)
Owns merchant business administration, POS, products, inventory, payroll, payments, accounting, analytics. Adds storefront **recommendation cards** and Storefront links/deep links only. Clicking a theme opens the Storefront Portal listing; Atlas Web does not reproduce full template catalog, creator management or storefront publishing features.

### Atlas Storefront Portal — Merchant workspace
- Landing/overview: own stores, status, theme releases, domains, drafts and actionable alerts.
- Theme marketplace: featured/recommended/search/filter/theme version/compatibility/reviews/demo/pricing/licenses.
- Acquisition: purchase/claim verified license via **Atlas Hub platform billing**; then install and activate selected approved release.
- Configuration: logo, favicon, colors, fonts, copy, banners, social links, collection references, layout toggles **where developer declares fields**; default values guaranteed from manifest.
- Assets: secure upload, library, audit, quotas, previews, reuse and deletion.
- Preview and publication: non-indexable draft preview, publish/rollback, compare theme updates and config migrations, preview custom domain, view release diagnostics.
- Domains: connect existing domains, DNS/ownership proof, certificate status, redirects, renewal warnings; optional domain registrar sale later.
- Usage/support: publication history, website health and basic theme licenses; general inventory/order management stays in Atlas Web.

### Atlas Storefront Portal — Creator workspace
- Creator registration without mandatory business/legal-entity onboarding for initial authoring; earnings may require applicable verification.
- Templates: create, listing details, versions, release reviews/errors, preview, prices, draft/listed/unlisted status, feedback.
- Developer tools: CLI sign-in, scoped token/device auth, SDK version, validation warnings, API docs, compatibility matrix.
- Performance/business: installation/version adoption, theme sales, earnings, adjustments/refunds, support contact, security notices.
- No drag-and-drop creator editor in initial scope. Creators write React themes locally and use SDK/CLI.

### Shared user can access both
One central AtlasHub User may have CreatorProfile and memberships in multiple Organizations, each with independent permission. Switch roles/workspaces without changing identity; organization must be selected for merchant context. A creator cannot access a store's merchant configuration just because they wrote its theme. A merchant cannot modify a creator's immutable uploaded source.

## Proposed routes (examples, not implemented)
```
https://app.<brand>/dashboard
https://app.<brand>/commerce/...          # Atlas Web

https://storefront.<brand>/marketplace
https://storefront.<brand>/marketplace/:themeId
https://storefront.<brand>/merchant/stores
https://storefront.<brand>/merchant/stores/:storeId/themes
https://storefront.<brand>/merchant/stores/:storeId/settings
https://storefront.<brand>/merchant/stores/:storeId/domains
https://storefront.<brand>/merchant/stores/:storeId/publish
https://storefront.<brand>/creator/templates
https://storefront.<brand>/creator/templates/:themeId/versions
https://storefront.<brand>/creator/reviews
https://storefront.<brand>/creator/earnings
```
Example addresses are **not** registered assets, confirmed brands or live services.

## Single sign-on (SSO): desired security behavior

1. The browser has an authenticated **Atlas Hub identity-provider session** established through the central `atlashub_auth_web` login UI and Java/IdP identity backend; independent user registration requires no organization or Anchor KYB.
2. Clicking an Atlas Web theme recommendation navigates to the portal via URL containing **only a public theme ID and optional safe return context**, never a token or private org authority.
3. Portal initiates OpenID Connect Authorization Code + PKCE flow against central identity service (preferred) or a secure BFF equivalent.
4. IdP recognizes the existing session and returns one-time code to portal callback. The portal establishes its **own** secured session (HTTP-only, Secure, SameSite policy chosen per flow). This *may* appear automatic to user, but SSO cannot be assumed until Java auth supports it.
5. API enforces signed session identity, membership and active org selection on each mutation, resolves entitlements via Java.
6. Logout/session revocation, account suspension, org role removal and creator suspension propagate; portal invalidates its own session or forces fresh auth.

Do **not**:
- Pass access/refresh tokens in redirects/query/hash fragments.
- Read Atlas Web localStorage and copy JWT into Portal origin.
- Treat an organization ID or creator ID from the URL as authorization.
- Expose IdP refresh token or merchant admin bearer token to developer theme code.
- Share management cookies with arbitrary merchant custom-store domains.
- Require independent duplicate credentials or merchant KYC solely to become a theme creator.

## Identity/permissions matrix

| Operation | Merchant admin or delegated publisher | Independent creator | Atlas staff reviewer |
|---|---:|---:|---:|
| Browse public marketplace | Yes | Yes | Yes |
| Buy/install theme for authorized org | Yes | No (unless also merchant) | Only scoped support |
| Configure/publish own store | Yes | No (unless also merchant) | Scoped audited support only |
| Create local theme with SDK | Yes if enrolled creator | Yes | Not special |
| Upload/publish own theme release | If creator identity | Yes (own theme) | No by default |
| Review/approve public listings | No | No | Permission-gated and audited |
| Access catalog pricing/orders/inventory | Existing Java role permits | Public storefront API only | Existing policy |

Separate entitlement/capability checks for subscriptions and available countries as required.

## URL and iframe safety
All cross-app return/deep links must use an allowlisted origin and internal route; reject open redirects. Portal preview should use separate, unprivileged origins and block framing/messaging from untrusted origins unless purpose-built contract is used. CSRF protection for authenticated portal mutations and CLI OAuth device flows.

## Existing source touchpoints (implementation later)
Atlas Web currently has `src/app/router/routes/merchant.routes.tsx` merchant auth/layout and `src/app/shell/main/sidebar/SidebarNavigation.tsx`; use them to add recommendation navigation only.
Java IAM/auth is authoritative; extend to central SSO and creator role/profiles through API contracts. Do not add a standalone identity database to the storefront backend.

## Decisions
Separate portal **yes, separate frontend repository**. Separate Auth Web frontend **yes**. Additional new backend repo **no**. Independent SDK/CLI repo **planned, not yet created**. Shared identity with app-specific session **yes**. Independent creator onboarding **yes**. Drag-and-drop editor **not in initial release**.
