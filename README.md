# F1CC Wiki

The development and testing git for the F1 Manager Community Career Wiki.

**TO DO LIST (Eren add if need be)**
- Fix Records no Emoji bug
- Continue to create, assign and fill in driver photos
- Add Team Principal Box to teams

**Live site:** https://jrfoster09.github.io/f1cc-wiki.github.io/

-----

## File Structure

```
f1cc-wiki/
├── index.html                  ← Homepage
├── css/style.css               ← All styling (navy theme, team colours, layout)
├── js/core.js                  ← Shared helpers (teamPill, posBadge, driverLink etc.)
├── assets/
│   └── photos/                 ← Driver profile photos (PNG/JPG/AVIF)
├── data/
│   ├── db.json                 ← Master database (drivers, teams, seasons, standings)
│   ├── races_2024.json         ← 2024 race-by-race results
│   ├── races_2025.json         ← 2025 race-by-race results
│   └── races_2026.json         ← 2026 race-by-race results (fills during season)
└── pages/
    ├── seasons.html            ← All seasons index
    ├── season.html             ← Single season (standings + calendar + constructors)
    ├── race.html               ← Single race result
    ├── drivers.html            ← All drivers grid (searchable)
    ├── driver.html             ← Driver profile (career stats, season history)
    ├── teams.html              ← Teams grid
    ├── team.html               ← Team profile
    ├── records.html            ← All-time records (auto-computed live)
    └── admin.html              ← Password-protected result entry panel
```

-----

## How the Data Works

All pages fetch `data/db.json` and the relevant `races_YYYY.json` at runtime. There is no build step — edit a JSON file, push, done.

### db.json Structure

```json
{
  "meta": {
    "last_updated": "2026-05-10",
    "currentSeason": "2026"        ← String, not int. Must match a key in "seasons".
  },
  "teams": {
    "vcarb": {
      "name": "VCARB",
      "short": "VCARB",
      "color": "#6692FF"           ← Hex colour used for team pills throughout the site
    }
  },
  "drivers": {
    "Quarter Kevu": {              ← Full name is the key AND the ID used everywhere
      "flag": "ee",               ← ISO 3166-1 alpha-2 country code (lowercase)
      "flag_emoji": "🇪🇪",        ← Pre-computed emoji (do not edit manually)
      "name": "Quarter Kevu",
      "photo": "https://jrfoster09.github.io/f1cc-wiki.github.io/assets/photos/quarter-kevu.png",
      "seasons": {
        "2025": { "team": "vcarb", "pos": 1, "points": 259 }
      }
    }
  },
  "seasons": {
    "2025": {
      "year": 2025,
      "rounds": 30,
      "status": "complete",        ← "complete" or "live"
      "champion_driver": "Quarter Kevu",
      "champion_constructor": "vcarb",
      "calendar": [...],
      "driver_standings": [...],
      "constructor_standings": [...]
    }
  }
}
```

### races_YYYY.json Structure

```json
{
  "races": [
    {
      "round": 1,
      "id": "aus_r1",
      "name": "AUS",
      "flag": "🇦🇺",
      "circuit": "Albert Park",
      "date": "2025",
      "sprint": false,
      "status": "complete",        ← "complete" or "upcoming"
      "results": [
        {
          "driver": "Liam Dela Cruz",   ← Must exactly match the key in db.json drivers
          "pos": 1,                     ← Integer, or "DNF" / "DSQ"
          "team": "ferrari",            ← Team ID matching db.json teams
          "fastest_lap": false,
          "quali_pos": 4,              ← Qualifying position (1 = pole)
          "points": 25                 ← Computed points including FL bonus
        }
      ]
    }
  ]
}
```

-----

## Points System

|Position|Race|Sprint|
|--------|----|------|
|P1      |25  |8     |
|P2      |18  |7     |
|P3      |15  |6     |
|P4      |12  |5     |
|P5      |10  |4     |
|P6      |8   |3     |
|P7      |6   |2     |
|P8      |4   |1     |
|P9      |2   |—     |
|P10     |1   |—     |

**Fastest lap bonus:** +1 point, but only in non-sprint races and only if the driver finishes in the top 10.

**Abu Dhabi 2025:** Double points applied to all positions.

-----

## How to Add a Race Result

The easiest way is via the **admin panel** at `/pages/admin.html` (password: `f1cc2025`).

