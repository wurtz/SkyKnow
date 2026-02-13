# SkyKnow — Project Spec & Dev Guide

## Overview

A custom Tronbyt/Tidbyt app that monitors weather forecasts and displays precipitation alerts when rain or snow is expected. Shows event duration, day, probability, and accumulation — with severity-based colors and animated particle effects.

---

## Display Layout (64x32 pixels)

### Hero Duration Layout — Precipitation Alert

Three-line information hierarchy optimized for 1-2 second glances on an ambient display.

```
┌────────────────────────────────────────┐
│  ❄ SNOW 6h                            │  Line 1: Icon + Type + Duration (severity colored)
│  Thu                                   │  Line 2: Day label (dimmed gray)
│  85% 3-6"                              │  Line 3: Probability + Accumulation (dimmed gray)
└────────────────────────────────────────┘
```

Animated falling particles play behind the text — snowflakes (1px dots with drift) for snow, rain streaks (1x2px) for rain.

**Information Hierarchy:**

| Line | Content | Color | Purpose |
|------|---------|-------|---------|
| **Line 1** | Icon + TYPE + Duration | Severity color | **The alert.** What + how long. Eye hits this first. |
| **Line 2** | Day label | Dimmed gray (`#777` – `#999` by severity) | **The anchor.** Grounds the duration in real life. |
| **Line 3** | Probability% + Accumulation | Dimmed gray (`#777` – `#AAA` by severity) | **The details.** Probability leads (gates attention), accumulation follows. |

**Design Principle:** The particle animation + severity color already encode type and severity visually. Text confirms what the animation communicates — it does not duplicate it.

### Severity Colors

| Severity | Snow Header | Rain Header |
|----------|-------------|-------------|
| Light    | Blue `#4488FF` | Cyan `#44AAFF` |
| Moderate | Amber `#FFAA00` | Amber `#FFAA00` |
| Heavy    | Red `#FF4444` | Orange-Red `#FF6633` |

Lines 2–3 use neutral gray that brightens slightly at higher severity levels:
- Light: `#777777`
- Moderate: `#888888` / `#999999`
- Heavy: `#999999` / `#AAAAAA`

### Duration Format

- `NOW` — event is currently happening
- `6h` — event lasts 6 hours
- `1d3h` — event lasts 1 day, 3 hours

### Day Label Rules

Line 2 uses human-friendly day labels. Keep text short — 64px display fits ~12-13 characters max per line with the 4px font.

| Condition | Label | Example |
|-----------|-------|---------|
| Currently happening | `Thru [time]` | `Thru 8pm` |
| Later today (daytime) | `Today` | `Today` |
| Tonight (evening/overnight) | `Tonight` | `Tonight` |
| Tomorrow | `Tomorrow` | `Tomorrow` |
| 2+ days out | Abbreviated day | `Wed`, `Thu`, `Fri` |

**NOW state exception:** When precipitation is actively happening, the user's question shifts from "when does it start?" to "when does it stop?" — so Line 2 shows the estimated end time (`Thru 8pm`, `Thru 3am`) instead of a day name.

### Accumulation Labels

**Snow:** Dusting, <1", 1-3", 3-6", 6-12", 12"+

**Rain:** Light, Moderate, Heavy (or numeric for heavy, e.g., `2.5"`)

> **Note:** "Trace" was removed from rain labels — it's meteorology jargon that causes confusion on an ambient display. "Light" communicates the same intent ("don't worry about it") without making anyone pause. Snow uses "Dusting" which is universally understood.

### Example Screens — Snow

```
Light                    Moderate                 Heavy                    NOW
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ ❄ SNOW 6h        │    │ ❄ SNOW 1d3h      │    │ ❄ SNOW 12h       │    │ ❄ SNOW NOW       │
│ Thu               │    │ Fri               │    │ Tonight           │    │ Thru 8pm          │
│ 40% Dusting       │    │ 85% 3-6"          │    │ 95% 12"+          │    │ 100% 6-12"        │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
  Blue header              Amber header             Red header              Red header
```

### Example Screens — Rain

```
Light                    Moderate                 Heavy                    NOW
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ 💧 RAIN 2d        │    │ 💧 RAIN 18h       │    │ 💧 RAIN 4h        │    │ 💧 RAIN NOW       │
│ Sun               │    │ Tomorrow          │    │ Today              │    │ Thru 3pm          │
│ 25% Light         │    │ 70% Moderate      │    │ 90% Heavy          │    │ 100% Heavy        │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
  Cyan header              Amber header           Orange-Red header       Orange-Red header
```

### All Clear Screen

