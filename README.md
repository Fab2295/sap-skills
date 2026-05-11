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
    └── sap-cap-code-review/
        ├── SKILL.md
        ├── references/
        └── templates/
```

`SKILL.md` is the contract the agent reads. `references/` are the
capire-anchored rules each skill cites. `templates/` are the markdown
or JS files the skill writes into the user's project.

## License

[MIT](LICENSE) © 2026 Fabricio
