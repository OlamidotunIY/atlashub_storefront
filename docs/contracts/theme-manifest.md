# Theme manifest contract — proposal

**Status:** Draft. Schema and parser not yet implemented.

The theme engine consumes versioned manifest + declarative page definitions. The manifest is metadata and a contract, NOT executable JavaScript.

## Proposed theme manifest
```json
{
  "schemaVersion": "1.0",
  "themeId": "studio-minimal",
  "themeVersion": "1.0.0",
  "creatorId": "atlashub",
  "engineCompatibility": ">=1.0.0 <2.0.0",
  "supportedPages": ["HOME","COLLECTION","PRODUCT","CART","CONTENT"],
  "supportedLocales": ["en"],
  "sections": [
    {"type": "hero", "settingsSchema": "hero.v1"},
    {"type": "product-grid", "settingsSchema": "product-grid.v1"}
  ],
  "designTokens": {"colors": {}, "typography": {}, "spacing": {}},
  "assets": [],
  "integrityDigest": "sha256:<computed-release-hash>"
}
```
Example is illustrative; `integrityDigest` must be calculated by the build pipeline, never accepted as arbitrary author input.

## Other contracts
- `Storefront`: organizationId, storefrontId, activeThemeReleaseId, market, locale, draft/published status.
- `ThemeRelease`: themeId, version, immutable asset bundle, schemaVersion, approval, compatibility, signature/hash and publish timestamps.
- `DomainBinding`: canonical hostname, tenant, verification challenge, status, TLS status and alias/redirect policy.
- `PageDefinition`: pageType, ordered typed section instances, allowed settings, safe references, translations.
- `ThemeLicense`: buyer organization, theme/version entitlements, permitted installations and refund/revocation policy.

## Safe bindings
Allow only versioned view-model fields (product title, image, variant, published price, stock status, merchant branding). Pages must not access private database tables, provider credentials, customer PII or arbitrary remote endpoints. Server-side commerce APIs recalculate all prices and inventory at checkout regardless of template output.

## Backward compatibility
Release bundle immutable. Engine supports declared schema versions for an explicit window; migrations preserve merchant customization drafts and publication history. Theme author changes do not overwrite published releases without review.
