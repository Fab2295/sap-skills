# Upgrade Checklist (canonical procedure)

The skill executes this exact sequence inside the project's working directory. Every step has an explicit purpose; the order matters because it isolates "version-caused" failures from pre-existing ones.

## Mode gate

The skill defaults to **plan** mode unless the invocation prompt explicitly authorizes apply mode (keywords listed in `SKILL.md` → "Modes"). Plan mode runs steps **0 → 2 → 7 (plan emit)** only — no edits, no install, no validation. Apply mode runs the full sequence below.

## 0. Preconditions

- `package.json` exists in `cwd`.
- `node` and `npm` resolvable in `PATH`.
- At least one in-scope package present (per `packages-catalog.md`). If not → emit `status:"no_changes"` and stop.

## 1. Capture baseline (BEFORE any bump) — apply mode only

Run, **without modifying anything**:

1. `npx --no-install cds build --production` (or `npx cds build` if `--production` not supported by the installed `@sap/cds-dk`).
2. If `package.json` has `scripts.test`, run `npm test -- --silent || true`.

Persist exit codes and stderr to memory as `baseline.failures[]` (one entry per non-zero command, with the captured stderr text). Do not write any files for this — it lives in the skill's working memory only.

If `cds build` is itself broken before any bump (baseline failure count > 0), all those errors are now part of `baseline.failures` and will **never** be reported as version-caused.

In **plan mode**, skip this step entirely — no commands run, no baseline captured.

## 2. Resolve target versions

For every dependency in `package.json` (under `dependencies`, `devDependencies`, `peerDependencies`, `optionalDependencies`) whose name matches the regex in `packages-catalog.md`:

```sh
npm view <pkg> dist-tags.latest
```

One call per package. Capture stdout. Aggregate into `{ "@sap/cds": "9.12.0", "@cap-js/sqlite": "2.6.0", ... }`.

In **plan mode**, this is the last operational step. Build the `bumped[]` proposal (only entries whose target differs from the current version, preserving the original range operator) and emit the plan JSON documented in `SKILL.md` → "Plan mode". Do not proceed past this point.

## 3. Apply bumps

Edit `package.json` in place. For each in-scope dep:
- Preserve the existing range operator (`^`, `~`, exact, or `>=`); replace only the numeric tail.
- If the current spec is a tag (`latest`, `next`) or a non-semver (URL, git+, file:), **skip** that dep and add a `notes` entry explaining why.
- Record `{name, from, to, major_jump: <bool>}` per bump.

Do **not** touch any other dependency, script, field, or formatting. Do not rewrite the file beyond the version strings.

## 4. Install

```sh
npm install --no-fund --no-audit
```

If exit code != 0 → emit `status:"install_failed"` with stderr in `notes` and stop. Do not roll back: the user owns the lockfile.

## 5. Re-validate (AFTER bump)

Run the same commands as step 1 and capture `post.failures[]`.

## 6. Diff + attribute

Compute `new_failures = post.failures \ baseline.failures` (set difference by error signature: command + first 200 chars of stderr).

Apply `bug-attribution-rules.md` to every entry in `new_failures`. Each attempted attribution either:
- produces a `version_caused_bug` entry (regex hit + version-crossing match), or
- goes to `discarded[]` with `reason`.

## 7. Emit JSON

Last message of the skill is the strict JSON object documented in `SKILL.md` ("Output (contract com o agente)"). No additional prose after the JSON.

## Forbidden actions

- `git add`, `git commit`, `git push`, `git checkout`, `git restore`, `git stash` — never.
- Editing source files (anything outside `package.json` and `package-lock.json`/`npm-shrinkwrap.json`/`yarn.lock` as side-effect of `npm install`).
- Calling the Senua skill — that is the coordinator agent's job.
- Running `npm publish`, `npm link`, `npm dedupe`, `npm prune --production`.
