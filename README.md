# FIFA World Cup 2026 — Home Assistant Dashboard

A Home Assistant dashboard showing live match scores, today's fixtures, and all 12 group stage standings tables, powered by the public ESPN API (no API key needed).

Tournament starts: **11 June 2026**

---

## Screenshot

*Live match cards + group tables (A–L) across a 3-column responsive grid*
<img width="1202" height="577" alt="image" src="https://github.com/user-attachments/assets/5dcb1a04-2da9-44dd-8522-7fd38758d89d" />


---

## What It Does

- **Live team cards** — one card per nation showing their current/next match, score, and status (pre-match countdown, live minute, final score)
- **Today's fixtures** — markdown scoreboard listing all matches today with live scores
- **Group tables A–F** and **G–L** — full standings (P/W/D/L/GD/Pts) auto-updating every 30 minutes from the ESPN API
- **Responsive layout** — 3 columns on wide screens, 2 on medium, 1 on mobile

---

## Requirements

### HACS Integrations
Install via HACS > Integrations:
- [teamtracker](https://github.com/vasqued2/ha-teamtracker) — provides the live match sensor for each team

### HACS Frontend (Lovelace Cards)
Install via HACS > Frontend:
- [lovelace-layout-card](https://github.com/thomasloven/lovelace-layout-card) — provides `custom:grid-layout`
- [card-mod](https://github.com/thomasloven/lovelace-card-mod) — provides card styling

---

## Setup

### Step 1 — Add the package sensor

Copy `packages/world_cup_2026.yaml` into your HA config packages folder, then enable packages in `configuration.yaml` if not already done:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

This creates two sensors:
- `sensor.wc_standings` — group table data (updates every 30 min)
- `sensor.wc_scoreboard` — today's match scores (updates every 2 min)

### Step 2 — Add TeamTracker entries for your teams

Go to **Settings > Devices & Services > Add Integration > TeamTracker** and add one entry per team you want a live card for.

Use these settings:

| Team        | Sport  | League ID | League Path  | Team ID |
|-------------|--------|-----------|--------------|---------|
| England     | soccer | WC        | fifa.world   | 448     |
| Spain       | soccer | WC        | fifa.world   | 164     |
| France      | soccer | WC        | fifa.world   | 478     |
| Germany     | soccer | WC        | fifa.world   | 481     |
| Portugal    | soccer | WC        | fifa.world   | 482     |
| Netherlands | soccer | WC        | fifa.world   | 449     |

To find the team ID for other nations, browse the ESPN API:
```
https://site.api.espn.com/apis/site/v2/sports/soccer/fifa.world/teams
```

Name each entry `<Country> WC` so the entity IDs match the dashboard (e.g. `sensor.england_wc`).

### Step 3 — Add the Lovelace view

1. Open your Lovelace dashboard
2. Click the three-dot menu > **Edit Dashboard** > **Raw configuration editor**
3. Under `views:`, paste the contents of `lovelace/world_cup_view.yaml` as a new list item
4. Save and close the editor

### Step 4 — Restart HA

Restart Home Assistant to load the new package sensors. The group tables will populate on first successful API call.

---

## Customising Teams

To add or swap teams, add extra TeamTracker config entries (Step 2) and then add the corresponding card to the `live` column in `lovelace/world_cup_view.yaml`:

```yaml
- type: custom:teamtracker-card
  entity: sensor.brazil_wc
  card_mod:
    style: "ha-card { font-size: 85%; } :host { zoom: 0.85; }"
```

---

## How It Works

- **Standings** — calls `site.api.espn.com/apis/v2/sports/soccer/fifa.world/standings?season=2026` and stores the `children` array (one object per group) as a JSON attribute on the sensor. The Lovelace markdown card loops over this with Jinja2 to render the tables.
- **Scoreboard** — calls the ESPN scoreboard endpoint for today's matches and renders them inline via a Jinja2 markdown card.
- **TeamTracker** — the integration polls ESPN's match API for each team and provides a purpose-built card showing live match state.

---

## No Support

This dashboard is shared as-is with no support provided. It was built for personal use and shared for others to adapt freely.

---

*Built with [Claude Code](https://claude.ai/claude-code)*
