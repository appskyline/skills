---
name: store-listing-audit
description: Audit an iOS App Store, Mac App Store, Google Play, or Microsoft Store listing with AppSkyline. Use when asked to review an app title, subtitle, keyword field, description, localization, competitor positioning, or listing metadata and produce evidence-based recommendations without inventing search volume or store support.
license: Apache-2.0
compatibility: Requires the AppSkyline MCP connector (https://mcp.appskyline.com/mcp) and an AppSkyline account
metadata:
  author: Appskyline
  version: '1.0.0'
  repository: https://github.com/appskyline/skills
---

# Audit a store listing with AppSkyline

Use this workflow to compare the current listing with observable keyword and
competitor evidence. Never assume that metadata cached in AppSkyline is the
currently published store version; state the observation time and source.

## Resolve the app and store

1. Call `list_apps` and match the user's app by name.
2. Call `get_app` and identify which store-native ids are configured.
3. Ask which country and language matter if the user did not specify them.
4. Do not substitute one store's identifier for another. Apple ids are numeric,
   Google Play uses a package name, and Microsoft Store uses a product id.

Use the exact API store values: `ios-app-store`, `macos-app-store`,
`google-play`, and `microsoft-store`.

## Read the current evidence

For an app connected to the account:

- Call `get_store_metadata` for its synchronized listing fields.
- Call `list_keywords` for the terms already tracked for that app.
- Call `get_keyword_rank` for the important terms in the selected store and
  country. Report the scanned depth with every position.
- Call `get_keyword_history` before describing a rank as improving or falling.

For the visible competitive field:

- Call `search_store_results` once per selected query and market.
- Use `get_store_listing` for the strongest relevant competitors returned by
  that search, using their store-native ids.
- Treat a live result as an observation, not a permanent rank.

## Validate keyword demand

Collect all candidate terms first, then call `get_keyword_overview` once per
country with the full list. The tool supports a batch of terms and may trigger
billable upstream research for uncached terms, so never loop over one term per
request.

Do not claim that a term has demand when the tool returned no volume. Do not
equate high volume with relevance or achievable rank. Prefer a defensible mix
of relevant terms, observed competitors, current rank, and measured demand.

## Audit each field

Review only fields the selected store actually supports:

- Title or app name: clear product identity plus the strongest relevant phrase
  that still reads naturally.
- Apple subtitle: a concise differentiator; do not repeat the title verbatim.
- Apple keyword field: check the reported character count against Apple's
  100-character limit, remove avoidable duplicates, and do not add spaces after
  commas merely for readability.
- Short and long descriptions: explain the product before listing features,
  connect claims to observable capability, and avoid unsupported superlatives.
- Localization: evaluate each country and language separately. Never translate
  a term blindly when local store results use a different phrase.
- Release notes and promotional text: keep time-sensitive claims out of stable
  metadata unless the user intends to maintain them.

## Produce the recommendation

Return these sections:

1. `Current listing` — store, country, language, source, and observed fields.
2. `Keyword evidence` — current positions with scan depth, history, and volume.
3. `Competitive evidence` — the relevant listings actually returned by search.
4. `Gaps` — unclear positioning, unused field space, duplication, localization,
   or evidence the listing targets terms that do not match visible results.
5. `Proposed metadata` — field-by-field copy, preserving each store's limits.
6. `Verification plan` — what to measure after publication and when to compare.

Do not upload or mutate metadata unless the user explicitly requests it after
reviewing the proposal. If an upload is requested, show the exact changed fields
and target locale first, use a dry run where available, and preserve a copy of
the prior listing for rollback.
