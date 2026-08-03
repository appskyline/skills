---
name: appskyline
description: Query and manage App Store Optimization data with the `appskyline` CLI — check where an app ranks for a keyword on the iOS App Store, macOS App Store, Google Play or Microsoft Store, list live top-N search results, track keywords, look up monthly search volume and difficulty for keyword ideas, audit and upload App Store Connect listing metadata, and pull Google Play reviews and install reports. Use when asked about app store rankings, keyword research, ASO reports, store-listing metadata, or anything involving the `appskyline` command or app.appskyline.com data.
license: Apache-2.0
compatibility: Requires Node.js 18+, network access, and an AppSkyline account (https://app.appskyline.com)
metadata:
  author: Appskyline
  version: '1.1.0'
  repository: https://github.com/appskyline/skills
allowed-tools: Bash(appskyline:*) Bash(npx appskyline:*)
---

# AppSkyline CLI

`appskyline` reads and manages App Store Optimization data on
[app.appskyline.com](https://app.appskyline.com): tracked apps and keywords,
live store search results and rankings across four stores, keyword volume and
difficulty, store-listing metadata, and Google Play reviews and install
reports.

Run it as `appskyline` when installed globally (`npm install -g appskyline`)
or as `npx appskyline` otherwise. `appskyline --help` and
`appskyline <command> --help` are accurate and self-sufficient; prefer them
over memory when unsure of a flag. `appskyline skills list` names the guides
bundled with the installed version, and `appskyline skills get appskyline`
prints the copy of this guide matching it.

## Setup

Two ways to authenticate; both persist in `~/.appskyline/`:

```bash
appskyline login               # interactive: choose browser or API key
appskyline login --browser     # opens app.appskyline.com to authorize; stores session tokens
appskyline login --with-key    # masked prompt for an API key secret; verified before storing
appskyline logout              # clears the stored session and any stored key
```

Headless agents skip `login` entirely: export `APPSKYLINE_API_KEY=<key secret>`
and every command authenticates with it (create keys per app in the AppSkyline
web UI; new keys default to read-only scopes). Precedence: a stored browser
session wins over a stored key, which wins over the environment variable. The
secret is never accepted as a command argument — prompt or environment only.

Browser sessions refresh automatically. Without a terminal and without
credentials, commands fail fast with exit code 1 and instructions on stderr —
nothing ever opens a browser headlessly. A `401` means the session or key is
missing, expired, or wrong: run `appskyline login` (or fix
`APPSKYLINE_API_KEY`) and retry.

## Where to start

| You want                                   | Run                                    |
| ------------------------------------------ | -------------------------------------- |
| The apps in this account, with their ids   | `appskyline apps list`                 |
| Where an app ranks for a term              | `appskyline rank …`                    |
| Who occupies the top N for a term          | `appskyline search <store> …`          |
| The tracked keyword set for an app         | `appskyline keywords list --appId …`   |
| Search volume / difficulty for term ideas  | `appskyline keywords overview …`       |
| The store listing text (name, keywords, …) | `appskyline ios-app-store metadata …`  |
| A competitor's public listing              | `appskyline ios-app-store app <id>` (or the google-play / macos-app-store / microsoft-store equivalent) |

Almost everything needs an `appId` — the AppSkyline app id from
`apps list`, not a store-native id. Store-native ids (numeric Apple id,
Google Play package name, Microsoft Store product id) appear only in
`search` results, `rank` matching, and the per-store `app` lookups.

## Stores

Four store values are accepted wherever a store is named. Use them exactly:

`ios-app-store` · `macos-app-store` · `google-play` · `microsoft-store`

## Apps and keywords

```bash
appskyline apps list                        # id, name, per-store ids (--json for raw)
appskyline apps get <appId>                 # raw JSON for one app
appskyline apps read <appId>                # formatted single-app view

appskyline keywords list --appId <id>       # tracked terms: id, term, language, country
appskyline keywords get <keywordId>
appskyline keywords add --appId <id> --term "dental software" --lang en --country US
appskyline keywords update <keywordId> --body '{"bid":1.5}'
appskyline keywords delete <keywordId>      # variadic: delete <id1> <id2> …
appskyline keywords translate --term "dental software" --locales "it,es,de" --appId <id>
```

`keywords add` is not idempotent — adding the same term twice tracks it twice.
List first and check the term is not already tracked for that language and
country. Deleting stops future rank snapshots and the gap cannot be
backfilled; confirm the user wants it and resolve the term to its keyword id
via `keywords list` rather than guessing.

## Rankings

```bash
appskyline rank <storeAppId> --store ios-app-store --term "dental software" --country US --lang en
appskyline rank com.example.app --store google-play --term "crm" --country US -n 50
```

`<storeAppId>` is the store-native id; matching is case-insensitive. The
output reports a 1-indexed position within the scanned window — "#6 of 25
scanned" is honest, "#6" alone is not, so keep the scan depth in your wording.
"Not in top N" means genuinely absent from that window; retry with a larger
`-n` (up to 200) before concluding an app does not rank. A single rank is a
point sample of a noisy series — do not call a change a regression or a win
from one reading.

## Live search results

```bash
appskyline search ios-app-store --term "restaurant pos" --country US --lang en -n 10
appskyline search macos-app-store --term "invoice" --country US -n 10
appskyline search google-play --term "dental software" --country US --lang en -n 10
appskyline search microsoft-store --term "pos" --country US -n 10
```

Each result row carries title, rating, developer, and the store-native id —
the competitive field for a term, useful before and after metadata changes.

## Keyword volume and difficulty — batch it

```bash
appskyline keywords overview --country US --terms "dental software,dentist app,clinic software"
```

Returns monthly search volume, CPC, competition, and difficulty per term.
Terms missing from the shared cache trigger billable upstream lookups, so
never loop this command over single keywords: collect the full candidate list
first, then make one call per country with up to 100 comma-separated terms.
Rank without volume is vanity — check volume before recommending a term.

## Store-listing metadata (App Store Connect)

```bash
appskyline ios-app-store metadata download <appId>   # pull fresh from App Store Connect
appskyline ios-app-store metadata list --appId <id>  # read the cache written by download
appskyline ios-app-store metadata upload <appId> --path ./fastlane/ios \
  --platform IOS --locale en-US --only keywords --dry-run
```

`list` reads a cache that can be days stale — run `download` first and never
infer the current listing state, version editability, or live keywords from
`list` alone. Covers iOS and macOS (they share App Store Connect): select
with `--platform IOS` or `--platform MAC_OS`, or the app's editable version
list decides for you — often wrongly, so pass it. `upload` walks
fastlane-shaped `<path>/metadata/<locale>/*.txt` files (`name.txt`,
`subtitle.txt`, `keywords.txt`, `description.txt`, `release_notes.txt`, …);
use `--only` to push a single field, `--skip-app-info` when only the version
draft is editable, and `--dry-run` to preview the payload. Requires an App
Store Connect API key configured for the app in the AppSkyline web UI. Apple's
keyword field caps at 100 characters — unused characters are wasted ranking
surface.

Diagnostics from the same API key:

```bash
appskyline ios-app-store diagnostics <appId> --platform IOS   # hangs, disk writes, launches
appskyline ios-app-store crashes <appId> --platform IOS       # TestFlight crash feed
```

## Competitor lookups (public listings, by store-native id)

```bash
appskyline ios-app-store app <appleId> --country US --lang en
appskyline macos-app-store app <appleId> --country US
appskyline google-play app <packageName> --country US --lang en
appskyline microsoft-store app <productId> --country US
```

The `searches` sibling reports which of your tracked terms an app appears in:

```bash
appskyline ios-app-store searches --id <appleId> --country US
appskyline macos-app-store searches --id <appleId> --country US
appskyline microsoft-store searches --id <productId> --country US
```

## Google Play

```bash
appskyline google-play reviews <appId> --max 50                 # paginated; pass --token from the previous page
appskyline google-play installs <appId> --month 202607 --dimension country
```

`reviews` and `installs` require a Google Play service account configured for
the app in the AppSkyline web UI; without one they fail with an upstream
permission error, which is setup, not a bug.

## Team management

```bash
appskyline users list
appskyline users get <userId>
appskyline roles list
appskyline roles add --userId <id> --type admin
appskyline roles delete <roleId>
appskyline invites list
appskyline invites add --email x@y.com --type admin
appskyline invites delete <inviteId>
appskyline zapier-hooks add --url https://hooks.example.com --event KEYWORD_ADDED
appskyline zapier-hooks delete <hookId>
```

## Output discipline

Every data subcommand accepts `--json`: real, parseable JSON on stdout — no
colors, no ANSI escapes. Without the flag, output is for human reading.
Mutations with `--json` print a small result object
(`{ "ok": true, "id": … }`). Errors always go to stderr with exit code 1;
in `--json` mode the error is a single JSON line
(`{"error":{"message":…,"status":…}}`), so stdout stays parseable.

`appskyline schema` prints the whole command tree (options, arguments,
subcommands) as JSON — discover capabilities from it instead of scraping
help text:

```bash
appskyline schema                   # the full command tree as JSON
appskyline schema keywords list     # one subtree only
```

When exploring, project instead of dumping — pipe through `jq` and keep only
the fields you need:

```bash
appskyline apps list --json | jq '.[] | {id: ._id, name, iosAppStoreId}'
```

## Configuration

A `appskyline.json` in the working directory (or any parent) supplies the
default `--appId`:

```json
{ "appId": "…" }
```

`APPSKYLINE_API_URL` overrides the API endpoint — only needed when testing
against a non-production deployment. `APPSKYLINE_API_KEY` supplies an API key
secret for headless auth (see Setup).

## Errors → remediation

| Symptom                              | Fix                                                        |
| ------------------------------------ | ---------------------------------------------------------- |
| `401` on any command                 | `appskyline login` (or set `APPSKYLINE_API_KEY`), then retry |
| `403` with an API key                | Key scopes too narrow — widen them in the AppSkyline web UI, or log in |
| App not found / empty keyword list   | Wrong id kind — get the AppSkyline `appId` from `apps list` |
| `metadata list` looks outdated       | It is a cache — run `metadata download <appId>` first      |
| `reviews` / `installs` permission error | Service account not configured in the AppSkyline web UI |
| Rank "not in top N" unexpectedly     | Re-run with `-n 200`; the app may rank deeper              |
