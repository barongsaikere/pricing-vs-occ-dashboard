# Arkamara Hotel — Pricing & Occupancy Dashboard

## Project Overview
Static HTML dashboard for monitoring hotel pricing strategy and occupancy performance. Built for Arkamara Hotel's ecommerce team.

- **Stack:** Single-file HTML, Chart.js 4.4.1, vanilla JS, inline CSS
- **Deploy:** Vercel (auto-deploy from `master` branch)
- **Staging:** https://pricing-vs-occ-dashboard.vercel.app/dashboard.html
- **Repo:** https://github.com/barongsaikere/pricing-vs-occ-dashboard

## File Structure
- `index.html` — Landing/home page
- `dashboard.html` — Main dashboard (all logic in one file: CSS + HTML + JS, ~2400+ lines)
- `forecast_stat_arkamara.csv` — Daily room statistics (791 rows, 2024-12-31 to 2027-03-01)

## Architecture
The dashboard is a single monolithic HTML file with:
1. **Inline CSS** (~115 lines) — Dark theme, responsive grid layout, `.dm-card`, `.dm-table`, `.sig-*`, `.zone-*`, `.panic-box`
2. **HTML sections** — Channel Rate Parity, Occupancy & Revenue, Daily Pricing Monitor, Gap Analysis
3. **Inline JS** (lines 1740+) — Chart.js charts, table generation, filter logic, Pearson correlation

All data is hardcoded in JS variables (no API calls):
- `chEvo` / `chDates` — Channel price evolution per check-in date (10 OTA channels)
- `pd` — Daily pricing comparison (website vs booking vs best external)
- `oa` — Monthly occupancy, ARR, RevPAR, totalRevenue (completed + future)
- `violData` — Rate parity violation counts per channel
- `dailyOcc` — Daily occupancy percentages extracted from forecast_stat_arkamara.csv
- `dm` — Daily monitor data array (gap, signal, zone, lead time per date)
- `enriched` — Extended daily data with per-competitor pricing from chEvo

## Key Sections
1. **Channel Rate Parity Control** — Monitors if website is cheapest, violations per OTA
2. **Occupancy & Revenue Analysis** — OCC/ARR charts, pricing vs competitors
3. **Daily Pricing Control Room** — Lead time scatter chart, panic cost box, action table with signal/zone filters
4. **Gap Analysis — Price Position vs Occupancy** — Multi-chart analysis:
   - 6 KPI cards (Avg OCC, Competitive OCC, Overpriced OCC, OCC Gap Delta, Revenue Opportunity, Empty Room-Nights)
   - Price Gap vs Daily OCC scatter (colored by month, linear trend line)
   - Avg Occupancy by Price Position bar chart (3 tiers: competitive/at-market/overpriced)
   - "What Drives Occupancy?" correlation bar chart (6 factors incl. competitor pricing)
   - "Competitor Landscape — Who Beats Us?" undercut analysis chart
   - Monthly Gap Summary table
5. **Occupancy-First Pricing Insights** — Actionable recommendations

## Business Context
- Hotel property: Arkamara Hotel (~43 rooms, varies by date due to renovations)
- Currency: IDR (Indonesian Rupiah)
- Channels: Official Site, Booking.com, Expedia, Agoda, Trip.com, tiket.com, Traveloka, klook, Tripadvisor, Bluepillow.id
- Core problem: Ecommerce team tends to panic-discount when they see low OCC for future months
- Feb 2026 evidence: 85.7% OCC but lowest ARR (1.37M) — panic-discounting hurt revenue
- Dashboard goal: Daily confidence signals to prevent premature discounting while maximizing OCC

## Signal Logic (OCC-First Strategy)
Strategy: Maximize occupancy. Every guest = room + F&B + spa + ancillary revenue.
- **REDUCE (gap >3%):** Overpriced vs market — match or beat to capture bookings
- **WATCH (gap 0-3%):** Slightly above market — monitor pickup daily
- **HOLD (gap 0 to -3%):** At market rate — price is right, focus on distribution
- **COMPETITIVE (gap < -3%):** Below market — push marketing to maximize bookings
Note: "RAISE" was deliberately removed. Business priority is filling rooms, not maximizing ADR.

## Data Model
- `dailyOcc` maps date strings to `total_sold_persen` from forecast CSV (118 dates, Mar 6 - Jul 4, 2026)
- `enriched` array adds per-date competitor pricing: `cheaperCount`, `spread`, `avgComp`, `minComp` from chEvo
- Multi-factor Pearson correlation analyzes 6 dimensions: gap, website price, cheapest competitor, avg competitor, # channels cheaper, market spread
- Key finding: #1 OCC driver is "# channels cheaper than us" (r=-0.29), not absolute price (r=-0.04)

## Charts (Chart.js instances)
- `channelEvoChart` — Channel price evolution line chart
- `violationsChart` — Rate parity violations bar chart
- `priceChart` — Website vs competitors line chart
- `occArrChart` — Monthly OCC/ARR dual-axis chart
- `leadTimeChart` — Price position vs lead time scatter
- `gapOccScatter` — Price gap vs daily OCC scatter (colored by month)
- `tierBarChart` — Average OCC by price position tier
- `corrChart` — Multi-factor correlation horizontal bar chart
- `undercutChart` — Competitor undercut analysis bar chart

## Conventions
- Language: Dashboard UI in English, user communication in Bahasa Indonesia
- Styling: Dark theme (#0f172a background), Tailwind-inspired color palette
- Charts: Chart.js scatter, line, bar — responsive, interactive tooltips
- Data format: Prices in raw IDR (displayed as X.XXM), dates as YYYY-MM-DD
- File is large (~96K tokens) — use offset/limit when reading with Read tool
