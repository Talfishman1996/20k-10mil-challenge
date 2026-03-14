# TradeVault

Mobile-first trading risk dashboard with adaptive position sizing. Personal tool for tracking the $20K to $10M equity challenge.

**Live:** [talfishman1996.github.io/trade-challenge](https://talfishman1996.github.io/trade-challenge/)

## Stack

- React 19 + Vite 7
- Tailwind CSS v4 (via `@tailwindcss/vite`)
- Recharts (charts/visualizations)
- Framer Motion (animations)
- Lucide React (icons)
- localStorage for persistence (no backend except optional cloud sync)

## Quick Start

```bash
npm install
npm run dev       # http://localhost:5173/trade-challenge/
npm run build     # Production build -> dist/
```

## Architecture

```
src/
  main.jsx                  # Entry point
  index.css                 # Tailwind + custom styles
  sync.js                   # Cloud sync (Cloudflare Workers KV)
  math/
    constants.js            # Model constants, milestones, chart config
    risk.js                 # 2/3 Power Decay model (core math)
    format.js               # Number formatting utilities
    monte-carlo.js          # Monte Carlo simulation engine
    analytics.js            # Trade analytics (streaks, R-multiples, etc.)
    index.js                # Barrel export
  store/
    trades.js               # Trade state management + localStorage
    settings.js             # Settings state + localStorage
    tags.js                 # Tag definitions
  utils/
    dataIO.js               # JSON import/export
    imageDB.js              # Trade image storage (IndexedDB)
  components/
    App.jsx                 # Root: navigation, tabs, layout, sync orchestration
    Home.jsx                # Dashboard: equity display, stats, sparkline, summit tracker
    Trades.jsx              # Trade log: grid/table views, stats bar, sorting
    TradeEntry.jsx          # Trade form: win/loss, long/short, P&L, dates, notes
    Analysis.jsx            # Analytics: milestones, risk curves, stress test, projections
    Settings.jsx            # Settings: risk params, cloud sync, data management
    EquityCurve.jsx         # Recharts equity curve with milestone markers
    Heatmap.jsx             # Calendar heatmap of daily P&L
    ScatterPlot.jsx         # R-multiple scatter plot
    GPSJourney.jsx          # Visual equity funnel (danger zone -> goal)
    FilterBar.jsx           # Date/tag/direction filters
    MetricCard.jsx          # Reusable stat card component
    Celebration.jsx         # Milestone achievement animation
    TagPicker.jsx           # Multi-select tag picker
    shared/                 # Shared component utilities
worker/
  src/index.js              # Cloudflare Worker (sync API)
  wrangler.toml             # Worker config
.github/
  workflows/                # GitHub Pages deploy on push to master
```

## Core Math Model: 2/3 Power Decay

The position sizing engine uses a piecewise risk function `rN(equity)`:

| Equity Range | Risk % | Behavior |
|-------------|--------|----------|
| $0 - $20K | 100% | All-in (survival mode) |
| $20K - $50K | 100% -> 50% | Linear decrease |
| $50K - $87.5K | 50% -> 33% | Linear decrease to anchor |
| $87.5K+ | 33% * (87500/E)^(2/3) | Power decay |

**Key constants:**
- Anchor equity (E0): $87,500 (risk = 33% here)
- Start: $20,000
- Target: $10,000,000

**At scale:** Risk decays smoothly -- e.g., ~8.5% at $500K, ~4.4% at $1M, ~1.4% at $5M.

The model balances aggressive growth at low equity with capital preservation at high equity, using a continuous curve (no cliff edges).

## Features

### Dashboard (Home)
- Current equity with animated display
- 30-day P&L, best trade, current streak, win rate
- Mini equity sparkline
- Summit Tracker (milestone progress trail)
- Phase indicator (Growth / Anchor / Decay)

### Trade Logging
- Win/Loss + Long/Short entry
- Open date, close date, duration tracking
- Notes and tags per trade
- Undo/redo support
- Grid view (cards) and Table view (sortable)

### Analysis
- **Milestones:** Progress toward $100K, $250K, $500K, $1M, $4M, $10M
- **Data Matrix:** Risk/reward at various equity levels
- **Risk Curves:** Visual comparison of 2/3 Power Decay vs alternatives
- **Stress Test:** Consecutive win/loss streak simulator
- **Projections:** Monte Carlo growth simulation
- **Compare:** Side-by-side model comparison

### Settings
- Adjustable risk parameters (anchor, base risk, reward ratio)
- R-multiple toggle
- Cloud sync management
- Data import/export (JSON)
- Clear data with confirmation

### Cloud Sync
- Cloudflare Workers + KV backend
- Auto-sync on trade entry, mount, and 60s interval
- Share via URL hash (`#sync=KEY_ID`)
- Timestamp-based conflict resolution

## Design

- Dark theme: slate-950 background
- Semantic colors: emerald (positive/wins), rose/red (negative/losses), blue (UI chrome), amber (warnings)
- JetBrains Mono for all numeric displays
- Mobile-first: bottom tab navigation + center FAB for trade entry
- Desktop: fixed left sidebar with icon navigation

## Deployment

Auto-deploys to GitHub Pages on push to `master` via GitHub Actions. Vite base path is `/trade-challenge/`.

## License

ISC
