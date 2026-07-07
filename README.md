# Horizon Stats

Machine-updated statistics for [F-Chat Horizon](https://github.com/Fchat-Horizon/Horizon): shields.io badge endpoints plus historical time series for downloads, stars, forks, and other repo metrics. A [scheduled workflow](.github/workflows/snapshot.yml) snapshots the numbers daily at 03:43 UTC (and on manual dispatch) and commits the result here, so this repo's history _is_ the dataset.

Why a separate repo? Two reasons:

1. **Accuracy.** GitHub's own download counter includes `latest*.yml`, `*.blockmap`, and `*.zsync` files that the auto-updater fetches on every update check, inflating the number by ~40%. We compute the real count ourselves.
2. **History.** GitHub only exposes _current_ totals, never how they got there. Daily snapshots build the time series — for badges, graphs on the website, or anything else.

(It also keeps the daily bot commits out of Horizon's push webhooks, which ping Discord.)

## Badges

[`badges/downloads.json`](badges/downloads.json) is [shields endpoint](https://shields.io/badges/endpoint-badge) data for the accurate download count:

```markdown
![Downloads](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2FFchat-Horizon%2Fstats%2Fmain%2Fbadges%2Fdownloads.json&style=flat-square)
```

To add another custom badge, write another endpoint-schema JSON into `badges/` from the workflow and reference it the same way.

## Data files

All history files are NDJSON (one JSON object per line) — trivial to consume from a browser: `(await (await fetch(url)).text()).trim().split('\n').map(JSON.parse)`.

### `data/history.ndjson`

The main series. One line per day (a re-run on the same day replaces that day's line):

```jsonc
{
  "date": "2026-07-07",
  "repo": {
    "stars": 49,
    "forks": 33,
    "subscribers": 6, // watchers
    "openIssues": 122, // true issues, PRs excluded
    "openPRs": 14,
    "contributors": 41, // non-anonymous, default branch
    "commits": 2756, // default branch
    "releases": 99,
  },
  "downloads": {
    "real": 86091, // lifetime, excluding updater metadata
    "total": 144016, // raw sum — what the stock GitHub badge shows
    "byPlatform": { "windows": 66511, "linux": 15146, "mac": 4434 },
    "byArch": { "x64": 78288, "universal": 4912, "arm64": 2887, "...": 0 },
    "byExtension": { "exe": 66288, "AppImage": 8287, "...": 0 },
    "byRelease": {
      "v2.3.1": { "total": 1500, "byPlatform": { "windows": 1200, "...": 0 } },
    },
  },
}
```

- `downloads.real`, `byPlatform`, `byArch`, `byExtension`, and `byRelease` exclude updater metadata (`latest*.yml`, `*.blockmap`, `*.zsync`, `SHASUMS*`); `downloads.total` includes it.
- Platform/arch are classified from asset filenames. `byArch.universal` covers multi-arch bundles: macOS universal builds and the arch-suffix-less Windows installers (e.g. `F-Chat.Horizon-win.exe`), which pack every built arch into one exe.
- Counts are lifetime cumulative; diff consecutive days to get dailies.

### `data/star-history.ndjson` and `data/fork-history.ndjson`

Sparse cumulative series — a line is added only on days the count changed:

```json
{ "date": "2025-02-21", "stars": 1 }
{ "date": "2025-02-22", "forks": 2 }
```

Seeded on 2026-07-07 by backfilling `starred_at` timestamps from the stargazers API and `created_at` from the forks API. Both APIs only report _current_ stargazers/forks, so un-stars and deleted forks before the seed date are invisible; from the seed date onward the numbers are exact daily observations.

### `data/releases.json`

Release index (tag, name, publish date, prerelease flag), sorted oldest-first, regenerated on every run. Useful for annotating graphs with release markers.

## Caveats

- Download history cannot be backfilled — GitHub keeps no record. The series starts 2026-07-07.
- GitHub disables scheduled workflows in repos with no activity for 60 days. The daily commits keep this alive, but if the workflow errors for 60 days straight it will stop; re-enable it from the Actions tab.
- Possible future additions: Weblate translation progress (translate.horizn.moe API), nightly-build metrics, more badges.
