# Sources of Truth — monza

This file pins the canonical upstream URLs for every changelog mirrored in `references/changelogs/`. The `monza` skill **must** cite a `rule_id` anchored in one of these mirrors when reporting a `version_caused_bug`. Anything not anchored here is, by definition, not version-attributable.

`scripts/refresh-references.js` rewrites this file on every run.

---

## Source 1 — SAP Cloud Application Programming Model (CAP)

- **Index URL**: https://cap.cloud.sap/docs/releases/
- **Per-year changelog URL pattern**: `https://cap.cloud.sap/docs/releases/<YYYY>/changelog`
- **Per-month release URL pattern**: `https://cap.cloud.sap/docs/releases/<YYYY>/<mon><YY>` (e.g. `apr26`, `dec25`)
- **Local mirror**:
  - Yearly: `references/changelogs/cap/changelog-<YYYY>.md`
  - Monthly: `references/releases/<YYYY>/<mon><YY>.md`
- **Coverage** (npm package patterns): `@sap/cds`, `@sap/cds-dk`, `@sap/cds-lsp`, `@sap/cds-compiler`, `@sap/cds-mtxs`, `@sap/cds-fiori`, `@sap/cds-hana`, `@sap/eslint-plugin-cds`, `@cap-js/*`
- **Section names that count as "incompatible change"**: `Changed`, `Removed`, `Fixed`, `Breaking Changes`, `Migration`
- `last_fetched`: 2026-05-09T02:26:56.619Z

---

## Source 2 — SAP Cloud SDK for JavaScript

- **Index URL**: https://sap.github.io/cloud-sdk/docs/js/release-notes
- **Per-major URL pattern**: `https://sap.github.io/cloud-sdk/docs/js/v<N>/release-notes`
- **Local mirror**: `references/changelogs/cloud-sdk-js/changelog-v<N>.md`
- **Coverage** (npm package patterns): `@sap-cloud-sdk/*` (e.g. `connectivity`, `http-client`, `odata-common`, `odata-v2`, `odata-v4`, `openapi`, `generator`, `resilience`, `util`, `eslint-config`)
- **Section name that counts as "incompatible change"**: `Compatibility Notes`
- `last_fetched`: 2026-05-09T02:26:57.487Z

---

## Refresh policy

The coordinator agent runs `node scripts/refresh-references.js` automatically when, for any source:
- `last_fetched` is older than 30 days, OR
- the upgrade target version has no entry in the corresponding mirror.

Manual refresh: `node ~/.claude/skills/monza/scripts/refresh-references.js`.