```
┌──────────────────────────┐
│         🐸               │  Pixel art tree frog (centered, with blink + throat puff animation)
│      ALL CLEAR           │  Dimmed text `#445566` (centered)
│      NEXT 2D             │  Forecast window `#334455` (centered)
└──────────────────────────┘
```

No particle animation. Quiet and friendly — stays out of the way in app rotation. The frog sprite, text, and forecast window are all center-aligned on the display.

### Multiple Events

When multiple precipitation events are detected (e.g., rain tomorrow morning then snow tomorrow night), the app animates between them — each event shows for ~3.2 seconds with its own particle animation before transitioning to the next.

---

## Frog Sprite — All Clear Screen

Front-facing chunky tree frog with half-lidded "I'm not impressed" eyes, big yellow belly.

### Sprite Data (13x11 pixels)

Color key: `0`=transparent, `D`=dark green, `G`=green, `Y`=yellow belly, `K`=eyelid (dark), `E`=eye yellow, `P`=pupil, `T`=toe pad

**Idle:**
```
000DDDDDDD000
0KDDDDDDDDDK0
0KKDDDDDDDKK0
0EPEDDDDDEPE0
0DDDDDDDDDDD0
0DDDDDDDDDDD0
DGGYYYYYYYGGD
DGGYYYYYYYGGD
DGGYYYYYYYGGD
0DDDDDDDDDDD0
0TT00DDD00TT0
```

**Blink** (eyes closed — lids cover eye slit):
```
000DDDDDDD000
0KDDDDDDDDDK0
0KKDDDDDDDKK0
0KKDDDDDDDKK0   ← lids shut
0DDDDDDDDDDD0
0DDDDDDDDDDD0
DGGYYYYYYYGGD
DGGYYYYYYYGGD
DGGYYYYYYYGGD
0DDDDDDDDDDD0
0TT00DDD00TT0
```

**Throat Puff** (yellow belly expands up into chin):
```
000DDDDDDD000
0KDDDDDDDDDK0
0KKDDDDDDDKK0
0EPEDDDDDEPE0
0DDDDDDDDDDD0
0DDYYYYYYYDD0   ← yellow creeps up
DGGYYYYYYYGGD
DGGYYYYYYYGGD
DGGYYYYYYYGGD
0DDDDDDDDDDD0
0TT00DDD00TT0
```

### Color Palette

| Key | Color | Hex | Usage |
|-----|-------|-----|-------|
| D | Dark green | `#2B7A2B` | Body, head, outline |
| G | Green | `#45A845` | Lighter body sides |
| Y | Yellow | `#E2BB38` | Belly |
| K | Eyelid | `#1E5E1E` | Heavy drooping lids |
| E | Eye | `#CCCC55` | Exposed eye slit |
| P | Pupil | `#111111` | Pupil dot |
| T | Toe | `#3D9E3D` | Toe pads |

### Animation Cycle (90 frames @ 100ms = 9 seconds)

| Frames | State | Duration |
|--------|-------|----------|
| 0–49 | Idle | 5.0s |
| 50–53 | Blink | 0.4s |
| 54–73 | Idle | 2.0s |
| 74–79 | Throat Puff | 0.6s |
| 80–89 | Idle | 1.0s |

---

## Tech Stack

