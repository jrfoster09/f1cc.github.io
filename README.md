# F1CC Wiki

The official wiki and database for the F1 Manager Community Career.

🏁 **Live site:** `https://[your-username].github.io/f1cc-wiki`

---

## 📁 Repo Structure

```
f1cc-wiki/
├── index.html              ← Homepage
├── pages/
│   ├── seasons.html        ← All seasons
│   ├── season.html         ← Single season (standings + calendar)
│   ├── race.html           ← Single race result
│   ├── drivers.html        ← Driver grid
│   ├── driver.html         ← Driver profile (live career stats)
│   ├── teams.html          ← Teams grid
│   ├── team.html           ← Team profile
│   └── records.html        ← All-time records (auto-computed)
├── data/
│   ├── db.json             ← Master database (drivers, teams, season standings)
│   ├── races_2024.json     ← 2024 race-by-race results
│   ├── races_2025.json     ← 2025 race-by-race results
│   └── races_YYYY.json     ← Add a new file for each future season
├── css/style.css           ← Global styles
└── js/core.js              ← Shared JS utilities
```

---

## 🏎️ Adding a Race Result (Weekly Update — ~5 mins)

After each race weekend, open `data/races_YYYY.json` (where YYYY is the current season year).

Find the race entry for that round and fill in the `results` array:

```json
{
  "id": "2025_r05",
  "round": 5,
  "name": "Monaco Grand Prix",
  "circuit": "Circuit de Monaco",
  "country": "MC",
  "flag": "🇲🇨",
  "date": "2025-02-09",
  "status": "complete",
  "vod_url": "https://youtube.com/watch?v=XXXX",
  "summary": "A wet Monaco produced drama from start to finish...",
  "results": [
    {"pos":1,"driver":"quarter_kevu","team":"racing_bulls","points":25,"fastest_lap":false,"quali_pos":1},
    {"pos":2,"driver":"liam_dela_cruz","team":"ferrari","points":18,"fastest_lap":false,"quali_pos":3},
    ...
  ]
}
```

**Points system:** 25-18-15-12-10-8-6-4-2-1 for P1–P10. Fastest lap = 0 extra points (not tracked separately in standings).

After saving, also update `db.json` → the relevant season's `driver_standings` and `constructor_standings` arrays with the new cumulative totals.

---

## ➕ Adding a New Season

1. In `db.json`, add a new entry under `"seasons"`:
```json
"2026": {
  "year": 2026,
  "status": "live",
  "champion_driver": null,
  "champion_constructor": null,
  "rounds": 24,
  "races": [],
  "driver_standings": [],
  "constructor_standings": []
}
```

2. Update `"currentSeason": 2026` in `db.json` → `"meta"`.

3. Create `data/races_2026.json` with all the planned race entries (status: "upcoming").

4. That's it — the whole site updates automatically.

---

## ➕ Adding a New Driver

In `db.json` → `"drivers"`, add a new entry:

```json
"new_driver_id": {
  "id": "new_driver_id",
  "name": "Driver Name",
  "nationality": "GB",
  "flag": "🇬🇧",
  "bio": "Short bio here.",
  "seasons": {
    "2026": { "team": "ferrari", "number": 44 }
  }
}
```

Then reference `"new_driver_id"` in race results and standings — done.

---

## 🔄 Changing a Team Name

In `db.json` → `"teams"`, update the `"name"` field. The old name is preserved in history because standings reference team IDs, not names.

---

## 🚀 Setting Up GitHub Pages

1. Create a GitHub account at [github.com](https://github.com)
2. Create a new repository named `f1cc-wiki` (or `[username].github.io`)
3. Upload all these files (drag and drop in the GitHub web interface)
4. Go to **Settings → Pages → Source → main branch → / (root)**
5. Your site is live at `https://[username].github.io/f1cc-wiki`

Every time you update a file on GitHub, the site updates within ~30 seconds.
