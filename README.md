# SkyKnow

**Precipitation alert app for Tidbyt/Tronbyt with animated weather effects and a friendly tree frog.**

Know when it'll rain or snow—and how long it'll last.

<div align="center">
<pre>
    @....@
   (------)
  (  ^vv^  )
 (   ~~~~   )
  ^^  ~~  ^^
</pre>
</div>

---

## Features

- **Event Duration Display** — Shows how long precipitation will last, not just when it starts
- **Smart Day Labels** — Contextual labels like "Today", "Tonight", "Tomorrow", or specific weekdays
- **Animated Particles** — Snowflakes with sine wave drift for snow, vertical streaks for rain
- **Severity-Based Colors** — Blue/cyan for light, amber for moderate, red/orange-red for heavy
- **Animated Frog** — A charming pixel art tree frog that blinks and puffs its throat on all-clear days
- **Multi-Event Support** — Handles complex forecasts like "rain in the morning, snow overnight"
- **Free Weather API** — Uses Open-Meteo (no API key required, 10k requests/day)

---

## Display Examples

### Precipitation Alert
```
┌──────────────────┐
│ ❄ SNOW 6h        │  ← Severity-colored header (blue/amber/red)
│ Tomorrow          │  ← Smart day label (dimmed gray)
│ 85% 3-6"          │  ← Probability + accumulation
└──────────────────┘
   + animated falling particles
```

### All Clear Screen
```
┌──────────────────┐
│       🐸         │  ← Animated tree frog (blinks + throat puff)
│    ALL CLEAR     │
│     NEXT 2D      │
└──────────────────┘
```

---

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/wurtz/SkyKnow.git
cd SkyKnow
```

### 2. Install Pixlet (for local testing)
```bash
# macOS
brew install tidbyt/tidbyt/pixlet

# Linux - download from GitHub releases
# https://github.com/tidbyt/pixlet/releases
```

### 3. Test Locally
```bash
# Render with test data
pixlet render sky_know.star test_mode=snow_moderate

# Live preview in browser (hot reload)
pixlet serve sky_know.star
# Open http://localhost:8080
```

---

## Deployment

### Upload to Tronbyt
1. Go to your Tronbyt web interface
2. Upload `sky_know.star` (the YAML file is not needed)
3. Configure your location and forecast window
4. Done! The app will appear in your device rotation

---

## Configuration

| Setting | Options | Description |
|---------|---------|-------------|
| **Location** | Location picker | Your location for weather forecasts |
| **Forecast Window** | 24h, 48h, 3d, 5d, 7d | How far ahead to check for precipitation (default: 48h) |

---

## Test Modes

Test the app with fake data to see all display variants:

```bash
# Snow variants
pixlet render sky_know.star test_mode=snow_light     # Light snow, 6h away
pixlet render sky_know.star test_mode=snow_moderate  # Moderate snow, tomorrow
pixlet render sky_know.star test_mode=snow_heavy     # Heavy snow, 12h away
pixlet render sky_know.star test_mode=snow_now       # Snow happening now

# Rain variants
pixlet render sky_know.star test_mode=rain_light     # Light rain, 2d away
pixlet render sky_know.star test_mode=rain_moderate  # Moderate rain, tomorrow
pixlet render sky_know.star test_mode=rain_heavy     # Heavy rain, 4h away
pixlet render sky_know.star test_mode=rain_now       # Rain happening now

# Other
pixlet render sky_know.star test_mode=mixed          # Multiple events
pixlet render sky_know.star test_mode=clear          # All clear (with frog!)
pixlet render sky_know.star test_mode=error          # API error screen
```

**Pro tip:** Add `--gif -m 4 -o output.gif` to export magnified animated GIFs for debugging!

---

## How It Works

1. **Fetches hourly forecast** from Open-Meteo API (30-min cache)
2. **Classifies hours** as snow, rain, or clear using WMO weather codes + accumulation data
3. **Groups into events** — consecutive hours of the same type (tolerates 3-hour gaps)
4. **Calculates severity** based on accumulation (light/moderate/heavy)
5. **Renders alert** with animated particles, or shows the frog if all clear

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Language** | Starlark (Python-like, used by Tidbyt) |
| **Weather API** | [Open-Meteo](https://open-meteo.com/en/docs) (free, no API key) |
| **Rendering** | Tidbyt `render` and `schema` modules |
| **Dev Tools** | [Pixlet CLI](https://tidbyt.dev/docs/build/build-for-tidbyt) |

---

## Project Structure

```
SkyKnow/
├── sky_know.star              # Main app (deploy this to Tronbyt)
├── manifest.yaml              # App metadata
├── SkyKnow.md                 # Full project specification
├── IMPLEMENTATION_SUMMARY.md  # Implementation notes
├── .gitignore                 # Protects private files
└── README.md                  # This file
```

---

## Severity System

Accumulation determines alert severity:

| Severity | Snow | Rain |
|----------|------|------|
| **Light** | < 1" | < 0.25" |
| **Moderate** | 1-4" | 0.25-1" |
| **Heavy** | > 4" | > 1" |

Each severity level has its own color scheme:
- **Light:** Blue (snow) / Cyan (rain)
- **Moderate:** Amber
- **Heavy:** Red (snow) / Orange-Red (rain)

---

## Particle Animation

Each event renders as 32 frames at 100ms delay (~3.2 second loop):

- **Snow:** 12 particles, 1x1px dots, speeds 0.4-0.8, horizontal sine wave drift
- **Rain:** 16 particles, 1x2px streaks, speeds 1.4-2.0, straight vertical fall

---

## Frog Animation

The all-clear screen features a 13x11 pixel tree frog with three states:

- **Idle** — Half-lidded "not impressed" expression (5 seconds)
- **Blink** — Eyes close briefly (0.4 seconds)
- **Throat Puff** — Yellow belly expands up into chin (0.6 seconds)

Total cycle: 90 frames @ 100ms = 9 seconds

---

## Future Ideas

- Wind chill / temperature context
- Multi-day view for separate events
- iOS Shortcuts integration for push notifications
- Configurable particle density
- Option to hide when clear (return empty from `main()`)
- Seasonal frog variants (Santa hat in winter, sunglasses in summer)

---

## Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

## Credits

- **Weather Data:** [Open-Meteo](https://open-meteo.com) (free weather API)
- **Platform:** [Tidbyt](https://tidbyt.com) / [Tronbyt](https://github.com/tronbyt)
- **Development:** Built with help from [Claude](https://claude.ai) (overnight shift 🤖)

---

## License

MIT

---

**Made with ☁️ and 🐸 by wurtz**