| Component         | Choice                                                                 |
|-------------------|------------------------------------------------------------------------|
| **Language**       | Starlark (Python-like, used by Tidbyt/Tronbyt)                       |
| **Weather API**    | [Open-Meteo](https://open-meteo.com/en/docs) (free, no API key)      |
| **Rendering**      | Tidbyt `render` and `schema` modules                                  |
| **Dev Tools**      | [Pixlet](https://tidbyt.dev/docs/build/build-for-tidbyt) CLI         |
| **Deployment**     | Push via Tronbyt or `pixlet push`                                     |

---

## Weather API Details

### Open-Meteo Hourly Forecast Endpoint

```
https://api.open-meteo.com/v1/forecast
  ?latitude={lat}
  &longitude={lng}
  &hourly=snowfall,rain,precipitation_probability,weathercode
  &temperature_unit=fahrenheit
  &timezone={tz}
  &forecast_days={1-7}
```

The `forecast_days` parameter is derived from the user's chosen forecast interval.

### Key Response Fields

| Field                        | Description                                | Units     |
|------------------------------|--------------------------------------------|-----------|
| `hourly.snowfall`            | Snowfall per hour                          | inches    |
| `hourly.rain`                | Rainfall per hour                          | inches    |
| `hourly.precipitation_probability` | Probability of precip (rain or snow) | %         |
| `hourly.weathercode`         | WMO weather code                           | integer   |
| `hourly.time`                | ISO timestamps for each hour               | string    |

### WMO Weather Codes

**Snow:** 71 (slight), 73 (moderate), 75 (heavy), 77 (snow grains), 85 (slight showers), 86 (heavy showers)

**Rain:** 51, 53, 55 (drizzle), 56, 57 (freezing drizzle), 61, 63, 65 (rain), 66, 67 (freezing rain), 80, 81, 82 (showers), 95, 96, 99 (thunderstorm)

**Strategy:** Classify each hour by checking `weathercode` against snow/rain code lists OR checking `snowfall > 0` / `rain > 0`.

---

## App Architecture

### Event-Based Model

Rather than just finding "snow hours," the app groups consecutive precipitation hours into discrete **events**. This handles real-world forecasts like "rain in the morning, then snow overnight" as two separate alerts.

**Grouping rules:**
- Consecutive hours of the same precip type are merged into one event
- Gaps of up to 3 hours are tolerated within an event
- A type change (rain → snow) always starts a new event

### Core Flow

```
main(config)
  → parse location + interval from schema config
  → fetch forecast from Open-Meteo (cached 30min)
  → classify each hour as snow, rain, or clear
  → group into events
  → for each event: determine severity, duration, day label, probability, accumulation
  → render animated alert frames (or "all clear" with frog)
```

### Severity System

Accumulation determines severity level (light/moderate/heavy), which drives:
- Header text color (Line 1)
- Supporting text brightness (Lines 2-3)
- Which icon variant is shown

| Severity | Snow Accum | Rain Accum |
|----------|-----------|-----------|
| Light    | < 1"      | < 0.25"   |
| Moderate | 1-4"      | 0.25-1"   |
| Heavy    | > 4"      | > 1"      |

### Particle Animation

Each event renders as 32 frames at 100ms delay (~3.2 second cycle):
- **Snow:** 12 particles, 1x1px dim dots, speed=0.4-0.8, with horizontal drift + sine wave
- **Rain:** 16 particles, 1x2px dim streaks, speed=1.4-2.0, straight down
- Particles are layered behind text using `render.Stack`
- No particles on the All Clear screen

---

## File Structure

```
SkyKnow/
├── sky_know.star      # Main app file
├── manifest.yaml      # App metadata (for Tronbyt)
└── SkyKnow.md         # This file
```

---

## Configuration (Schema)

| Field | Type | Description |
|-------|------|-------------|
| `location` | `schema.Location` | Location picker — returns JSON with lat, lng, timezone |
| `interval` | `schema.Dropdown` | Forecast window: 24h, 48h, 3d, 5d, 7d (default: 48h) |

---

## Test Mode

Pass `test_mode` as a config parameter to render with fake data:

```bash
pixlet render sky_know.star test_mode=snow_light    # Light snow alert
pixlet render sky_know.star test_mode=snow_moderate  # Moderate snow alert
pixlet render sky_know.star test_mode=snow_heavy     # Heavy snow alert
pixlet render sky_know.star test_mode=snow_now       # Snow happening now
pixlet render sky_know.star test_mode=rain_light     # Light rain alert
pixlet render sky_know.star test_mode=rain_moderate  # Moderate rain alert
pixlet render sky_know.star test_mode=rain_heavy     # Heavy rain alert
pixlet render sky_know.star test_mode=rain_now       # Rain happening now
pixlet render sky_know.star test_mode=mixed          # Rain → snow multi-event
pixlet render sky_know.star test_mode=clear          # All clear screen (with frog)
pixlet render sky_know.star test_mode=error          # API error screen
```

Useful flags:
- `--gif` — output as animated GIF instead of WebP
- `-m 4` — 4x magnification for debugging
- `-o filename` — custom output path

---

## Development Workflow

### 1. Install Pixlet

```bash
# macOS
brew install tidbyt/tidbyt/pixlet

# Linux — download from GitHub releases
# https://github.com/tidbyt/pixlet/releases
```

### 2. Preview Locally

```bash
# Render a single frame
pixlet render sky_know.star

# Serve with live preview in browser (hot reload)
pixlet serve sky_know.star
# Then open http://localhost:8080
```

### 3. Push to Your Tidbyt via Tronbyt

```bash
# If using Tronbyt, follow their custom app upload process
# If using pixlet push directly:
pixlet render sky_know.star
pixlet push  sky_know.webp
```

---

## Enhancement Ideas (Future)

- **Wind chill / temperature** — add context to the alert
- **Multi-day view** — if snow is expected on multiple separate days
- **Integration with iOS Shortcuts** — trigger a push notification to your phone when the app detects snow (requires a webhook setup)
- **Configurable particle density** — let users adjust animation intensity
- **Hide when clear** — option to return `[]` from `main()` so app doesn't show in rotation
- **Frog variants** — seasonal frog (Santa hat in winter, sunglasses in summer)

---

## Quick Reference Links

- [Open-Meteo API Docs](https://open-meteo.com/en/docs)
- [Pixlet Documentation](https://tidbyt.dev/docs/build/build-for-tidbyt)
- [Starlark Language Spec](https://github.com/bazelbuild/starlark/blob/master/spec.md)
- [Tidbyt Render Package](https://tidbyt.dev/docs/reference/render)
- [Tidbyt Schema Package](https://tidbyt.dev/docs/reference/schema)
- [Tronbyt GitHub](https://github.com/tronbyt)
- [WMO Weather Codes Reference](https://open-meteo.com/en/docs#weathervariables)
