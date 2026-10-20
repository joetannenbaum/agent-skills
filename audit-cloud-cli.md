---
name: Audit Cloud CLI
description: Audit the Laravel Cloud CLI against the Cloud API OpenAPI spec and report gaps — missing endpoints, deprecated endpoints still in use, changed params, DTO drift. Read-only.
---

Audit the Cloud CLI (look at the current directory to know which flavor I'm asking you to audit) on an up-to-date `main` against the live Laravel Cloud API spec and report every gap. **Read-only — change nothing, commit nothing, open no PR.** The output is a report.

## 1. Get a clean main tree

The current directory is my working checkout and is often dirty or on a PR branch. Never switch its branch, stash, or touch anything in it. Audit a fresh clone instead:

```bash
git clone --depth 1 https://github.com/laravel/cloud-cli-{flavor}.git ~/Dev/clones/cloud-cli-{flavor}-audit
```

If that directory already exists, and it's clean and on an up-to-date `main`, reuse it (`git pull`). Otherwise clone to a new suffixed directory. Audit from there and leave it in place — no need to delete it.

Tell me which directory you audited.

## 2. Get the spec

The whole API is one OpenAPI document — fetch it, don't scrape the docs pages:

```bash
curl -sSL https://cloud.laravel.com/api-docs/api.json -o /tmp/cloud-api.json
```

Human-readable pages are at `https://laravel.com/cloud/docs/api/<group>/<page>.md`, indexed at `https://cloud.laravel.com/docs/llms.txt`. Use those only to explain something the spec leaves ambiguous.

If the spec fetch fails, stop and tell me — don't audit against memory.

## 3. Build both inventories

From the spec, every operation with its deprecation flag:

```bash
jq -r '.paths | to_entries[] | .key as $p | .value | to_entries[]
  | select(.key | IN("get","post","patch","put","delete"))
  | "\(.key|ascii_upcase)\t\($p)\t\(.value.operationId)\t\(if .value.deprecated == true then "DEPRECATED" else "-" end)"' \
  /tmp/cloud-api.json | sort -k2
```

From the CLI, every endpoint the SDK calls:

```bash
grep -rhoE 'return "[^"]+"' --include='*.php' app/Client/Resources | sed 's/return "//; s/"$//' | sort -u
```

The HTTP verb is the `protected Method $method = Method::X;` line in the same request class. CLI paths interpolate PHP properties (`/applications/{$this->applicationId}`); normalize both sides to a comparable shape before diffing.

Where things live: request classes in `app/Client/Resources/<Group>/`, bodies in `app/Client/Requests/*RequestData.php` (constructor args plus the `toRequestData()` array), responses in `app/Dto/`, and the method-per-endpoint facades in `app/Client/Resources/*Resource.php`.

## 4. What to look for

- **Missing endpoints** — in the spec, no request class in the CLI. Note whether it's worth a command or just SDK coverage.
- **Deprecated endpoints still called** — spec `deprecated: true`. The endpoint's description usually names the replacement.
- **Dead endpoints** — the CLI calls a path the spec no longer has.
- **Body params** — diff `components.schemas.<X>Request.properties` against the matching `*RequestData` class. Flag fields in the spec the CLI never sends, fields the CLI sends that the spec doesn't accept, and required-ness that disagrees (`required` in the spec vs. nullable-with-default in the constructor).
- **Path and query params** — missing `filter[...]`, `include`, pagination or sort params the spec offers.
- **Enums** — spec enum values vs. the matching `app/Enums/` case list, both directions.
- **Response DTO drift** — spec response schema properties vs. the DTO's promoted properties. Missing fields mean `--fields` and `--json` can't surface them.
- **Types** — string where the spec says integer, missing nullability, wrong array shape.

## 5. Report back

Group by resource. Per finding give: the endpoint or field, what the spec says, what the CLI does, and the file to change. Rank by what actually bites a user — a deprecated endpoint or a missing required param beats an unused optional filter.

Lead with a one-line count (`X missing endpoints, Y deprecated in use, Z param mismatches`), then the detail. Skip anything you couldn't confirm in both sources; say what you couldn't check rather than guessing.

Feel free to cross reference (or check out in a tmp dir a fresh `production` branch) ~/Herd/cloud

You should only report back anything you are 100% certain on, if you are at all uncertain say so clearly.
