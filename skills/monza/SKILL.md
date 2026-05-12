---
name: monza
description: |
  Upgrade SAP CAP and SAP-related libraries (@sap/cds*, @cap-js/*, @sap-cloud-sdk/*,
  @sap/eslint-plugin-cds) in the current working directory. Strategy: latest stable
  including majors. Runs `cds build` + `npm test` (when present), cross-checks failures
  against locally mirrored official changelogs (CAP + Cloud SDK JS) and reports ONLY
  bugs caused by the version bump.
  Use when the user asks to "upgrade CAP libs", "atualiza CAP", "bump @sap/cds", or
  similar. Read/upgrade only — never edits source code, never commits, never pushes.
license: GPL-3.0
metadata:
  version: "0.1.0"
  last_verified: "2026-05-08"
  sources:
    - "https://cap.cloud.sap/docs/releases/"
    - "https://sap.github.io/cloud-sdk/docs/js/release-notes"
---

# Monza — CAP Upgrade Skill

This skill performs ONE thing: bumps in-scope SAP/CAP packages in the current project's `package.json` to the latest stable version (including majors), runs the project's build + test commands, and emits a strict JSON report of bugs caused **specifically** by the bump.

It is project-agnostic — every operation runs against the current working directory. It is read/upgrade-only — it touches `package.json` and `package-lock.json` (the latter via `npm install`), nothing else. It never invokes Git, never edits source code, and never calls another skill.

## Related skills

- **sap-cap-capire (Senua)** — CAP development. The monza skill never invokes Senua directly; the coordinator agent (`cap-upgrade-coordinator`) does that based on the JSON this skill emits.

## Hard invariants

1. Source code (anything outside `package.json`/lockfile) MUST NOT be modified.
2. `git add`/`commit`/`push`/`checkout`/`restore`/`stash` MUST NOT be invoked.
3. A failure MUST NOT be reported as `version_caused_bug` unless it satisfies all three criteria in `references/bug-attribution-rules.md` (baseline diff + regex hit in an official changelog entry + version crossing). When in doubt, discard.
4. Only packages matching the regex in `references/packages-catalog.md` are bumped.
5. The skill's terminal message MUST be the strict JSON object documented below — no prose after.
6. Default mode is **plan** (read-only preview). Switch to **apply** mode ONLY when the invocation prompt explicitly contains one of: `apply`, `aplicar`, `confirm`, `confirmado`, `proceed`, `prosseguir`, `execute`, `executar`, `go`. In any other case, run plan mode.

## Modes

The skill has two modes. Pick the mode by inspecting the invocation prompt; default to plan when unclear.

### Plan mode (default — read-only)

Goal: preview the upgrade without touching anything.

Run only steps 0 + 1 (preconditions) + 2 (resolve target versions) of the migration checklist. **Do not** edit `package.json`. **Do not** run `npm install`. **Do not** capture baseline failures or run `cds build`/`npm test`. Just read `package.json`, identify in-scope deps, query npm for latest versions, and emit:

```json
{
  "skill": "monza",
  "status": "plan",
  "bumped": [
    { "name": "@sap/cds", "from": "^9.9.1", "to": "^9.12.0", "major_jump": false }
  ],
  "skipped": [
    { "name": "@cap-js/sqlite", "current": "next", "reason": "non-semver spec (tag)" }
  ],
  "notes": []
}
```

`bumped[]` here means *proposed*, not *applied*. `from` and `to` MUST include the original range operator (`^`, `~`, exact, etc.) so the user sees what will actually be written. If a package is already at latest, omit it from `bumped[]` (don't include zero-diff entries). If no in-scope deps exist or all are already at latest, emit `status: "no_changes"` instead of `"plan"`.

### Apply mode

Goal: actually perform the upgrade and validate.

Run the full migration checklist (steps 0–7). The terminal JSON uses `status: "ok" | "no_changes" | "install_failed" | "build_failed_unrelated"` — never `"plan"`.

## Bundled resources

- `references/source.md` — canonical upstream URLs + last_fetched per source.
- `references/packages-catalog.md` — in-scope regex + per-family routing table.
- `references/migration-checklist.md` — exact upgrade procedure (steps 0–7).
- `references/bug-attribution-rules.md` — strict A∧B∧C criteria + blacklist.
- `references/changelogs/cap/changelog-<YYYY>.md` — mirrors of CAP yearly changelogs.
- `references/changelogs/cloud-sdk-js/changelog-v<N>.md` — mirrors of Cloud SDK JS per-major release notes.
- `references/releases/<YYYY>/<mon><YY>.md` — optional CAP per-month detail mirrors.

> The companion helper scripts (`latest-versions.js`, `refresh-references.js`) are NOT bundled with this distribution. The skill calls `npm` directly instead — see step 3 of the workflow and the "Refresh references when needed" section below for the exact commands.

Read these in this order before doing anything: `migration-checklist.md` → `packages-catalog.md` → `bug-attribution-rules.md`. The first defines the workflow; the second decides what to touch; the third decides what to report.

## Workflow (summary)

Follow `references/migration-checklist.md` literally. Plan mode runs steps 0–2 only; apply mode runs all of them.

1. **Preconditions** — `package.json` exists; `node`/`npm` resolvable; at least one in-scope dep present (otherwise emit `status:"no_changes"` and stop).
2. **Capture baseline** _(apply mode only)_ — run `npx --no-install cds build --production` (fall back to `npx cds build` if `--production` flag unsupported) and `npm test` (only if `scripts.test` exists). Persist failures in working memory; do NOT write any file.
3. **Resolve target versions** — for each in-scope dep, run `npm view <pkg> dist-tags.latest` (one call per package; capture stdout). The skill MUST NOT use `npm view` with wildcards or fields that hit the registry more than necessary. **Plan mode stops here and emits the plan JSON.**
4. **Apply bumps** _(apply mode only)_ — edit `package.json` in place, preserving range operators (`^`, `~`, exact). Skip non-semver specs (tags, URLs, git+, file:) and log them in `notes`.
5. **Install** _(apply mode only)_ — `npm install --no-fund --no-audit`. On non-zero exit, emit `status:"install_failed"` and stop.
6. **Re-validate** _(apply mode only)_ — repeat step 2 commands; capture post-bump failures.
7. **Diff + attribute** _(apply mode only)_ — apply A∧B∧C from `bug-attribution-rules.md` to every new failure. Producers go to `version_caused_bugs[]`; everything else goes to `discarded[]`.
8. **Emit JSON** — final terminal message is the contract below (plan or apply shape, depending on mode).

## Identifying in-scope packages

Use the regex from `references/packages-catalog.md`:

```
^(@sap/cds(-.*)?|@cap-js/.+|@sap-cloud-sdk/.+|@sap/eslint-plugin-cds)$
```

Inspect `package.json` keys under `dependencies`, `devDependencies`, `peerDependencies`, and `optionalDependencies`. For each match, record the original spec and the routing target (CAP changelog or Cloud SDK JS changelog) per the catalog's routing table.

## Bug attribution

A failure becomes a `version_caused_bug` ONLY when all three hold:

- **A. Baseline diff** — present post-bump, absent pre-bump (signature = command + first 200 chars of normalized stderr).
- **B. Regex hit** — error text matches a regex extracted from a concrete entry in the routed changelog mirror, in a section that denotes incompatible change (`Changed`/`Removed`/`Fixed`/`Breaking Changes`/`Migration` for CAP; `Compatibility Notes` for Cloud SDK JS).
- **C. Version crossing** — the bumped package's `from→to` interval includes the version of the matched entry.

Anything failing one of A/B/C goes to `discarded[]` with `reason`. Anything in the blacklist (`bug-attribution-rules.md` §"Mandatory blacklist") is always discarded.

## Output contract

The terminal message of this skill — and ONLY the terminal message — is one strict JSON object:

```json
{
  "skill": "monza",
  "status": "ok | no_changes | install_failed | build_failed_unrelated",
  "bumped": [
    { "name": "@sap/cds", "from": "9.9.1", "to": "9.12.0", "major_jump": false }
  ],
  "version_caused_bugs": [
    {
      "file": "<repo-relative path>",
      "line": 142,
      "error": "<captured error excerpt>",
      "rule_id": "<source>#<entry-anchor>",
      "from": "9.9.1",
      "to": "9.12.0",
      "fix_hint": "<one-line hint extracted from the changelog entry>",
      "ref": "references/changelogs/<source>/<file>.md#<entry-anchor>"
    }
  ],
  "discarded": [
    { "error_excerpt": "<…>", "reason": "unmatched | matched non-breaking section | version not extractable from rule | blacklisted: <subrule> | ambiguous source" }
  ],
  "baseline_failures_count": 0,
  "post_bump_failures_count": 0,
  "notes": []
}
```

Field rules:

- `status: "ok"` — at least one bump applied AND validation completed (regardless of whether bugs were attributed).
- `status: "no_changes"` — no in-scope deps in `package.json`, OR `npm view <pkg> dist-tags.latest` resolved no newer version for any of them.
- `status: "install_failed"` — `npm install` returned non-zero. `notes[0]` MUST contain the exact stderr (truncated to 4 KB).
- `status: "build_failed_unrelated"` — post-bump build/test failed but no failure satisfied A∧B∧C, AND `discarded[].length >= 5`. Use `notes` to add `"high discard count — consider refreshing references/ from the upstream URLs listed in references/source.md"`.
- `bumped[]` may be empty when `status` is `no_changes`.
- `version_caused_bugs[].rule_id` MUST anchor to a heading present in the cited mirror file. If the anchor cannot be derived, the entry MUST be discarded instead.
- `notes[]` is for advisory text only — never put bugs there.

Do NOT print explanatory prose before, after, or interleaved with the JSON. The coordinator agent parses the last assistant message verbatim.

## Refresh references when needed

If the upgrade target is a version newer than any entry in the relevant mirror, OR `references/source.md` shows the source's `last_fetched` is older than 30 days, the skill MUST stop and surface a request for a manual refresh — it does NOT fetch upstream content on its own in this distribution.

Manual refresh procedure (run by the user):

1. Open `references/source.md` and copy the canonical URLs for the affected source (CAP yearly changelog, Cloud SDK JS per-major release notes, or CAP monthly release page).
2. Fetch each URL with curl/wget or a browser export and overwrite the corresponding mirror file under `references/changelogs/...` or `references/releases/...`.
3. Update `last_fetched` in `references/source.md` to today's date.

The skill writes mirrors only when explicitly told to during refresh; otherwise, refresh is the user's call. Refresh — when it happens — must occur before step 7 (attribution), never before step 1 (baseline capture), so a refresh doesn't change baseline semantics mid-run.

## What this skill never does

- Does not invoke `Skill` for `sap-cap-capire` or any other skill.
- Does not write files outside `package.json` and `package-lock.json` (the latter via `npm install`). Mirror files under `references/` are refreshed manually by the user — the skill does NOT fetch upstream content on its own.
- Does not run dev servers, generators (`cds add`, `cds init`), code-mods, or formatters.
- Does not interpret `notes[]` as actionable bugs.
- Does not "soft-report" suspicions — every entry in `version_caused_bugs[]` is a strict A∧B∧C hit.
