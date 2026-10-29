# PRD: 2N Rack Power Model

## Problem & user
Who it's for: a data centre technician planning a rack install, and recruiters reviewing my portfolio
Problem it solves: estimates what a dual-corded (2N) rack really draws from each feed, using independent 80 PLUS lab test data for real server power supplies, and checks the rack stays within limits if either feed fails

## Electrical basis (Australia)
### Supply and loads
- Single-phase: 230 V, 50 Hz. Three-phase: 400 V line-to-line, 230 V line-to-neutral
- Each single-phase load on a three-phase PDU connects line-to-neutral (230 V) on L1, L2 or L3
- Each server and fixed load is dual-corded (shared across feeds A and B) or single-corded (assigned to feed A or B). A single-corded load loses power if its feed fails and is listed as "loses power"
- Fixed loads (e.g. a switch) are entered directly as AC W and PF, and keep the same AC W and PF when they move to one feed
- Server DC load = power delivered by the PSUs (DC side), not AC wall power

### PSU data and model
- PSU test data (from 80 PLUS reports, 230 V input): at 10/20/50/100% of rated output, the output W, input W, input current (Irms), power factor (PF), current THD and efficiency
- PSU test data was measured at 230 V, 60 Hz; I apply it at 50 Hz as a stated assumption
- PSU load fraction = DC power that PSU supplies ÷ its rated output
- Between tested points: linear interpolation of efficiency and PF (my stated assumption)
- Below 10% or above 100% load: show "outside tested range"; never extrapolate
- Input power P_in = P_out ÷ efficiency
- Apparent power S = P_in ÷ PF
- Input current I = S ÷ 230 V
- Dual PSUs share the server's DC load equally in normal operation (my stated assumption; some servers use hot standby instead). The unused PSU in a single-PSU comparison is assumed to draw 0 W
- If a feed fails, the remaining PSU carries the server's full DC load at its new load fraction
- If a surviving PSU's DC load exceeds its rated output (100% load), flag "PSU overloaded" as a failure

### Cross-check (data integrity)
- Computed I is compared against the report's Irms at each tested point
- A row is flagged when the difference is more than 2% (editable). Why 2%: the reports round input W, PF and Irms, which explains differences up to about 0.7% on normal rows; 2% sits above that and below the one real discrepancy found (YSEC2000AM at 20% load, +2.5%)
- Flagged data is kept exactly as published and the discrepancy is noted on the Method page

### Rack limits
- Phase current = sum of device currents on that phase (added arithmetically; typically conservative when PFs differ)
- Single-phase capacity = 230 V × rated current; three-phase capacity = √3 × 400 V × rated current per phase
- Derated limit = breaker rating × derating factor (default 80%, editable; common planning practice, site standard decides)
- Phase imbalance % = (highest phase current − average) ÷ average × 100
- Phase imbalance is shown only when average phase current is above 1 A (editable), because the percentage is meaningless at tiny currents
- Imbalance warning at 20% (editable; a rule of thumb, not a standard)
- Steady-state only: no inrush, breaker trip curves or voltage variation

### Heat and cost
- Heat: 1 W = 3.412 BTU/h
- Redundancy cost = rack input power with PSUs sharing − rack input power if one PSU per server carried the full load

## MVP features (only what v1 can't live without)
1. PSU Library: 2–3 real server PSUs from content/psus.json, each with its tested points, charts of efficiency vs load and PF vs load, a link to its report, and a cross-check column (computed I vs reported Irms, flagged above the threshold)
2. Server builder: DC load per server (W), chosen PSU model, number of PSUs (1 or 2), dual-corded or single-corded (feed A or B)
3. Rack builder: PDU type per feed (1-phase 16 A, 1-phase 32 A, 3-phase 16 A, 3-phase 32 A), servers and fixed loads with quantities, each assigned to L1/L2/L3
4. Load view: per-phase current on each feed vs its derated limit, as bars, for Normal, Feed A lost and Feed B lost. Loads that lose power are listed
5. Warnings: any phase over its derated limit in any state, any PSU outside its tested range, any PSU overloaded, any load that loses power, any imbalance above the threshold
6. Redundancy cost panel: extra input W, extra heat (BTU/h) and the PF each PSU runs at, sharing vs single PSU
7. "Show the working": every KPI and warning expands to show the formula with the actual numbers plugged in
8. Two presets: "Looks fine until a feed fails" (fixed loads) and "Real servers" (built from my PSU data)
9. Method page: my explanations, assumptions, thresholds and sources

Not in v1: login, database, saving racks, cooling simulation

