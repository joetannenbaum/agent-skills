---
name: Sync Cloud CLI JS from PHP
description: Catch the TypeScript Cloud CLI (cloud-cli-js) up with changes that landed in the PHP Cloud CLI (cloud-cli). Works out what each PHP change accomplished and rebuilds it the JS repo's way, never as a line-by-line port. Use when I ask to sync, catch up, or port PHP CLI changes into the JS CLI.
---

The PHP CLI (`~/Dev/cloud-cli`) ships first; the TypeScript CLI (`~/Dev/cloud-cli-js`) has to catch up. They share a command surface and an API, but nothing else: different frameworks, layouts and rules. Your job: find every PHP change since the last sync, work out what each one accomplished for a user, and get the JS CLI to the same place by following `cloud-cli-js/CLAUDE.md`. Copying the PHP shape is the wrong answer even when it would work.

Work in `~/Dev/cloud-cli-js`. Read its `CLAUDE.md` in full before writing any code; it is the rulebook and most of its rules came from breaking them.

## 1. Check both trees

**JS repo.** Run `git status`. If it is dirty, stop and tell me what is modified. It may be a catchup already in progress, and I decide whether to build on it. Never stash, reset or switch branches.

**PHP repo.** My checkout is usually on a feature branch and often dirty. Never switch its branch, stash, or touch it. Read it through git only:

```bash
git -C ~/Dev/cloud-cli fetch origin
```

The source of truth is `origin/main`. Read files with `git -C ~/Dev/cloud-cli show origin/main:<path>`, never from the working tree. If the current PHP branch has commits not yet on `origin/main`, list them in the report but don't port them unless I ask.

## 2. List the PHP PRs

Take every PR merged into the PHP repo's `main` in the last two weeks:

```bash
gh pr list --repo laravel/cloud-cli --base main --state merged \
  --search "merged:>=$(date -v-14d +%Y-%m-%d)" --limit 100 \
  --json number,title,mergedAt,mergeCommit --jq 'sort_by(.mergedAt)[]'
```

Some of these will already be in JS from an earlier catchup; step 4 sorts that out. Get the intent from each PR, not just the diff:

```bash
gh pr view <n> --repo laravel/cloud-cli --json title,body,files
```

Skip dependency bumps and release PRs without comment.

## 3. Refresh the spec first

Many PHP changes follow an API change (presets, sizes, types, new fields). Bring the vendored spec current before porting, so the compiler tells you what moved:

```bash
pnpm run spec:drift
pnpm run api:fetch && pnpm run api:generate
```

Read what `openapi/normalize.mjs` prints. A workaround that went quiet means upstream fixed it; say so, and remove the workaround if `CLAUDE.md`'s spec section calls for it. Delete entries in `src/api/undocumented.ts` that the spec now covers. Type errors after regenerating are leads, not noise: each names a command and a field.

## 4. Triage each PHP change

For every commit, write down the behaviour it changed as a user would see it: the command, the input, what happened before, what happens now. Not a summary of the patch.

Then sort it into one of these:

- **Already in JS.** Find the code that covers it and name it. Search for the behaviour, not the PHP class names. The JS repo has often reached the same place by another route.
- **Needs porting.** The behaviour is missing or different in JS.
- **Doesn't apply.** PHP-only: Laravel Prompts renderers, Saloon plumbing, composer or phpstan, Pest helpers, PHP release and version bumps, `CHANGELOG.md` updates. Say why in one line.
- **Conflicts with a JS rule.** The PHP behaviour breaks something `CLAUDE.md` pins down (the `--json` contract, a quirk kept on purpose, a design decision). Don't port it. Flag it for me.

Show me the triage table before writing code when there are more than a handful of changes, or when anything is in the last bucket.

## 5. Port the behaviour

Rebuild each change the JS way. Use this map to find where the equivalent lives, then follow `CLAUDE.md` for how it should be built there:

| PHP | JS |
| --- | --- |
| `app/Commands/<Thing><Action>.php` | `src/commands/<thing>/<action>.ts` (no topic: `src/commands/<name>.ts`) |
| `app/Client/Resources/*`, `app/Client/Requests/*RequestData.php` | the typed client in `src/api/` against generated `schema.ts` |
| `app/Dto/*` | `src/models/*` |
| `app/Resolvers/*` | resolvers beside each resource, `src/core/resolve*.ts` |
| `app/Support/Form.php` | nib's `createForm` |
| `app/Prompts/*` | nib components, or Cloud's views in `src/ui/` |
| `app/Enums/*` | spec enums emitted as `*Values` arrays in `schema.ts`, or an endpoint if one serves the list |
| `skills/deploying-laravel-cloud/` | `skills/deploying-to-cloud/` |
| `tests/Feature/*Test.php` | `test/feature/*.test.ts` via `runCloud` |

The PHP CLI is going away and this one will be the only CLI, so leave no trace of the sync. Nothing in the JS repo mentions the PHP CLI, a port, or a PHP PR: not code comments, `CLAUDE.md`, `CHANGELOG.md`, test names or commit messages. Write each change as if it started here.

Points where the two repos differ most:

- **Wording and flow follow JS rules.** Copy rules, resolver labels, the timeline and the `spin` messages all come from `CLAUDE.md`. Changing what a command asks, in what order, or what it defaults to is a design decision; if the PHP change does that, say so before adopting it.
- **The `--json` shape is a contract.** A new field means updating `contract/json-fields.json` and the model's `fields`. A removed or renamed field, or a changed value, goes under **Breaking** in `CHANGELOG.md`'s Unreleased section, written like the entries already there. A new command or flag needs no changelog line unless it changes existing output.
- **Request bodies come from the spec types**, not from the PHP `toRequestData()` array. If the PHP body disagrees with the spec, the spec usually wins. If the API really does disagree with the spec, fix it in `normalize.mjs` as `CLAUDE.md` describes.
- **New commands** need a feature test (`audit:commands` fails otherwise) and need to work headlessly.
- **Skill docs**: port the content change into `skills/deploying-to-cloud/`, rewritten to match that skill's own structure and command list.
- **Update `CLAUDE.md`** when a port adds a rule, removes an exception, or makes a listed quirk obsolete.

## 6. Port the tests

Port what each PHP test proves, not its code. Keep its input and rewrite the assertions for `runCloud`, asserting on the request bodies sent as well as the JSON emitted. Use real id prefixes from `src/core/id-prefixes.ts` and fixture shapes from the spec or a real response. A test that passes before your change proves nothing; check it fails without the port where you can.

## 7. Verify

```bash
pnpm run fix
pnpm run verify
pnpm run audit
```

Everything must pass. Never edit `contract/json-fields.json` just to quiet the ratchet: a field that stopped publishing is a regression unless it is a deliberate, logged breaking change.

## 8. Report

Give me:

- The triage table: each PHP change, its bucket, and for ported ones the JS files that changed
- Anything flagged for a decision, and why
- Spec changes and `normalize.mjs` warnings that appeared or went quiet
- PHP commits on unmerged branches that you skipped
- The `verify` and `audit` results

Don't commit unless I ask.
