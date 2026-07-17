# PrintCalc v2 — Setup Guide (Hinglish)

Ye v2 hai — ab **26 products** (signage + letters + boards + vinyl +
stationery) cover karta hai, aur poora frontend **ek hi `index.html` file**
me hai (HTML+CSS+JS combined, jaisa maanga tha). Backend (Google Sheets +
Apps Script) same tarah se kaam karta hai, bas ab zyada powerful hai —
har product ka apna **formula** hota hai jo Sheet me likha hota hai, code me
nahi.

---

## Naya kya hai (v1 se)

1. **Ek file me sab kuch** — `index.html` ke andar hi `<style>` aur
   `<script>` tags me poora CSS/JS hai. Alag `css/`, `js/` folders ab
   nahi chahiye — bas ye ek file host karo.
2. **Formula Engine** — har product ka pricing formula ab ek text string
   hai (`FormulaMaster` sheet me), jaise:
   ```
   AREA_SQFT*RATE_MATERIAL + PERIMETER_FT*RATE_LED_TYPE + RATE_FABRICATION
   ```
   Ye formula browser me safely calculate hota hai (koi `eval()` nahi,
   khud ka likha hua parser hai — security ke liye zaroori tha). Formula
   change karna ho to bas Sheet ka cell edit karo.
3. **MasterRates sheet** — Depth/Thickness, LED Type, ACP Thickness/Color,
   Acrylic Thickness, Foam Thickness, Vinyl Type/Thickness, Frame Type —
   sab ek hi sheet me, `master_name` column se group kiya hua. Naya
   material master banana ho to bas naya `master_name` value use karke
   rows add karo.
4. **Depth/Thickness ab genuinely cost badalta hai** — Glow Sign Board,
   ACP Panel Board, Acrylic Letter/Box/Channel Letter, Foam Board Name
   Plate, aur Vinyl Car Wrapping me depth dropdown select karte hi price
   turant update hoti hai.
5. **Square Inch pricing** — chhote items (Name Plates) ke liye, sq.ft ke
   sath-sath.
6. **AddOns sheet** (pehle "ExtraCharges" thi) — ab CNC/Laser/Router
   cutting, UV printing, Lamination, Eyelets, Pocket, Rope, Welding,
   Powder Coating, Painting, Installation, Site Visit, Transport, Design
   Charges, Urgent Charges, Packaging — sab cover karta hai.

---

## Step 1 — Google Sheet + Apps Script (same as pehle)

1. Naya Google Sheet banao.
2. Usi Sheet ke andar se **Extensions → Apps Script** kholo (zaroori:
   standalone script.google.com se nahi, Sheet ke andar se hi).
3. `gas/Code.gs` aur `gas/SheetSetup.gs` — dono files paste karo (naye
   files banake, jaise pehle kiya tha).
4. Function dropdown se **`setupAllSheets`** select karke **Run** dabao,
   permissions approve karo.
5. Ab **8 tabs** dikhenge: `Products`, `ProductFields`, `MasterRates`,
   `FormulaMaster`, `Slabs`, `AddOns`, `Settings`, `Quotations` — sab 26
   products ke defaults ke saath already bhare hue.

## Step 2 — Deploy (same as pehle)

1. **Deploy → New deployment → Web app**.
2. **Execute as:** Me. **Who has access:** **Anyone** ⚠️ (sabse zyada
   common galti yahi hoti hai).
3. Deploy karo, URL (`.../exec`) copy karo.
4. `index.html` kholo, `<script>` tag ke bilkul shuru me ye line dhundho:
   ```js
   const API_BASE = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
   ```
   Apna copied URL yahan paste karo.

## Step 3 — Host karo

Pehle jaisa hi — `index.html` ek static file hai, GitHub Pages pe push
karo ya kisi bhi static host pe daalo. Seedha double-click karke mat
kholna (neeche troubleshooting me wajah hai).

---

## 🔴 "Data fetch nahi ho raha" — same 9 checks (ab bhi valid hain)

App ab bhi khud batata hai — header me connection status pill, aur agar
fail ho to orange banner top pe khulega exact reason ke saath. Phir bhi
poori list:

1. **`API_BASE` update nahi kiya** — placeholder waisa hi reh gaya.
2. **Deployment "Anyone" access pe nahi hai** — sabse common. Deploy →
   Manage deployments → pencil icon → "Who has access" = Anyone.
3. **`.../dev` URL use kar liya** — hamesha `.../exec` wala Deploy URL
   use karo, Test deployment ka `/dev` nahi.
4. **Code edit kiya par redeploy nahi kiya** — Deploy → Manage
   deployments → pencil icon → Version: "New version" → Deploy. Sirf save
   karna kaafi nahi.
