---
name: aso-research
description: Research and improve an app's App Store Optimization standing using AppSkyline — keyword ranks and history across the iOS App Store, macOS App Store, Google Play and Microsoft Store, search-volume and difficulty lookups, suggested titles, subtitles and keyword fields built from the tracked keywords, store-listing metadata audits against Apple's character limits, competitor listings, and engagement/review signals. Use when asked for an ASO report, to check where an app ranks for a keyword, to find keywords worth targeting, which keywords to put in a title or subtitle, to audit or improve store metadata, or to compare an app against competitors.
license: Apache-2.0
compatibility: Requires the AppSkyline MCP connector (https://mcp.appskyline.com/mcp) and an AppSkyline account with at least one tracked app
metadata:
  author: Appskyline
  version: '1.2.0'
  repository: https://github.com/appskyline/skills
---

# ASO research with AppSkyline

This guide drives the AppSkyline MCP connector. Every tool below is provided by
that connector, so the user must have connected AppSkyline and be signed in to
an account with access to at least one tracked app. If a tool call fails with an
authorization error, tell the user to connect AppSkyline rather than retrying.

## Interactive workspace

Use `open_appskyline` with no arguments when the user wants to open the product
and browse visually. It is a read-only sidebar entrypoint with up to 100 items,
using the same tenant authorization as the other tools. Selection shares bounded
entity context with the host; subsequent writes still require their normal approval.

## Stores

Four stores are supported everywhere a `store` parameter appears. Use these
exact values:

`ios-app-store` · `macos-app-store` · `google-play` · `microsoft-store`

An app only has data for the stores it is registered on. When a tool returns
`store-not-configured`, the app has no id set for that store — say so and move
on rather than retrying with different parameters.

## Identifiers: two different things

Getting these confused is the most common source of empty results.

- **`appId`** is the AppSkyline app id (a uuid). Every tool that reads _your_
  data takes this: `get_app`, `list_keywords`, `get_keyword_rank`,
  `get_keyword_history`, `get_store_metadata`, `get_app_engagement_summary`,
  `list_google_play_reviews`, `add_keyword`.
- **`storeId`** is the store-native identifier: a numeric Apple id, a Google
  Play package name like `com.example.app`, or a Microsoft Store product id
  like `9wXXXXXXXX`. Only `get_store_listing` takes this, because it reads
  public listings for apps that need not be tracked in AppSkyline at all.

Start from `list_apps` when you do not yet have an `appId`. It returns each
app's id, name, default locale, and per-store ids.

## Available tools

Reading your own data:

| Tool                         | Use it when someone asks…                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------------------- |
| `list_apps`                  | which apps they track; also resolves the app id the other tools need                      |
| `get_app`                    | about one app's store ids, locales and timestamps                                         |
| `list_keywords`              | which keywords they track: id, term, language, country, bid                               |
| `get_keyword_rank`           | where their app ranks right now for one term in one store and country                     |
| `get_keyword_history`        | how that rank changed over time, one row per stored search snapshot                       |
| `get_store_metadata`         | to audit their synced listing text per locale, or whether the latest version is in review |
| `suggest_store_metadata`     | which keywords to put in a title, subtitle, keyword field or short description, per app   |
| `get_app_engagement_summary` | how many downloads, installs, uninstalls or crashes over a period, plus top countries     |
| `list_google_play_reviews`   | what Google Play users said in the last 7 days: stars, text, version, reply state         |
| `show_app_overview`          | for a one-card overview of one app, its stores and up to 50 tracked keywords              |

Reading public or shared data:

| Tool                   | Use it when someone asks…                                                        |
| ---------------------- | -------------------------------------------------------------------------------- |
| `search_store_results` | which apps rank for a term right now — the competitive field                     |
| `get_keyword_overview` | how popular terms are: Google web-search volume, difficulty, CPC, up to 50 terms |
| `get_store_listing`    | about any app's public listing by store-native id — competitor research          |

Writing:

| Tool             | Use it when someone asks…                                        |
| ---------------- | ---------------------------------------------------------------- |
| `add_keyword`    | to start tracking a term, with a country. Non-idempotent — below |
| `delete_keyword` | to stop tracking a term, by keyword id. Idempotent               |

## Batch `get_keyword_overview`; never loop it

This tool accepts up to 50 terms per call and is the one place where careless
usage costs real money. Terms missing or stale in AppSkyline's shared cache
trigger a live billable upstream lookup. Calling it once per keyword turns one
cheap request into dozens of billable ones.

Always collect the full candidate list first, then make a single call per
country. Note that `country` is required here and must be exactly two letters,
unlike most other tools where it is optional.

The volume is Google web-search demand in that country: a proxy for interest
in a term, not a count of App Store, Google Play or Microsoft Store searches.
Say so when you report it.

## Writes need the user's intent, not just their question

`add_keyword` and `delete_keyword` change the user's tracked set.

`add_keyword` is **not idempotent** — adding the same term twice creates two
rows. Before adding, call `list_keywords` and check the term is not already
tracked for that language and country. Pass a `country` with every new
keyword: the weekly rank check skips keywords that have none. When adding
several keywords, show the list and get agreement first; do not infer a batch
add from a research question.

`delete_keyword` stops future snapshots but leaves historical search data
intact, so it is recoverable in the sense that past rows survive — but the
tracking gap it creates is not backfillable. Confirm before deleting, and use
`list_keywords` to map a term the user named to its keyword id rather than
guessing.

## Suggesting title, subtitle and keywords

When someone asks which keywords to put in their title or subtitle, what to
change in a listing, or how to improve their metadata, call
`suggest_store_metadata` before answering, and answer with concrete text for
each app rather than a method. Without `appId` one call covers every app,
each in its main market, so it also answers "for each app". Each result
compares the listing with the app's tracked keywords in that market: the
current fields, drafts that already fit the store limits (App Store app name,
subtitle and keyword field; Google Play title, plus the phrases its short
description should mention; Microsoft Store search terms), how many tracked
keywords are covered now and after the drafts, the top opportunities with
their stored rank and cached Google demand, and issues.

The drafts place words; they are not finished copy. Rewrite a draft that
reads like a list into natural text in the listing's language and keep the
brand, then call `suggest_store_metadata` again with `appId`, `store` and
your proposed `title`, `subtitle`, `keywords` or `shortDescription`: the check
reports the tracked keywords gained and lost and any limit it breaks. Show
only wording that passes.

- On the App Store a word counts once across the app name, subtitle and
  keyword field. Never repeat one, and separate keywords with bare commas.
- On Google Play, work the phrases into a readable short description; never
  stuff keywords.
- `competitorWords` are words other apps put in their titles. Suggest one only
  when it describes the user's app, and never a brand or another app's name.
- Demand comes from the shared cache. When the top opportunities have none,
  call `get_keyword_overview` once for those terms in that market, then
  suggest again.
- Status `listing-unavailable` means the app is not live in that storefront,
  `no-tracked-keywords` means the app needs tracked keywords first, and
  `deferred` means ask again for that app alone.

## Reading rank results correctly

`get_keyword_rank` returns a `status` that must drive your wording:

- `ranked` — the app appears; `rank` is its position and `resultsScanned` is how
  deep the scan went. "#6 of 10 scanned" is honest; "#6" alone implies a
  ranking universe that was not measured.
- `not-ranked` — the app is genuinely absent from the scanned window. Say "not
  in the top N", not "rank unknown". Widening `num` (up to 200) can find an app
  ranked deeper.
- `app-not-found` — the `appId` is wrong. Re-check with `list_apps`.
- `store-not-configured` — the app has no id for that store.

A single rank is a point sample of a noisy series. Before calling a change a
regression or a win, pull `get_keyword_history` for the same term and country
and look at the trend. Ranks move on their own day to day. History has one row
per stored search snapshot: tracked keywords are re-checked weekly, plus any
day the term was searched in AppSkyline, so gaps between rows are normal.

## A standard ASO report

When asked for an ASO report on an app, work in this order:

1. `list_apps` → resolve the app and see which stores it is registered on.
2. `list_keywords` → the tracked set, which is the report's spine.
3. `get_keyword_rank` per important term and market, for the stores that are
   configured. Prefer the markets the user cares about over an exhaustive sweep.
4. `get_keyword_history` for terms whose current rank looks notably good or bad,
   to separate a real move from daily noise.
5. `get_keyword_overview` — **one batched call** per country over every term
   from steps 2–4 plus any candidates you are considering, to learn which terms
   are worth the effort at all.
6. `suggest_store_metadata` → audit titles, subtitles and the keyword field
   against the tracked keywords and get drafts within each store's limits;
   `get_store_metadata` adds every other synced locale. On Apple, unused
   keyword-field characters are wasted ranking surface.
7. `search_store_results` on the top few terms → who actually occupies the
   results, and `get_store_listing` on any competitor worth understanding.

Then report: where the app stands, which terms have volume the app is not
competitive on, which metadata fields are underused, and what you would change.
Rank without volume is vanity — a #1 for a term nobody searches is not a win,
and `get_keyword_overview` is what tells you the difference.

## Interactive cards

`get_keyword_rank` renders an interactive rank card, and the connector also
provides an app-overview card, in hosts that support MCP Apps. When a card
renders, do not restate its full contents in prose. Add what the card cannot
show: the trend behind the number, and what to do about it.
