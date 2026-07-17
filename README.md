# PrintCalc v2 — Signage & Printing Costing Engine

> Hinglish me detailed setup + troubleshooting ke liye **`README-hinglish.md`** dekho.

A single-file (`index.html` — HTML+CSS+JS combined) commercial signage &
printing calculator covering **26 products**: Glow Sign Board, Sign Board,
ACP Panel Board, Foam Board Name Plate, Acrylic Name Plate, Acrylic Letter
Board, Acrylic Box Letter, Acrylic LED Letter, Acrylic + Channel Letter,
SS/MS/Brass Letter, Vinyl Printing, Vinyl Car Wrapping, One Way Vision,
Sun Board, Foam Board (3/5/10mm), Flex Printing, Flex with Frame, LED Sign
Board, Neon Sign, Backlit/Frontlit Board, Visiting Card, Brochure, Letterhead.

**Nothing about any product's math is in the code.** Every rate, material,
depth/thickness option, LED type, formula and add-on lives in Google
Sheets. Adding a 27th product needs zero deploys.

## What's different from v1

- **One file, not five.** `index.html` now contains all CSS and JS inline
  (you asked for combined code) — still zero build step, still just a
  static file to host.
- **A real formula engine.** Each product's cost formula is a plain-text
  expression stored in the `FormulaMaster` sheet — e.g.
  `AREA_SQFT*RATE_MATERIAL + PERIMETER_FT*RATE_LED_TYPE + RATE_FABRICATION`
  — parsed and evaluated safely in the browser (a hand-written parser, not
  `eval()`). Change pricing logic by editing a sheet cell.
- **MasterRates** — one shared table for every material/thickness/depth/
  LED/ACP/acrylic/vinyl/frame master. Depth (10/20/30/40/50/75/100mm) now
  genuinely drives cost for Glow Sign Board, ACP Panel Board, Acrylic
  Letter/Box/Channel Letter, Foam Board Name Plate and Vinyl Car Wrapping.
- **Square-inch pricing** for small items (name plates) alongside sq.ft,
  sq.m, running foot, per-letter, per-piece, fixed and quantity-slab.
- **AddOns** (renamed from "extras") now cover the full list: CNC/laser/
  router cutting, UV printing, lamination, eyelets, pocket, rope, welding,
  powder coating, painting, installation, site visit, transport, design
  charges, urgent charges, packaging — add a row to offer a new one.

## Files

```
index.html         Everything: markup, styles, and all JS modules
                    (API layer, formula engine, calculation pipeline,
                    form builder, rendering, app controller)
gas/Code.gs         REST API (doGet/doPost) reading the 8 sheets
gas/SheetSetup.gs   One-click script that builds + seeds all 8 sheets
                    with the 26-product starter catalogue
```

## Setup (same 4 steps as before)

1. **Create a Google Sheet**, open **Extensions → Apps Script** from
   inside it (must be bound, not a standalone script).
2. Paste `gas/Code.gs` and `gas/SheetSetup.gs` in as two files.
3. Run **`setupAllSheets`** once. Approve permissions. 8 tabs appear:
   `Products`, `ProductFields`, `MasterRates`, `FormulaMaster`, `Slabs`,
   `AddOns`, `Settings`, `Quotations`.
4. **Deploy → New deployment → Web app → Execute as Me → Access: Anyone.**
   Copy the `/exec` URL into `index.html`'s `API_BASE` constant (near the
   top of the `<script>` block).
5. Host `index.html` anywhere static (GitHub Pages, Render static site,
   or a local server for testing — see the Hinglish README for why
   double-clicking the file directly can fail).

## The formula language

Each product's `FormulaMaster.formula` is a plain arithmetic expression.
Supported: `+ - * / ( )`, numbers, variable names, and `MIN()` / `MAX()` /
`ROUND()`. Unknown variables evaluate to `0` instead of erroring, so a
formula can safely reference a field a particular product doesn't have.

**Geometry variables** (computed automatically from whatever fields the
product has): `WIDTH`, `HEIGHT`, `QTY`, `AREA_SQFT`, `AREA_SQIN`,
`AREA_SQM`, `PERIMETER_FT`, `RUNNING_FT`, `LETTER_COUNT`,
`LETTER_HEIGHT_IN`, `LETTER_WIDTH_IN`, `SLAB_RATE` (looked up from the
`Slabs` sheet by quantity).

**Base rate variables** (plain numbers from that product's row in
`Products`): `RATE_MATERIAL`, `RATE_PRINTING`, `RATE_FABRICATION`,
`RATE_LABOUR`, `RATE_INSTALL`, `RATE_DESIGN`, `RATE_TRANSPORT`,
`RATE_SITE_VISIT`, `RATE_PACKAGING`, `RATE_URGENT`. Whether a formula
multiplies one of these by area or just adds it as a flat charge is
entirely up to how you write the formula.

**Master-driven variables**: any `select` field in `ProductFields` with a
`master_name` set exposes `RATE_<FIELD_ID>` once the user picks an option
— e.g. a field `depth_mm` backed by master `SIGNAGE_DEPTH` gives you
`RATE_DEPTH_MM`; a field `led_type` backed by `LED_TYPE` gives you
`RATE_LED_TYPE`.

**Examples actually used in the seed data:**
```
Glow Sign Board:
  AREA_SQFT*RATE_MATERIAL + AREA_SQFT*RATE_PRINTING +
  AREA_SQFT*RATE_DEPTH_MM + PERIMETER_FT*RATE_LED_TYPE + RATE_FABRICATION

Acrylic Box Letter:
  LETTER_COUNT*LETTER_HEIGHT_IN*RATE_MATERIAL +
  LETTER_COUNT*RATE_DEPTH_MM + LETTER_COUNT*RATE_FABRICATION

Neon Sign (shows MIN/MAX — never bills below the fabrication floor):
  MAX(RUNNING_FT*RATE_MATERIAL, RATE_FABRICATION) + RUNNING_FT*RATE_LED_TYPE

Visiting Card (slab pricing):
  QTY*SLAB_RATE
```

## What happens after the formula (same for every product)

```
base_cost = formula result
 → + wastage %
 → + selected add-ons (fixed / % of base / per-unit × qty)
 → + profit margin %
 → − discount %
 → enforce minimum billing floor
 → + GST %
 → round per rounding rule
 = Final Selling Price   (÷ unit count = Cost per Unit)
```

## Adding a 27th product — no code changes

1. `Products`: add a row (rates, wastage/margin/discount/min-billing).
2. `ProductFields`: add its input fields — set `master_name` on any
   select field that should pull from a shared master (depth, LED type,
   etc.), or leave it blank and fill `options` with a pipe-separated list.
3. `FormulaMaster`: add its formula string using the variables above.
4. If it needs a brand-new material/thickness master, add rows to
   `MasterRates` with a new `master_name` — no other sheet changes needed.
5. `AddOns`: add optional services (cutting, lamination, installation…).
6. Reload — it appears in the dropdown with its own form and live pricing.

## Notes

- **PDF export** uses the browser print dialog with an embedded print
  stylesheet — choose "Save as PDF". No extra dependencies, works in
  WebViews.
- **CORS**: the API posts with `Content-Type: text/plain` — this avoids
  Apps Script's lack of an OPTIONS/preflight handler. Don't change it.
- The seeded rates across all 26 products are realistic-looking
  placeholders, not your actual pricing — tune `Products`, `MasterRates`,
  `Slabs` and `AddOns` before quoting real customers.
