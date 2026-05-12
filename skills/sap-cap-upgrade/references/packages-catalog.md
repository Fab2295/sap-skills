# Packages Catalog — In-Scope Libraries

The `sap-cap-upgrade` skill operates **only** on packages whose name matches the regex below. Anything else in `package.json` (including `@sap/xssec`, `@sap/approuter`, `@sap/hana-client`, `@sap/audit-logging` if present as standalone, etc.) is left untouched — even when listed in the same file.

## Aggregate regex (authoritative)

```
^(@sap/cds(-.*)?|@cap-js/.+|@sap-cloud-sdk/.+|@sap/eslint-plugin-cds)$
```

Each match is routed to **exactly one** changelog source. The skill must use the routing table below to decide which mirror to consult during bug attribution.

## Routing table

| Family | npm name pattern | Changelog source | Local mirror folder |
|---|---|---|---|
| CAP framework | `@sap/cds`, `@sap/cds-dk`, `@sap/cds-lsp`, `@sap/cds-compiler`, `@sap/cds-mtxs`, `@sap/cds-fiori`, `@sap/cds-hana` | https://cap.cloud.sap/docs/releases/ | `references/changelogs/cap/` |
| CAP plugins (community) | `@cap-js/sqlite`, `@cap-js/postgres`, `@cap-js/hana`, `@cap-js/openapi`, `@cap-js/asyncapi`, `@cap-js/telemetry`, `@cap-js/mcp-server`, `@cap-js/audit-logging`, `@cap-js/attachments`, `@cap-js/change-tracking`, all other `@cap-js/*` | https://cap.cloud.sap/docs/releases/ | `references/changelogs/cap/` |
| CAP lint | `@sap/eslint-plugin-cds` | https://cap.cloud.sap/docs/releases/ | `references/changelogs/cap/` |
| SAP Cloud SDK for JavaScript | `@sap-cloud-sdk/connectivity`, `http-client`, `odata-common`, `odata-v2`, `odata-v4`, `openapi`, `generator`, `resilience`, `util`, `eslint-config`, all other `@sap-cloud-sdk/*` | https://sap.github.io/cloud-sdk/docs/js/release-notes | `references/changelogs/cloud-sdk-js/` |

## Out of scope (never bumped, never blamed)

These packages may exist in `package.json` but are deliberately ignored by `sap-cap-upgrade`:

- `@sap/xssec`, `@sap/approuter`, `@sap/hana-client`, `@sap/audit-logging` (the standalone non-`@cap-js/*` variant), `@sap/instance-manager`, `@sap/textbundle`, `@sap/hdi-deploy`, `@sap/cds-dbm`
- Anything not under `@sap/cds*`, `@cap-js/*`, `@sap-cloud-sdk/*`, or `@sap/eslint-plugin-cds`
- Dev/runtime peers (`express`, `passport`, `sqlite3`, `better-sqlite3`, etc.)

## Disambiguation

When a package matches **both** `@sap/cds(-.*)?` and another rule (it cannot in practice), CAP wins. When a name has historical aliases (e.g. some `@sap/audit-logging` features migrated to `@cap-js/audit-logging`), only the package literally present in `package.json` is bumped — no automatic re-mapping.
