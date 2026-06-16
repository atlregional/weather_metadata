# Metro Atlanta Weather Pipeline — Daily Diagnostics

This repository serves as a public dashboard for an ongoing weather data collection effort across the 11-county Metro Atlanta region. It is updated automatically every morning and contains no raw weather data — only progress metrics, station locations, and daily health statistics for the pipeline that collects the data.

---

## What This Repo Shows

The pipeline continuously scrapes historical and real-time weather observations from ~2,600 personal weather stations across metro Atlanta, writing them to a private data store. This repo acts as a window into how that collection effort is going — how many stations are covered, how far back the data goes, and which stations have gone dark.

---

## Files in This Repo

### `index.html` — Interactive Station Map

An interactive web map (viewable via GitHub Pages) that plots every scraped station as a dot on a dark-themed map of the metro Atlanta area.

**Dot colors:**
- **Cyan** — active station (data received within the last 30 days)
- **Red** — extinct station (no new data for 30+ consecutive days)

**Control panel (top-left):**
- Toggle each of the four data sources on/off (WU, AWN, GDOT, UGA)
- Filter by status: All / Active / Extinct

**KPI panel (bottom-left):**
- Shows live counts for whatever stations are currently visible in the map viewport — total, active, extinct, per-source breakdown, and average years of data collected. Pan and zoom to focus on a sub-region; the KPIs update to reflect only what's on screen.

**Station popups:**
- Click any dot to see a card with the station's name, source, first and last record dates, total days and observations collected, and years of coverage.

### `scraper-update.png` — Static Diagnostic Image

A daily-regenerated PNG showing the same station map plus a KPI header line. Useful for a quick at-a-glance snapshot without opening the interactive map. Useful if GitHub Pages is not enabled or you want to embed the image elsewhere.

### `pipeline_stats.txt` — Plain-Text Daily Report

A machine-readable summary updated each morning. Contains:

- **Extinct station counts** by source, cumulative total, and the day-over-day delta (how many new stations went dark since yesterday's report)
- **Average years of data per station** by source, with active vs. total station counts

Example:
```
EXTINCT STATIONS  (>=30 consecutive days without new data)
------------------------------------------------------------
  Cumulative total extinct:        120  /  2,616 stations
  New extinct today:           +     0
  Currently active:              2,496

  Source    Extinct    Total   New today
  --------  -------  -------  ----------
  WU            114    1,475          +0
  AWN             4    1,118          +0
  GDOT            2       17          +0
  UGA             0        6          +0

AVERAGE YEARS OF DATA PER STATION
------------------------------------------------------------
  WU          2.89 yrs / station   (1,361 active / 1,475 total)
  AWN         2.37 yrs / station   (1,114 active / 1,118 total)
  GDOT       10.04 yrs / station   (15 active / 17 total)
  UGA        22.35 yrs / station   (6 active / 6 total)
```

### `stations.geojson` — Full Station Dataset

A GeoJSON FeatureCollection containing one Point feature per scraped station. This is what `index.html` loads to render the map. Each feature's `properties` include:

| Property | Description |
|---|---|
| `source` | `WU`, `AWN`, `GDOT`, or `UGA` |
| `station_id` | Source-specific station identifier |
| `name` | Human-readable station name |
| `earliest_date` | First date with collected data |
| `latest_date` | Most recent date with collected data |
| `total_days` | Number of distinct calendar days collected |
| `total_observations` | Total individual readings collected |
| `years_active` | `total_days / 365.25`, rounded to 2 decimal places |
| `days_since_update` | Days elapsed since `latest_date` |
| `extinct` | `true` if no data for 30+ consecutive days |

This file can be loaded into any GIS tool (QGIS, ArcGIS, Felt, etc.) for further spatial analysis.

---

## Data Sources

| Source | Label | Coverage | Data Type |
|---|---|---|---|
| Weather Underground | WU | ~1,475 personal weather stations | Historical, nightly catch-up |
| Ambient Weather Network | AWN | ~1,118 personal weather stations | Historical, nightly catch-up |
| GDOT Road Weather Stations | GDOT | 16 highway sensors (metro subset) | Real-time, every 30 min |
| UGA Georgia Weather Network | UGA | 6 research-grade stations | Real-time, every 15 min |

---

## Update Schedule

This repo is updated once per day at **7:00 AM Eastern time**.

Each daily run:
1. Regenerates `scraper-update.png` from the latest metadata
2. Rebuilds `stations.geojson` from all four source metadata files
3. Overwrites `pipeline_stats.txt` with fresh counts and deltas
4. Commits and pushes all changed files

The commit message for each push is `Daily pipeline update YYYY-MM-DD`.

---

## Monitoring Pipeline Health

**Normal behavior:** The "New extinct today" count in `pipeline_stats.txt` should be 0 or a small positive number on any given day. A sudden large spike indicates that many stations stopped reporting simultaneously, which may signal a scraper outage, a source API change, or a network issue — not necessarily that the stations themselves went offline.

**Avg years/station** is a measure of historical depth. As the pipeline catches up on backfilled history for each station, this number grows. Once the scraper is fully caught up (every station has data through yesterday), growth in this metric slows to roughly 1/365 per day.

**Extinct stations** are not deleted — they remain in the map and GeoJSON in red. The pipeline probes them weekly; if a station starts reporting again, its `extinct` flag is cleared automatically and it returns to cyan on the next daily push.

---

## Viewing the Interactive Map

If GitHub Pages is enabled on this repository, `index.html` is served as the root page. Navigate to the repository's GitHub Pages URL to open the interactive map directly in your browser.

If GitHub Pages is not enabled, you can download `index.html`, `stations.geojson`, `metro_atlanta_counties.geojson`, and the `fonts/` directory, keep them in the same folder, and open `index.html` locally in any modern browser.