## User flow
Lands on PSU Library → sees real test data and how PF and efficiency fall at low load → opens Rack Builder → loads "Looks fine until a feed fails" → switches to Feed A lost → sees a phase go red → expands "show the working" → loads "Real servers" → reads the redundancy cost → reads Method page

## Data
- content/psus.json: per PSU: manufacturer, model, rated_output_w, report_url, test_date, points[{load_percent, output_w, input_w, irms_a, pf, ithd_percent, efficiency_percent}]
- Rack state lives in the browser only

## Tech stack & why
- Next.js + Tailwind + shadcn/ui: frontend
- Vercel: hosting
- All electrical maths in pure TypeScript functions in /lib/power.ts, unit-tested

## Design direction

Principle: minimal and intuitive first. The UX stays clean and obvious, and the character comes from small UI details (type, colour, thin rules), never from extra clutter. It should feel designed by a person, not generated from a template, without ever getting in the way of reading the numbers.

- Feel: minimal, precise, technical, calm, editorial

### References (saved in /design-refs)
1. ref-1-design-wrap (creative agency site): take the contrast between large monospaced/pixel-style display text and plain sans text, the thin horizontal rules between rows, and one bright accent on an otherwise black-and-white page. Leave out the scroll effects, the oversized hero text and the imagery. Keep it far more minimal and professional.
2. ref-2-financial-dashboard: take the colour scheme (warm off-white/light grey, near-black text, one coral-orange accent) and the flat rounded cards. Make it simpler and more intuitive: fewer widgets, no avatars, no decorative icons, no greeting banner.
3. ref-3-wix-dashboard: take the clean layout: a small label above a large number, generous whitespace, list rows with a thin divider, one clear primary action per area. This is the model for the app's UX.
4. ref-4-cake-quicken: take the restraint: flat panels with 1px borders, small muted labels, lots of empty space.

### Layout and UX
- Top navigation with three tabs (PSU Library / Rack Builder / Method). No sidebar.
- One main question per page. The answer comes first, as a one-line verdict (for example "Rack survives losing either feed" or "Feed B overloads if Feed A fails"), then the supporting numbers.
- Detail on demand: formulas and working stay collapsed until "Show the working" is opened.
- Key figures shown as a small label above a large number with its unit.
- Desktop: inputs on the left, results on the right. Phone: one column, results first.
- No more than one primary button visible per section.

### Colour (starting values; check text contrast is at least 4.5:1)
- Page background: #F6F5F2 (warm off-white). Surfaces: #FFFFFF with a 1px border #E3E1DC. No drop shadows.
- Text: #111111. Muted labels: #6B6B6B.
- One accent colour only: coral #E8583A, used for the selected tab, the primary button, focus rings and one highlighted figure per page. Never used for load status.
- Status colours are used ONLY in load bars and status badges: OK #2E7D32, near limit #B26A00, over limit #C62828. Every status also carries a text label ("OK", "Near limit", "Over limit"), so colour is never the only signal.

### Type
- Page titles and every number, unit and formula: a monospaced font (for example JetBrains Mono or Space Mono). Titles can be large.
- Body text and labels: a clean sans that is not Inter (for example DM Sans or Instrument Sans).
- Two fonts in total. Units always visible (V, A, W, VA, kW, BTU/h, %).

### Components
- Thin horizontal rules between table rows and list items, as in reference 1.
- Flat cards with a 1px border and a small corner radius.
- Charts: thin lines, no gradients or fills, direct labels instead of legends where possible, muted gridlines.
- Subtle motion only: bars animate to their new value and expandable sections ease open.

### Avoid
- Purple gradients, Inter everywhere, identical shadowed cards, stock hero + 3 cards, emoji, decorative icons, more than one accent colour, drop shadows, gradients.
- Must look good on phone and desktop.

## Milestones (build in order, test each before moving on)
1. Project setup + layout shell + design system: colours, fonts and spacing defined once in a single tokens file, plus the three-tab navigation (PSU Library / Rack Builder / Method)
2. PSU Library page from content/psus.json + charts + Irms cross-check
3. /lib/power.ts with every formula above + unit tests (no UI changes)
4. Rack builder: PDUs, servers, fixed loads, cord type, phase assignment, Normal-state load view
5. Failover states + warnings + redundancy cost + "show the working" + presets
6. Method page + README + polish: mobile check, accessibility

## Definition of done (v1)
A visitor can build a rack from real PSU test data, see whether it survives losing either feed, see what redundancy costs in power and heat, and follow the arithmetic behind every number. It looks clean, minimal and deliberately designed, and the verdict on each page is clear within a few seconds.
