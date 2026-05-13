# sap-skills

[![skills.sh](https://skills.sh/b/Fab2295/sap-skills)](https://skills.sh/Fab2295/sap-skills)

> Monorepo of **SAP CAP** skills for [Claude Code](https://claude.com/claude-code).
> All skills are scoped to **SAP CAP Node.js** projects, are anchored to
> [capire](https://cap.cloud.sap/docs/) documentation, and have **strict
> negatives** (no commits, no pushes, no out-of-scope writes).

## Install

Install the whole bundle in one shot:

```sh
npx skills add Fab2295/sap-skills
```

The installer reads the `skills/<name>/SKILL.md` files in this repo and
deploys them to your agent's skill directory
(e.g. `~/.claude/skills/<name>/`).

## Skills in this repo

| Skill | Purpose | What it writes |
|---|---|---|
| [`sap-cap-test`](skills/sap-cap-test/) | Scaffolds and runs CAP tests with `cds add test` + `cds test`. Opt-in coverage via `c8`. | Test files under `test/` and one of `CAP-TEST-REPORT.md` / `CAP-TEST-FAILURE.md` at the project root. |
| [`sap-cap-code-review`](skills/sap-cap-code-review/) | Read-only static analysis of a PR / branch / file list. Classifies findings as Critical / High / Medium / Low. | `CAP-CODE-REVIEW.md` at the project root. |
| [`sap-cap-upgrade`](skills/sap-cap-upgrade/) | Upgrades `@sap/cds*`, `@cap-js/*`, `@sap-cloud-sdk/*`, `@sap/eslint-plugin-cds` to the latest stable (incl. majors). Runs a **vulnerability gate** on every target version (osv.dev primary, npm advisory bulk fallback) — aborts the upgrade when any target has an advisory ≥ moderate. Cross-checks failures against locally mirrored CAP + Cloud SDK JS changelogs and reports ONLY bugs caused by the version bump. | Edits `package.json` (and `package-lock.json` via `npm install`) in apply mode — and only when the vulnerability gate passes. Plan mode writes nothing. |
| [`sap-cap-nodejs-dev`](skills/sap-cap-nodejs-dev/) | CAP Node.js development guide. Domain-first ("less code → less mistakes"): prefers CDS schema, annotations, projections and CAP's generic providers over hand-written handlers. Strict scope: refuses UI/frontend, Java CAP, non-CAP backends, and any internal/protected/deprecated CAP API. | Suggests CDS / JS / config files inside the project's `db/`, `srv/`, and `package.json` / `.cdsrc.json`. Never edits `app/` (UI). Never runs `git add/commit/push` and never installs dependencies. |

### sap-cap-test (test-only)

| Allowed write targets |
|---|
| `test/**`, `tests/**`, `__tests__/**` (the project's test folder) |
| `CAP-TEST-REPORT.md`, `CAP-TEST-FAILURE.md` (project root) |

**Never**: edits `srv/`, `db/`, `app/`, `package.json`, `.cdsrc.json`,
`mta.yaml`, `xs-security.json`; runs `git add/commit/push`; installs
dependencies on its own; enables coverage without explicit opt-in.

Reference: <https://cap.cloud.sap/docs/node.js/cds-test>

### sap-cap-code-review (read-only review)

| Allowed write targets |
|---|
| `CAP-CODE-REVIEW.md` (project root) |

**Never**: edits any source file; runs `git add/commit/push`; emits a
finding that isn't anchored to a Rule ID under
`skills/sap-cap-code-review/references/`; pastes a verbatim secret
into the report.

> 🔒 **Secret redaction (mandatory).** Every Evidence and Suggested-fix
> excerpt is passed through the fail-closed filter at
> [`skills/sap-cap-code-review/references/secret-redaction.md`](skills/sap-cap-code-review/references/secret-redaction.md):
> credential-shaped keys (`clientSecret`, `password`, `token`, `apiKey`,
> `privateKey`, `connectionString`, etc.) and value-shape patterns
> (JWTs, `Bearer …`, PEM blocks, URLs with embedded `user:password@`,
> long hex/base64 blobs) are replaced with `[REDACTED:<kind>]`.
> Excerpts from credential-shaped files (`xs-security.json`,
> `manifest.yaml`, `mta.yaml`, `default-services.json`,
> `default-env.json`, `.env*`, `secrets/`) are replaced wholesale when
> any redaction trigger matches. SEC-007 (secrets inlined in source)
> uses a fixed redaction format — the value is never written to the
> report.

Reference rubric: `skills/sap-cap-code-review/references/severity-rubric.md`

### sap-cap-upgrade (CAP upgrade)

| Allowed write targets |
|---|
| `package.json` (in apply mode only) |
| `package-lock.json` — indirectly, via `npm install` |

**Never**: edits source code; runs `git add/commit/push/checkout/restore/stash`;
calls another skill; reports a failure as "version-caused" unless it satisfies
the strict A∧B∧C criteria in `skills/sap-cap-upgrade/references/bug-attribution-rules.md`.

**Default mode is `plan`** (read-only preview). Switch to `apply` only when
the invocation explicitly contains one of: `apply`, `aplicar`, `confirm`,
`confirmado`, `proceed`, `prosseguir`, `execute`, `executar`, `go`.

> 🔒 **Vulnerability gate (hard stop).** Before any `package.json` write the
> skill queries osv.dev (primary) and the npm advisory bulk endpoint
> (fallback) for every `<pkg>@<target>`. If any target has an advisory at
> **moderate severity or above**, the upgrade is cancelled — the JSON
> output uses `status: "vulnerable_target"` with the offending entries in
> `blocked_by_vulnerability[]`. Low-severity advisories surface as
> `vulnerability_warnings[]` and do not block. If both sources fail, status
> is `vuln_check_failed` (fail-closed). Contract: `skills/sap-cap-upgrade/references/vulnerability-check.md`.

> 🔒 **Output redaction (mandatory, fail-closed).** Every string the skill
> captures from `npm install`, `cds build`, the osv.dev response, and the
> npm advisory bulk endpoint is passed through
> [`skills/sap-cap-upgrade/references/output-redaction.md`](skills/sap-cap-upgrade/references/output-redaction.md)
> before being written into the JSON output (`notes[]`,
> `discarded[].error_excerpt`, etc.). Masks: `Authorization: Bearer …`,
> `_authToken=…` npmrc lines, JWTs, GitHub/AWS tokens, URLs with embedded
> `user:password@`, and unclassified token-shaped runs adjacent to
> credential keywords. The npm advisory fallback reads the token via a
> one-shot env var (`NPM_AUTH_TOKEN=$(...) curl ...`) — the token never
> reaches the visible command line. Truncation happens **after**
> redaction (not before — otherwise half a secret could survive at the
> boundary).

> ⚠️ This published version of `sap-cap-upgrade` is **docs-only** — the
> companion helper scripts (`latest-versions.js`, `refresh-references.js`)
> are NOT bundled. The skill uses `npm view <pkg> dist-tags.latest`
> directly for version resolution, and reference mirrors must be
> refreshed manually from the URLs in `skills/sap-cap-upgrade/references/source.md`.

Reference catalog: `skills/sap-cap-upgrade/references/packages-catalog.md`

### sap-cap-nodejs-dev (CAP Node.js development)

| Allowed write targets |
|---|
| `db/**` (CDS schema, views, seed CSV) |
| `srv/**` (service `.cds`, Node.js handlers, CAP-side `@UI.*` annotations) |
| `package.json`, `.cdsrc.json` (CDS config only — never adds runtime deps without the user) |

**Never**: edits `app/` (UI is out of scope); writes Java / Spring / Express / NestJS code; introduces non-CAP architectures (custom OData / REST / GraphQL outside `@cap-js/graphql`); imports from `@sap/cds/lib/...` or other internal paths; uses `@deprecated` / `@experimental` / `@internal` / `@protected` APIs; runs `git add/commit/push`; installs dependencies.

**Domain-first guarantee.** Decision order is enforced: schema → annotations → views/projections → `@cap-js/*` plugins → handler (last resort). The "is this PR domain-first?" checklist lives at [`skills/sap-cap-nodejs-dev/references/domain-first.md`](skills/sap-cap-nodejs-dev/references/domain-first.md).

> 🔒 **Audited surface.** The skill ships [`skills/sap-cap-nodejs-dev/SECURITY.md`](skills/sap-cap-nodejs-dev/SECURITY.md) with a 41-vector posture matrix (Prompt Injection, MCP/Tool Poisoning, Supply Chain, Eval / Command / SSRF / Path / Unsafe Deserialization, Credential / Token Leakage, Overpermission, Unauthorized Deploy, Telemetry Leakage, …) plus reproducible `grep` / Python checks the auditor can re-run. The only credential material in the corpus is mocked dev (alice/bob) and a postgres example, both `[development]`-profile gated and explicitly labeled DEV-ONLY.

Reference catalog: [`skills/sap-cap-nodejs-dev/SKILL.md`](skills/sap-cap-nodejs-dev/SKILL.md)

## Shared principles

Every skill in this repo follows the same hard rules:

1. **Read-mostly.** Writes are confined to a documented allowlist.
2. **No git mutations.** Read-only git (`status`, `diff`, `log`, `show`,
   `merge-base`, `rev-parse`, `ls-files`) is allowed. Anything that
   changes refs, working tree, or staging area is forbidden.
3. **No unsolicited installs.** A skill may surface
   `npm i -D <package>` and wait for the user; it never runs it.
4. **CAP Node.js only.** Java CAP / non-CAP Node.js projects are
   refused with an explicit, verbatim message.
5. **Anchored to capire.** Every behavior or finding traces back to a
   file under that skill's `references/` directory, which mirrors the
   relevant section of capire.

## Layout

```
sap-skills/
├── README.md            ← this file
├── LICENSE              ← MIT
├── .gitignore
└── skills/
    ├── sap-cap-test/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── sap-cap-code-review/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── sap-cap-upgrade/
    │   ├── SKILL.md
    │   └── references/          ← changelogs/, releases/, packages-catalog.md, ...
    └── sap-cap-nodejs-dev/
        ├── SKILL.md
        ├── SECURITY.md          ← 41-vector audit + reproducible checks
        ├── references/          ← domain-first.md, best-practices.md, capire mirrors
        └── templates/
```

`SKILL.md` is the contract the agent reads. `references/` are the
capire-anchored rules each skill cites. `templates/` are the markdown
or JS files the skill writes into the user's project.

## License

[MIT](LICENSE) © 2026 Fabricio
