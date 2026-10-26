# Storefront Management/Hosting — source folder blueprint

**Standalone TypeScript backend repo.** No `apps/portal`, no `atlashub_auth_web`, no SDK scaffold source here. **No Turborepo/Nx**. One deployment unit for API may have separate build/publish/dispatch workers inside same repo, independently deployed as required.

```text
atlashub_storefront/
├── docs/
│   ├── architecture/
│   ├── contracts/
│   ├── decisions/
│   ├── developer/                   # cross-repo SDK contract only
│   ├── guidelines/
│   ├── modules/
│   └── security/
├── src/
│   ├── domain/
│   │   ├── themes/                  # Theme, ThemeVersion, ThemeRelease, invariant rules
│   │   ├── creators/                # creator profile and theme ownership
│   │   ├── marketplace/             # listing, review, license eligibility views
│   │   ├── installations/           # per-store pinned release, entitlement reference
│   │   ├── storefronts/             # configs, drafts and publications
│   │   ├── assets/                  # owned media and release references
│   │   └── domains/                 # verified host/domain and TLS state
│   ├── application/
│   │   ├── ports/                   # repos, billing/JAVA auth, Cloudflare/R2/build/events
│   │   ├── commands/                # create/upload/review/install/config/publish
│   │   ├── queries/                 # catalog, installations, status and creator sales
│   │   ├── policies/                # tenancy, entitlement and visibility
│   │   └── dto/
│   ├── infrastructure/
│   │   ├── persistence/             # PostgreSQL, transactions, outbox
│   │   ├── object-storage/          # R2 signed uploads, object lifecycle
│   │   ├── cloudflare/              # Workers for Platforms, DNS/TLS, dispatch
│   │   ├── identity-java/           # centralized Java IAM/SSO validation
│   │   ├── commerce-java/           # public commerce gateway, billing/entitlements
│   │   ├── builder/                 # disposable source-build orchestrator
│   │   ├── queues/                  # durable jobs and scheduled cleanup
│   │   └── telemetry/
│   ├── presentation/
│   │   ├── http/                   # REST/BFF-ready management controllers
│   │   ├── webhooks/               # signed callbacks; verify before dispatch
│   │   └── jobs/                   # worker job consumers invoke app commands
│   └── bootstrap/                  # DI, app config, startup + health
├── workers/
│   ├── build/                      # isolated untrusted source build executable
│   ├── publish/                    # deployment and cache invalidation executor
│   ├── dispatch/                   # trusted hostname->Worker edge dispatcher
│   └── theme-runtime/              # edge SSR contracts only; theme code untrusted
├── contracts/
│   ├── management-api/
│   ├── theme-manifest/
│   └── public-commerce/
├── migrations/
├── tests/
│   ├── unit/
│   ├── contract/
│   ├── integration/
│   ├── e2e/
│   └── security/
├── package.json
└── tsconfig.json
```

Directory names are proposals, **not currently present**. Feature context may own full `domain/application/infrastructure/presentation` slices as it grows; choose one consistent code organization and enforce imports. No circular domain dependencies; cross-context communication through named application ports and events.

## External repository integrations
- Separate Auth UI: [atlashub_auth_web](https://github.com/OlamidotunIY/atlashub_auth_web).
- Separate Merchant/Creator Portal: [atlashub_storefront_portal](https://github.com/OlamidotunIY/atlashub_storefront_portal).
- SDK/CLI/scaffold: [atlashub_theme_sdk](https://github.com/OlamidotunIY/atlashub_theme_sdk).
- Java identity/business authority: [atlashub_backend](https://github.com/OlamidotunIY/atlashub_backend).
- Existing merchant operations: [atlashub_web](https://github.com/OlamidotunIY/atlashub_web).

Do not cross-import `src` across Git repositories; consume versioned OpenAPI/contracts and published packages. A browser portal never receives R2 root credentials, Cloudflare API credentials or arbitrary merchant secrets.
