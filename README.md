# Junk Removal Fee Schedule — Daddy's Junk Removal (Las Vegas, NV)

An interactive, self-hosted pricing calculator and quoting/invoicing tool for a solo-operator junk removal business running a 2020 Ram 1500 3.6L V6 4x4 towing a 4,000 lb dump trailer in the Las Vegas, NV area.

- **Mobile (iPhone-optimized):** [ackim178.github.io/junk-removal-fee-schedule](https://ackim178.github.io/junk-removal-fee-schedule/)
- **Desktop / laptop:** [ackim178.github.io/junk-removal-fee-schedule/desktop.html](https://ackim178.github.io/junk-removal-fee-schedule/desktop.html)

The root URL auto-detects your device (via `navigator.userAgent`) and routes you to the right layout. You can force either view by adding `?view=mobile` or `?view=desktop` to the URL.

## What this is

Every number on the page — gas price, trailer capacity, dump fees, labor rate, item prices, everything — is a **live input**, not a hard-coded table. Change any assumption and every price on the page recalculates instantly. Your edits are saved automatically to your browser's `localStorage`, so they persist between visits on the same device.

The tool has three parts:

1. **Cost assumptions** — the inputs that drive every price (fuel, trailer, dump fees, labor, markup).
2. **Pricing output** — a load-based pricing table (price by how full your trailer is) and an itemized "price by item" quick-reference table with checkboxes for building a customer quote.
3. **Quoting & invoicing** — check off the items a customer wants removed, see a running total, copy a plain-text quote, or generate and download a professional PDF invoice with your logo and business info (no internal cost data included).

## How the pricing model works

The core idea, borrowed from how most professional junk removal companies actually price jobs, is **"load fraction" pricing**: instead of pricing every possible item individually, you price based on what *fraction of your trailer's capacity* a job fills. A single chair might be 1/8 of a load; a full house cleanout is a full load. The per-item "Quick reference" table is just this same load-fraction formula applied to each item's estimated volume, so both pricing methods always agree with each other.

### Step 1 — What does a trip actually cost you?

For any load fraction `f` (0 = nothing, 1 = a completely full trailer), the model builds up a real dollar cost from four pieces:

| Cost component | Formula | What it represents |
|---|---|---|
| **Base stop cost** | `(localMiles / localMpg) × gasPrice + localMiles × vehicleCostPerMile` | Gas + vehicle wear for driving from your base to the job site and back — a fixed cost you incur on every job regardless of size |
| **Labor time cost** | `f × jobTimeHr × laborRate` | Your own hands-on-and-driving time for the whole job — arrival/walkthrough, loading the trailer, the drive to the dump and back, and unloading there — scaled by how much of the trailer you're filling (e.g. a 1/4 load bills 1/4 of your full-load job time) |
| **Dump fee** | `(f × trailerPayloadLbs / 2000) × dumpFeePerTon` | The actual landfill/transfer-station weight fee for your share of a full load |
| **Dump trip share** | `f × [ (dumpMiles / towMpg) × gasPrice + dumpMiles × vehicleCostPerMile ]` | Your share of the gas + vehicle wear for the round-trip drive to the dump and back, prorated by how much of the trailer this job's items represent |

Add all four together and you get the true, fully-loaded cost of doing that job — fuel, vehicle depreciation, dump tipping fees, *and* your own time, all in dollars.

`jobTimeHr` is one editable input right above the "Load-based pricing" table, with an hour dropdown (0–24) and a minute dropdown (0/15/30/45 in quarter-hour steps). It's intentionally a single number rather than separate walkthrough/loading/unloading fields — on a real job those all run together, so you just enter how long a **full load**, start to finish, actually takes you (default: **2.5 hours**, matching a realistic full-load job in the Las Vegas market), and the calculator prorates it for smaller jobs automatically. Every price on the page — load table and per-item table alike — recalculates instantly when you change it.

### Step 2 — Turn cost into a price

```
price = max( minimumJobPrice, roundToNiceNumber( (cost + extraFees) × markupMultiplier ) )
```

- **`markupMultiplier`** (default `2.0×`) — covers profit margin plus overhead that isn't captured hour-by-hour (insurance, tools/trailer maintenance reserve, slow days, no-shows, marketing), *and* positions pricing to match what the Las Vegas market actually bears rather than just your bare cost-plus-labor number.
- **`extraFees`** — a flat Freon-handling fee (`$40` default) for refrigerant-containing appliances, or a hazmat-handling fee (`$30` default) for motor oil/other household hazardous waste — these are added to the cost *before* markup, since they're pass-through compliance costs, not variable dump-weight costs.
- **Rounding** — final prices round to the nearest $5 under $150, nearest $10 under $500, and nearest $25 above that, so quotes look like clean, professional numbers instead of "$127.43."

### Two separate minimum-price floors — and why they're not the same number

The tool applies **two different floors** rather than one shared `minimumPrice`, because a whole-job trip and a single small item have very different real minimum-time costs:

- **`minimumPrice`** (default `$240`, applied to the **load-based table**) — the floor for a whole job/trip. It assumes at least ~1 hour of real hands-on-and-driving time (drive there, walk through, load, drive to the dump, unload) is unavoidable on *any* dispatch, priced at your labor rate: `baseStopCost + 1hr × laborRate + dump fee/share, then × markup`. This keeps the smallest whole-job quote from underpaying you relative to the labor rate you actually need to hit your take-home target.
- **`itemMinimumPrice`** (default `$89`, applied to the **Quick reference: price by item** table) — a lower floor for a *single item within a job*. It's intentionally lower than `minimumPrice` because the per-item table is meant for building up a multi-item quote (or a walk-up single-item job) where several small items can share one trip's fixed driving/dump costs — if every item individually had to clear the full $240 trip minimum, a 3-item pickup would price absurdly high, and the smallest items (a chair, a lamp) would all be indistinguishable from a hot tub.

Because most individual furniture/appliance items are such a small fraction of the trailer's total volume (a chair is ~1.7% of a full load), their raw cost-based price rarely exceeds a low floor — so most of them cluster at the `itemMinimumPrice` floor, while genuinely large single items (sectional sofa, piano, hot tub, Freon appliances) price above it based on actual volume/weight. **Only use the per-item prices for genuinely small, one-or-two-item jobs** — for anything filling a meaningful fraction of the trailer, quote from the load-based table instead, which reflects your real per-job time/labor floor.

### Step 3 — Reverse-engineer your labor rate from a take-home goal

Because you're a solo operator, `laborRate` isn't a wage you pay someone else — it's 100% your own time, and it needs to cover more than just take-home pay. The "Solo take-home → labor rate helper" runs the math backwards:

```
pretaxRate      = targetTakeHome / (1 − combinedTaxRatePct / 100)
suggestedLaborRate = (pretaxRate + overheadPerHour) / (billableUtilizationPct / 100)
```

- **Combined tax rate** grosses up your target take-home to cover self-employment tax + federal income tax (Nevada has no state income tax).
- **Overhead per hour** adds a reserve for insurance, licensing, tools, and trailer upkeep.
- **Billable utilization** accounts for the fact that not every working hour is billable — slow days, driving between jobs, scheduling, and invoicing all eat into the hours you can actually charge for, so the rate on your *billable* hours has to cover your *total* hours worked.

Example with the defaults ($60/hr take-home, 28% tax, $6/hr overhead, 80% utilization): pretax rate = $83.33/hr → suggested labor rate ≈ **$111.67/hr** (the calculator's default `laborRate` of $112/hr matches this suggestion, rounded to a clean number).

### Putting it together: why a full load defaults to $700

With the defaults above (`jobTimeHr` = 2.5 hr, `laborRate` = $112/hr, `markup` = 2.0×, plus the standard dump-fee/mileage assumptions), a **full load** (`f = 1`) costs about **$353** to actually run, and `$353 × 2.0 ≈ $700` after rounding — matching the $700+ full-load rate Las Vegas competitors charge, while the $60/hr take-home target and 2.5-hour job-time floor make sure that price still reflects a realistic full day-of-work cost, not just "whatever the market will bear." Smaller loads scale down proportionally down to the `minimumPrice` floor of **$240** (1/4 load and below all floor there, since even a small job still needs ~1 hour of real drive/load/dump/unload time at your labor rate); a 1/2 load runs ≈$360, a 3/4 load ≈$525. Since every one of these numbers (take-home, job time, labor rate, markup, both minimums) is a live input, you can nudge any of them if your actual costs or market shift — none of these figures are hard-coded, they all fall out of the assumptions above.

## Where the disposal cost numbers came from

The two rate sheets originally provided (`RSSN-RATES-2025-2026.pdf` and `Republic_Services_Rates_Effective_July_1,2025-June_30,2026.pdf`) turned out to be **franchise billing rates** — what Republic Services/Clark County charges HOAs, businesses, motels, and medical/sewage-waste accounts for curbside collection contracts. That's a different business model than a self-haul junk removal operator, so they aren't used in this calculator. Instead, the model uses the actual **self-haul gate rate**, sourced from:

- **[Waste Today Magazine — Clark County/Apex Landfill franchise agreement](https://www.wastetodaymagazine.com/news/clark-county-nevada-landfill-republic-franchise/)** — confirms Apex Regional Landfill (the only landfill in Clark County, operated by Republic Services under a 20-year county franchise) and its posted self-haul gate rate of **$37.54/ton**, weighed on entry. This is the `dumpFeePerTon` default in the calculator.
- **[Republic Services — Southern Nevada](https://www.republicservices.com/municipality/southern-nevada)** — confirms the Cheyenne & Henderson Transfer Stations' hours (7 days/week, 7 a.m.–3 p.m.) and that most general household items are accepted, with refrigerant-containing appliances requiring separate handling.
- **[Nevada Division of Environmental Protection — household hazardous waste guidance](https://ndep.nv.gov/nevada-recycles/recycle/where-can-i-recycle)** — the basis for how Freon-containing appliances and other household hazardous waste (paint, batteries, propane, motor oil) are treated as special-handling items in the calculator, each carrying its own pass-through fee rather than being priced by weight alone.

Western Elite is mentioned as a private, non-franchise, in-town alternative that bills by the cubic yard instead of by weight — worth price-comparing for smaller loads since it's closer to the Strip, but no specific rate is baked into the model since it wasn't confirmed at the time of writing.

**These fees change.** Call Republic Services at 702-735-5151 or the scale house directly before finalizing a routing plan, and update the `dumpFeePerTon` input if the posted rate changes.

## Where the vehicle numbers came from

The truck/trailer assumptions are based on published manufacturer specifications for a **2020 Ram 1500 3.6L Pentastar V6 4x4** (Stellantis/Mopar):

- Max tow rating: roughly **6,390–7,730 lbs** depending on cab configuration, bed length, and axle ratio.
- Max payload (which must include the trailer's tongue weight, typically ~10–15% of a loaded bumper-pull trailer's weight): roughly **1,770–2,300 lbs**.

With a 4,000 lb empty trailer, that leaves roughly 2,400–3,700 lbs of usable cargo capacity at the absolute maximum. The calculator defaults `trailerPayloadLbs` to **2,800 lbs** — a conservative middle-of-the-range number that builds in a safety margin for Las Vegas heat, grades, and tongue weight. **Always verify your own truck's exact tow rating (door-jamb Tire & Loading placard) and your trailer's VIN data plate** rather than relying on this default, since it varies by exact configuration.

Trailer platform capacity (`trailerLengthFt` × `trailerWidthFt` × `trailerStackHeightFt` ÷ 27) is based on **measured trailer dimensions — an 18 ft × 6.5 ft platform, stacked up to 4 ft high** — which works out to **~17.3 cubic yards**. This is a hard physical ceiling on volume, but for most real loads the **2,800 lb payload limit is reached well before the platform is physically full**, since typical household junk is far less dense than that. Treat the payload number as the everyday constraint and the platform volume as the "how much can I physically fit" ceiling for unusually bulky-but-light loads (empty boxes, foam furniture, patio sets).

Real-world towing MPG (`towMpg`, default 12) is an estimate — towing typically cuts a V6 Ram 1500's unloaded EPA rating (19 city / 24 hwy) by roughly 40–50%. Track your own fuel-ups while towing loaded and adjust this input to match reality.

## Where the per-item volume & weight estimates came from

The "Quick reference: price by item" table (furniture, appliances, hazmat, specialty items) estimates each item's **volume in cubic yards** and **typical weight in pounds** based on general, widely-used industry knowledge of common household item dimensions and densities — not a single external database or published source. These are reasonable starting points, not measured facts about *your* customers' specific items, so:

- Every item price is derived from the *same* load-fraction formula above (`f = itemVolume / trailerCapacity`), just pre-computed for convenience — it isn't a separately-set arbitrary number.
- Items ≥150 lbs are flagged "Plan for 2 people" as a safety/labor-time note.
- Items whose volume exceeds your trailer's total capacity are flagged as possibly needing a second trip.
- If an item you regularly deal with prices oddly, the right fix is to adjust its cubic-yard/weight estimate to match what you actually observe in the field — these are meant to be tuned over time, not treated as fixed truth.

Additional fees & surcharges (long carry, rush/same-day service, after-hours, disassembly, no-access) reflect common industry practice for junk removal and moving-adjacent services, not a specific cited source — they're flat/percentage adjustments you can tune to your own market and pain tolerance.

## Feature summary

- **Live cost model** — every assumption (fuel price, MPG, trailer size, dump fee, labor rate, markup, the two separate minimum-price floors, full-load job time) is editable and recalculates all output instantly.
- **Solo take-home → labor rate helper** — reverse-calculates the labor rate you need to bill to hit a real take-home target after tax, overhead, and utilization.
- **Load-based pricing table** — price by trailer-fraction (1/8 load through full load).
- **Quick-reference item pricing** — pre-computed prices for common furniture, appliances, hazmat, and specialty items.
- **Dark / light mode** — toggle in the header, preference saved per browser.
- **Customer quote builder** — check off items, see a running total in a sticky bar, expand to review/remove items, and copy a clean plain-text quote to your clipboard.
- **PDF invoice generation** — collects customer name/address/contact + invoice date/number, then generates and downloads a branded PDF (logo, business name, contact info, itemized lines, total) via [jsPDF](https://github.com/parallax/jsPDF). Internal cost/labor-rate data is never included on the customer-facing invoice. Invoice numbers auto-increment and persist in `localStorage`.
- **Auto device detection** — the root URL detects mobile vs. desktop user agents and routes accordingly, with manual override via `?view=mobile` / `?view=desktop`.

## Repo structure

```
index.html      Mobile-optimized layout (card-based, touch-friendly, iOS-tuned meta tags)
desktop.html    Desktop/laptop layout (dense tables, multi-column input grids)
manifest.json   Web app manifest (name, icons, theme color) for "Add to Home Screen"
favicon.ico     Browser tab icon
icons/          Favicon and home-screen icon PNGs generated from the business logo
README.md       This file
```

Both files are fully self-contained (inline CSS/JS, no build step) except for the PDF invoice feature, which lazy-loads jsPDF from a CDN (pinned to an exact version with a Subresource Integrity hash) only when you click "Download PDF."

**Why `index.html` and `desktop.html` duplicate all the logic instead of sharing a common file:** this is deliberate, not an oversight. Keeping each layout fully self-contained means either file can be opened directly, AirDropped, or emailed on its own and still work with zero dependencies beyond the CDN-loaded PDF library — there's no build step, bundler, or shared file that has to ship alongside it. The trade-off is that any change to pricing logic, item tables, or the invoice builder has to be made in both files by hand — see "Customizing this for your own operation" below.

## Running this locally

No build step or server required — since everything is plain HTML/CSS/JS, you can just open `index.html` or `desktop.html` directly in a browser. If you want it to behave exactly like the hosted version (e.g. to test the PDF invoice feature, which needs a real origin to fetch jsPDF from its CDN), serve the folder instead of opening the file directly:

```bash
cd junk-removal-fee-schedule-site
python3 -m http.server 8000
# then open http://localhost:8000/ (mobile) or http://localhost:8000/desktop.html (desktop)
```

## Customizing this for your own operation

All of the defaults live in a `DEFAULTS` object near the top of the `<script>` block in each HTML file, and the item tables (`FURNITURE`, `APPLIANCES`, `HAZMAT`, `SPECIALTY`) are plain arrays of `[name, cubicYards, weightLbs, extraFeeKey, note]`. To adjust the model permanently (rather than per-browser via the UI):

1. Edit the relevant array/constant in **both** `index.html` and `desktop.html` (they're independent copies, not shared code, so changes need to be made in both places).
2. Commit and push — GitHub Pages redeploys automatically.

Business contact info and the logo are set in the `BUSINESS` object and `LOGO_DATA_URI` constant near the top of the script in each file.

## Disclaimer

This is a planning and quoting tool, not financial or legal advice. Disposal fees, tax rates, and accepted-item rules at Nevada landfills/transfer stations change over time — verify current rates before relying on them, and talk to an accountant about your actual self-employment and income tax obligations.
