---
name: Wayfinder Main to Next
description: Port a fix or feature that just landed in laravel/wayfinder `main` over to the `next` branch. Checks whether `next` already has it, and if not, preps a clean branch with a reproducing test. Use when a wayfinder PR merges to main, or when I ask whether something needs porting to next.
---

A fix landing in `main` usually needs to land in `next` too, but `next` is a rewrite, so it is never a cherry-pick. Your job: work out what the `main` PR fixed, find out whether `next` still has that bug, and if it does, prep a branch with a failing test.

## 1. Identify what landed in `main`

Start from whatever I gave you. With no argument, use the current branch.

```bash
gh pr list --repo laravel/wayfinder --head "$(git branch --show-current)" --state all \
  --json number,title,headRefName,baseRefName,body,files,mergedAt
```

`gh pr view --repo laravel/wayfinder` errors out without a PR argument, and
`gh pr view` on its own only finds a PR once the branch has an upstream, so go
through `gh pr list --head`. If the branch has no PR, diff it against
`origin/main` and treat that as the change.

Pull out and tell me:

- PR number, title, head branch, and the issue it closes
- The behaviour that was wrong, stated as an input and a wrong output — not as a description of the patch
- The test files and workbench fixtures the PR added or touched

Confirm it is actually based on `main`. If `baseRefName` is already `next`, say so and stop.

## 2. Get a clean checkout on `next`

Never switch branches or stash in my working checkout — it is usually dirty and mid-task.

Reuse `~/Dev/clones/wayfinder-next` only if it is clean and on `next`. Otherwise clone fresh into `~/Dev/clones/wayfinder-next-<slug>`, where `<slug>` names the fix. Then fetch, hard reset to `origin/next`, and run `composer install` and `npm install`.

Tell me which directory you are in.

## 3. Find out whether `next` already has it

Cheap checks first — they can rule the port out, but a miss proves nothing:

```bash
gh pr list --repo laravel/wayfinder --base next --state all --search "<key terms>"
git log --oneline origin/next --grep "<key terms>"
```

`next` PRs that port a `main` fix are titled `[next] <same title>`, so a match is easy to spot.

The check that decides it is reproducing the bug. Port the `main` PR's test and workbench fixtures onto unmodified `next`:

- Tests are `tests/*.test.ts` (vitest), fixtures live in `workbench/` — routes in `workbench/routes/web.php`, controllers and models under `workbench/app/`. Test file names mostly match across branches, so start from the same-named file if there is one.
- `next` generates different output than `main`, so keep the test's *input* and rewrite its *assertions* against what `next` produces.

Run just that file with `npx vitest run tests/<Name>.test.ts`, then decide:

- **Passes on untouched `next`** — the rewrite already handles it. Report that, name the file and the code path in `next` that covers it, throw the scratch test away, and stop. Do not open a PR.
- **Fails** — the bug is live. Keep the test and go on.

## 4. Decide which repo the fix belongs in

`next` does not do its own route and code introspection. It leans on two packages, so the fix may not be a wayfinder fix at all:

- **[laravel/surveyor](https://github.com/laravel/surveyor)** — static analysis of PHP: classes, methods, parameters, return types, docblocks. Wrong or missing type information usually starts here.
- **[laravel/ranger](https://github.com/laravel/ranger)** — walks the app and hands back DTOs for routes, models, enums, broadcast events, env vars, and Inertia components. Wrong route data — URI, middleware, bindings, names — usually starts here.
- **wayfinder `next`** — turns those DTOs into generated TypeScript. Wrong output shape, naming, or file layout belongs here.

Trace the bad value back to where it enters wayfinder. If a Ranger DTO or a Surveyor result is already wrong when `next` receives it, fix it in that package and say so; a patch in wayfinder that papers over it is the wrong fix. Some changes need both — the package to report the right data, and wayfinder to use it.

Local checkouts are `~/Dev/clones/ranger` and `~/Dev/clones/surveyor`. Tell me which repo (or repos) you are targeting before you start writing the fix.

## 5. Prep the branch

Cut `<main-head-branch>-next` off `origin/next` with `--no-track`. That flag matters: basing off a remote-tracking branch otherwise sets the upstream to `origin/next`, and a later bare `git push` lands there. Push with `-u` as soon as there is a commit, so the branch has its own upstream of the same name.

Commit the failing test on its own, then write the fix. Re-derive it against `next`'s layout rather than translating the `main` diff line by line:

| `main` | `next` |
| --- | --- |
| `src/GenerateCommand.php` | `src/Console/GenerateCommand.php` |
| `src/TypeScript.php`, `resources/*.blade.ts` | `src/Langs/TypeScript/*` builders |
| — | `src/Converters/*`, `src/Registry/*`, `src/Results/*` |

The equivalent code in `next` is often in a different class at a different layer, and the `main` fix may be the wrong shape there. Say so if the right fix in `next` differs, and explain why.

## 6. Verify

Run `npm test`. `next` has more gates than `main`: phpunit, vitest, eslint, the cached-routes and inertia-component passes, and both tsc configs. Everything must pass. Never adjust a snapshot to match broken output.

## 7. Report

Give me:

- Whether the port is needed, and the evidence either way
- The directory and branch you prepped
- What changed in `next` and how it differs from the `main` fix
- The suite result

Do not open the PR unless I ask. When I do, base is `next` and head is the
`<main-head-branch>-next` branch from step 5. Title and description are:

```
[next] <same title as the main PR>

Bringing https://github.com/laravel/wayfinder/pull/<n> into `next`
```

The description is that one line and nothing else — the `main` PR already
explains the bug, so do not restate it, list the files, or paste the test.

[PR 319](https://github.com/laravel/wayfinder/pull/319) is the shape to copy:

```
[next] Key routes that share a URI by verb

Bringing https://github.com/laravel/wayfinder/pull/303 into `next`
```

When I ask for the title and description rather than for the PR, `pbcopy`
exactly that — title, one blank line, description. No `title:`/`base:`/`body:`
labels and no surrounding commentary.

If the `main` PR's own title is truncated — GitHub cuts it when it comes from a
long commit subject — use the full sentence and tell me the `main` title needs
fixing.
