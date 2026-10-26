## Architecture / design

- [ ] I identified the affected bounded context(s), use cases and domain invariants.
- [ ] My dependency direction follows **Domain ← Application ← Infrastructure / Presentation**, with ports owned inward and adapters outside.
- [ ] I checked the repository's [architecture](../docs/architecture/clean-hexagonal.md) and [folder structure](../docs/architecture/folder-structure.md) policies. No domain import of React/HTTP/ORM/ML/cloud frameworks.
- [ ] I applied DDD/CQRS **pragmatically**; no empty ceremony, circular dependencies or duplicate authority from Java/IdP/provider services.

## Contracts and security

- [ ] I updated versioned API/event/schema contracts and compatibility where applicable.
- [ ] I checked principal, organization/creator/storefront isolation, sensitive capabilities, secrets, data/retention and idempotency for affected surfaces.
- [ ] I reviewed the [engineering/security guidelines](../docs/guidelines/engineering-rules.md).
- [ ] User registration remains independent of business creation; no mandatory Atlas Anchor KYB/bank provisioning or phantom provider success is introduced.

## Testing and delivery

- [ ] Unit tests cover domain policy and application use case outcomes.
- [ ] Adapter/contract tests cover Java/IdP/Storefront services or model integrations when affected.
- [ ] E2E/security/a11y/model tests are updated for relevant changes and invalid/timeout/revoked cases.
- [ ] All applicable checks in [testing/release guidelines](../docs/guidelines/testing-release.md) pass.
- [ ] README, module plan, migrations and ADRs are updated for behavior or architectural decisions.

**Status:** This PR template is guidance. Automated static architecture checks and CI gates must be implemented before claiming enforcement.
