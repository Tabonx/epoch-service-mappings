# Epoch service mappings

Corrections for where an external service and TMDB disagree about show boundaries, season
numbering or episode numbering. Epoch fetches this list at runtime, so a mapping found on a
Tuesday does not wait for an App Store review.

The app ships a copy of the same file and falls back to it whenever anything here fails.

```
https://raw.githubusercontent.com/Tabonx/epoch-service-mappings/main/v1/service-mappings.json
```

## Do not change

Builds that will never be updated again read that URL. The repository name, the `main`
branch and the path are a contract. Never rename any of them, never move or delete a
published file, and never redefine an existing field. Adding a field is safe.

## Versioning

One directory per envelope version. Most edits do not need a new one: an app that meets a
rule type, a service or a numbering it cannot act on skips that rule and reads the rest of
the file.

Add `v2/` only when the envelope itself changes shape. Then point a new build at it and keep
editing `v1/` for as long as installs still read it.

## Rules

A season rule maps a service season onto a TMDB show. A service episode `n` is TMDB episode
`n + episodeOffset`. `episodeCount` bounds what the rule owns, so two entries sharing one
TMDB season cannot overwrite each other.

```json
{
  "type": "season",
  "service": "simkl",
  "numbering": "tvdb",
  "note": "Why this rule exists.",
  "from": { "tmdb": 67557, "season": 7 },
  "to": { "tmdb": 329471, "season": 1, "episodeOffset": 0, "episodeCount": 6 }
}
```

An entry rule says what a write needs beyond seasons. `ids` is what the write must carry to
reach the entry, for when the ids it advertises do not resolve. `tmdbShow: false` says the
TMDB id names no show on TMDB, which stops the app fetching it. Write that only when TMDB
actually answered, never from a failed request.

```json
{
  "type": "entry",
  "service": "simkl",
  "note": "Why this rule exists.",
  "entry": { "tmdb": 67557, "season": 7 },
  "ids": { "imdb": "tt5712554", "tmdb": 67557, "tvdb": 314087 }
}
```

Several entries can advertise the same TMDB id, so `entry.season` narrows a rule to one of
them. Without it the rule speaks for all of them.

## Adding a rule

Check the numbering against the service, do not assume an offset of zero:

```
GET https://api.simkl.com/tv/episodes/<simkl id>?client_id=<key>
GET https://api.simkl.com/tv/<simkl id>?extended=full&client_id=<key>
```

Confirm a write addressed the new way reaches the entry, then edit
`v1/service-mappings.json` and write a `note` saying what the disagreement is. Clients pick
the change up on their next refresh and apply it on the launch after that.
