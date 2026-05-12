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
`skills/sap-cap-code-review/references/`.

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

> ⚠️ This published version of `sap-cap-upgrade` is **docs-only** — the
> companion helper scripts (`latest-versions.js`, `refresh-references.js`)
> are NOT bundled. The skill uses `npm view <pkg> dist-tags.latest`
> directly for version resolution, and reference mirrors must be
> refreshed manually from the URLs in `skills/sap-cap-upgrade/references/source.md`.

Reference catalog: `skills/sap-cap-upgrade/references/packages-catalog.md`

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
    └── sap-cap-upgrade/
        ├── SKILL.md
        └── references/          ← changelogs/, releases/, packages-catalog.md, ...
```

`SKILL.md` is the contract the agent reads. `references/` are the
capire-anchored rules each skill cites. `templates/` are the markdown
or JS files the skill writes into the user's project.

## License

[MIT](LICENSE) © 2026 Fabricio