To do it manually in `races_YYYY.json`:

1. Find the race by round number
1. Set `"status": "complete"`
1. Fill in the `results` array — one object per driver, sorted P1 to last
1. Mark `"fastest_lap": true` on the relevant driver (only one per race)
1. Add `"quali_pos"` for each driver (1 = pole)
1. Compute `"points"` using the table above

Then update `db.json` driver standings:

- Update `wins`, `podiums`, `fastest_laps`, `poles`, `races`, `points` for each driver in `seasons.YYYY.driver_standings`
- Update `constructor_standings` if needed

-----

## How to Add a Driver Photo

1. Add the image file to `assets/photos/` (PNG, JPG or AVIF, ideally square crop)
1. In `db.json`, find the driver and set their `photo` field to the **full absolute URL**:

```json
"photo": "https://jrfoster09.github.io/f1cc-wiki.github.io/assets/photos/your-driver.png"
```

Do **not** use a relative path like `assets/photos/...` — it breaks on subpages.

-----

## How to Add a New Driver

1. Add an entry to `db.json` under `"drivers"`:

```json
"New Driver Name": {
  "flag": "gb",
  "flag_emoji": "🇬🇧",
  "name": "New Driver Name",
  "photo": null,
  "seasons": {}
}
```

1. Add their stint to the relevant season’s `driver_standings` array
1. Add their results to the race files

**Flag emoji reference:** Add the correct `flag_emoji` by looking up the country’s ISO code and finding the emoji — e.g. `gb` → 🇬🇧, `de` → 🇩🇪, `nl` → 🇳🇱.

-----

## How to Add a New Season

1. Add a new season block to `db.json` under `"seasons"`:

```json
"2027": {
  "year": 2027,
  "rounds": 24,
  "status": "live",
  "champion_driver": null,
  "champion_constructor": null,
  "calendar": [...],
  "driver_standings": [...],
  "constructor_standings": []
}
```

1. Create `data/races_2027.json` with empty race stubs
1. Update `"currentSeason": "2027"` in `db.json` meta

-----

## Critical JSON Rules

JSON is strict. One mistake breaks the entire file and the site goes blank.

**Always use a comma after every field except the last one in an object:**

```json
{
  "flag": "ee",        ← comma ✓
  "name": "Quarter Kevu",  ← comma ✓
  "photo": null        ← NO comma on last field ✓
}
```

**Validate your JSON before pushing** — paste it into https://jsonlint.com or use VS Code which highlights errors in real time.

-----

## Team IDs

|ID            |Display Name|
|--------------|------------|
|`vcarb`       |VCARB       |
|`red_bull`    |Red Bull    |
|`ferrari`     |Ferrari     |
|`mclaren`     |McLaren     |
|`mercedes`    |Mercedes    |
|`aston_martin`|Aston Martin|
|`williams`    |Williams    |
|`alpine`      |Alpine      |
|`haas`        |Haas        |
|`kick_sauber` |Kick Sauber |
|`audi`        |Audi        |

-----

## Tech Stack

- **Pure HTML/CSS/JS** — no frameworks, no build tools, no Node
- **GitHub Pages** — free static hosting, deploys on every push to main
- **Twemoji** — renders flag emoji as images for cross-platform support (loaded from CDN)
- **Google Fonts** — Barlow Condensed (display), Barlow (body), JetBrains Mono (numbers)

-----

## Admin Panel

Located at `/pages/admin.html`. Not linked in the public nav — bookmark it.

- Password: `f1cc2025` (change this in the HTML before sharing with anyone)
- Generates ready-to-paste JSON for race results
- Auto-calculates points
- Validates for duplicate drivers, multiple fastest laps etc.

-----

## Common Issues

**Site goes blank / shows “Loading…” everywhere**
→ `db.json` has a JSON syntax error. Validate at jsonlint.com.

**Photos not showing**
→ Check the photo URL is absolute (starts with `https://`), not relative.

**Flags showing as letters (EE, GB) instead of emoji**
→ Twemoji CDN may be slow or blocked. Check browser console for errors.

**New race results not appearing**
→ Hard refresh (`Ctrl+Shift+R`) — GitHub Pages caches aggressively.

**Season page blank**
→ Check that `currentSeason` in `db.json` meta matches a key in `seasons` exactly (as a string, e.g. `"2026"` not `2026`).
