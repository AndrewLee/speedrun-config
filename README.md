# speedrun-config

One small, public source of truth for the dates and versions that Speedrun's
**founder-facing surfaces** read at runtime — so we can re-date a cohort or ship
a tool update by editing one file here, with no app rebuild or reinstall.

It is consumed by:

- **[`sr-chrome-extension`](https://github.com/a16z-speedrun/sr-chrome-extension)** — the Demo Day new-tab countdown
- **[`speedrun-statusline`](https://github.com/a16z-speedrun/speedrun-statusline)** — the Claude Code status line

Both fetch [`cohort.json`](./cohort.json) (raw, public, no auth), cache it
locally, and **fall back to their own baked-in defaults** if the fetch fails —
so a network hiccup never breaks a countdown.

## `cohort.json`

```jsonc
{
  "cohorts": {
    "speedrun": {                              // Speedrun Demo Day (founder-facing)
      "cohort": "SR007",
      "milestone": "Demo Day",
      "demo_day": "2026-10-06T09:00:00-07:00", // ISO-8601 with explicit offset
      "date_label": "October 6, 2026"
    },
    "alpha": {                                 // Speedrun Alpha Showcase
      "cohort": "Alpha",
      "milestone": "Showcase",
      "demo_day": "2026-08-04T09:00:00-07:00",
      "date_label": "August 4, 2026"
    }
  },
  "statusline": {
    "latest_version": "1.1.0"                  // drives the status line's "⬆ update" nudge
  }
}
```

Each consumer reads only what it needs: the extension reads both cohorts (it has
a toggle); the status line reads only `cohorts.speedrun` plus `statusline.latest_version`.

**Always use an explicit UTC offset** on `demo_day` (e.g. `-07:00` for PDT) so the
countdown is identical on every machine in every timezone.

## How to make a change

| To… | Edit | Effect |
|---|---|---|
| Re-date a cohort | the cohort's `demo_day` / `date_label` | Both surfaces pick it up automatically (extension on next load, status line within ~6h) |
| Add the next cohort | a new key under `cohorts`, or repoint `speedrun` | Same |
| Push a status line update | `statusline.latest_version` | Installs surface the `⬆ update` nudge |

Commit to `main` — that's the published version. There is no build step.

## Note on the long-term home

The architecturally-correct home for canonical program data (cohorts, dates) is
**mission-control's `gold` schema** — the contract every app reads. This repo is a
pragmatic public projection for unauthenticated founder surfaces until a public
read-only `gold` endpoint exists; at that point this file becomes a thin mirror of it.