5. **Script Sheet se bound nahi hai** — Extensions → Apps Script se hi
   khola ho, standalone project se nahi.
6. **`setupAllSheets` chalaya hi nahi** — 8 tabs check karo Sheet me.
7. **`index.html` seedha double-click kiya (file://)** — local server se
   serve karo (`npx serve .` ya `python3 -m http.server 8000`), ya
   GitHub Pages pe deploy karo.
8. **Browser console check karo (F12 → Console)** — exact error milega:
   `Failed to fetch` (access/file:// issue), CORS error (redeploy ya
   access issue), ya JSON ki jagah HTML aana (Apps Script me hi error —
   Executions tab check karo).
9. **Sheet tab names match nahi ho rahe** — `Code.gs` exact ye 8 naam
   expect karta hai: `Products`, `ProductFields`, `MasterRates`,
   `FormulaMaster`, `Slabs`, `AddOns`, `Settings`, `Quotations`.

---

## Formula kaise likhen (naya, v2 me sabse important part)

`FormulaMaster` sheet me har product ke liye ek row hai — do columns
matter karte hain: `product_id` aur `formula`.

Formula ek simple math expression hai jisme ye cheezein use kar sakte ho:
`+ - * / ( )`, numbers, variable names, aur `MIN()` / `MAX()` / `ROUND()`
functions.

### Available variables

**Geometry** (form fields se automatically banti hain):
`WIDTH`, `HEIGHT`, `QTY`, `AREA_SQFT`, `AREA_SQIN`, `AREA_SQM`,
`PERIMETER_FT`, `RUNNING_FT`, `LETTER_COUNT`, `LETTER_HEIGHT_IN`,
`LETTER_WIDTH_IN`, `SLAB_RATE` (Slabs sheet se quantity ke hisaab se).

**Base rates** (`Products` sheet ke us product ki row se, seedhe numbers):
`RATE_MATERIAL`, `RATE_PRINTING`, `RATE_FABRICATION`, `RATE_LABOUR`,
`RATE_INSTALL`, `RATE_DESIGN`, `RATE_TRANSPORT`, `RATE_SITE_VISIT`,
`RATE_PACKAGING`, `RATE_URGENT`. Formula khud decide karta hai ki ye rate
area se multiply hogi ya flat charge ki tarah add hogi.

**Master-driven rates**: `ProductFields` me jo bhi `select` field ka
`master_name` set hai, uska selected option choose hote hi
`RATE_<FIELD_ID>` variable ban jaata hai. Jaise: field `depth_mm` +
master `SIGNAGE_DEPTH` → `RATE_DEPTH_MM`. Field `led_type` + master
`LED_TYPE` → `RATE_LED_TYPE`.

### Example (jo seed data me pehle se hai)

```
Glow Sign Board:
AREA_SQFT*RATE_MATERIAL + AREA_SQFT*RATE_PRINTING +
AREA_SQFT*RATE_DEPTH_MM + PERIMETER_FT*RATE_LED_TYPE + RATE_FABRICATION

Acrylic Box Letter (letter-count based):
LETTER_COUNT*LETTER_HEIGHT_IN*RATE_MATERIAL +
LETTER_COUNT*RATE_DEPTH_MM + LETTER_COUNT*RATE_FABRICATION

Neon Sign (MIN/MAX ka example — kabhi bhi fabrication floor se neeche
nahi jaayega):
MAX(RUNNING_FT*RATE_MATERIAL, RATE_FABRICATION) + RUNNING_FT*RATE_LED_TYPE

Visiting Card (slab pricing):
QTY*SLAB_RATE
```

Agar formula galat likha (typo, missing bracket), app crash nahi hoga —
"Base Product Cost" 0 dikhega aur breakdown panel me chhota sa warning
message aayega, taaki turant pata chal jaaye kuch galat likha hai.

Formula ke baad ka pipeline (wastage, add-ons, margin, discount, minimum
billing, GST, rounding) **har product ke liye same** hai — wo `Products`
sheet ke columns se control hota hai, formula me likhne ki zaroorat nahi.

---

## Naya (27th) product add karna hai?

1. `Products` me naya row (rates + wastage/margin/discount/min-billing).
2. `ProductFields` me uske form fields — depth/LED/material jaisi koi
   dropdown chahiye to `master_name` column me existing master ka naam
   daalo (ya naya master_name banao `MasterRates` me).
3. `FormulaMaster` me uska formula likho.
4. Zaroorat ho to `AddOns` me optional services add karo.
5. App reload karo — naya product dropdown me turant dikhega, apna form
   aur apni live pricing ke saath.

Koi bhi step Code.gs ya index.html chhoone ki zaroorat nahi padegi.
