# Creator developer experience — Vite scaffold, React SDK and theme CLI

**State:** Specification; the following commands are examples, not published npm packages yet.

## Design intention

A creator should have roughly the same experience as creating a Vite/React application. They get routing contracts, fixture/mock data, SDK/client, TypeScript schema, accessibility checks, preview, manifests, and validated build/publish commands. Their React components, layout, animations and styles remain largely theirs.

**Selected scaffold:** Vite + React + TypeScript + React Router Framework Mode. SSR/prerender compatibility is required for indexable public pages. Restrict server runtime to audited Worker-compatible imports. Next.js is not used for creator projects.

## Developer flow

\`\`\`bash
# Proposed future commands — not installed yet
npx create-atlashub-theme fashion-studio
cd fashion-studio
npm install
npm run dev

npx atlashub login
npx atlashub whoami
npx atlashub theme validate
npx atlashub theme build
npx atlashub theme preview
npx atlashub theme publish --version 1.0.0
npx atlashub theme versions
npx atlashub theme unlist
\`\`\`

\`create-atlashub-theme\` is distributed as a CLI scaffolder. \`@atlashub/storefront-sdk\` is the versioned library built on **safe public commerce APIs**. \`@atlashub/theme-contracts\` holds manifest and page/settings schemas. Choose stable final package names/ownership after npm namespace availability check. Never ask developers for merchant API secrets.

### Scaffold anatomy

\`\`\`text
fashion-studio/
  app/
    routes/
      home.tsx
      products.$handle.tsx
      collections.$handle.tsx
      cart.tsx
      pages.$slug.tsx
      404.tsx
    components/
    layouts/
    styles/
  public/
    theme-defaults/logo.svg
    theme-defaults/hero.webp
  theme.settings.ts
  atlashub.theme.json
  react-router.config.ts
  vite.config.ts
  package.json
  package-lock.json
  tsconfig.json
\`\`\`

Illustrative file names; final React Router route convention chosen in actual scaffold. Include test/lint/format and documentation.

## SDK public capabilities

Read-only: \`products.list/get\`, \`collections.list/get\`, \`storefront.getPublicDetails\`, media URL resolution, published prices and availability; client React hooks for loading/cache/error.

Customer cart: \`cart.create/get/add/update/remove\` with untrusted session handling and Java recalculation. Checkout: \`checkout.start\` returns trusted checkout URL/session, **not** a way to capture card/bank details inside arbitrary theme code. Customer auth, order history and PII require a separate designed, restricted customer flow.

Theme: \`useThemeSettings()\` and \`useStorefrontContext()\` return public, validated active theme config and locale/market; developer need not manually pass org ID. SDK should not expose internal billing/merchant settlement, pay:controls, ledger, HR, staff, inventory stock-edit, customer private records, or admin mutation APIs.

Example only:

\`\`\`tsx
import { atlashub, useThemeSettings } from "@atlashub/storefront-sdk";

export function HomeHero() {
  const { heroTitle, primaryColor, logo } = useThemeSettings();
  const { data, isLoading } = atlashub.products.useList({limit: 8});
  return (
    <main style={{color: primaryColor}}>
      <img alt="Store logo" src={logo}/>
      <h1>{heroTitle}</h1>
      {isLoading ? <p>Loading</p> : data?.items.map(p => <ProductCard key={p.id} product={p}/>)}
    </main>
  );
}
\`\`\`

Do not confuse API client typing with security: the server always limits data by verified store identity and authoritative Java policies.

## CLI behavior and security

- Login through browser authorization/device flow against Atlas Hub central identity; token short-lived/scope-limited and revocable. Local storage in OS credential vault when available, not committed to repo.
- \`init\`: create scaffold, provide JSON schema/SDK version, fixture data and basic default media.
- \`dev\`: local mock data or sandbox store with strictly limited data, no privileged tokens; server/browser preview shows default/merchant-overridden settings safely.
- \`validate\`: verify pages, manifest, settings defaults, capabilities, assets, dependencies and SSR compatibility; report precise diagnostics.
- \`build\`: reproducible Vite + React Router SSR/client bundles; source maps kept private when necessary; generated digest/size reports.
- \`preview\`: local or remote expiring isolated preview, no-index, sample/authorized merchant data only.
- \`publish\`: never accept a local build as pre-approved. Request signed source-upload target, send archive and version metadata, trigger **server-side** independent validation/build/review. Response provides upload/build/review ID, **not** immediate public listing.
- \`versions\` / \`unlist\`: show releases, metadata and review statuses; unlist prevents new sales, not breaking every installed store.
- \`delete\`: distinguish draft deletion, unlisted theme and hard legal/safety removal; fail safely if active installations/licenses depend on a release.

## Marketplace and versioning flow

Local -> authenticated upload -> private quarantine -> builder -> diagnostics -> private test preview -> submit review -> reviewer approval -> immutable theme release -> listing -> tenant entitlement -> install/preview/publish -> updates. Detailed state machine: [marketplace lifecycle](../architecture/marketplace-lifecycle.md).

## CSS, external libraries and creative freedom

Allow arbitrary CSS/layout/responsive styles and React components that comply with route/accessibility/security contracts. Allow vetted animation/media libraries, avoid blanket design restrictions. Unknown package dependencies and custom scripts go through controlled review; deny or sandbox privileged Node/browser capabilities. No custom checkout card forms or privileged server adapters.

## Compatibility

Version \`theme manifest\`, \`public SDK\`, \`route contract\`, \`SSR adapter\`, and \`Worker deployment\` independently. Warn if a theme uses deprecated hooks or browser-only APIs during SSR. Require locked dependencies and CI regression with at least one full end-to-end theme example.
