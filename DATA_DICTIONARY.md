# Data Dictionary — `waffle_houses.csv`

**Source file:** `waffle_houses.csv`
**Records:** 2,006 data rows (+ 1 header row)
**Columns:** 14
**Generated:** 2026-06-19

A directory of U.S. Waffle House locations with address, geocoordinates, contact, ownership, and operating-hours information.

---

## Column Definitions

| # | Column | Type | Description | Example |
|---|--------|------|-------------|---------|
| 1 | `Store Code` | String (mostly integer) | Unique store identifier. Numeric for all stores except one special record. | `100`, `1000`, `WH_Museum` |
| 2 | `Business Name` | String | Display name, formatted `Waffle House #<code>`. | `Waffle House #100` |
| 3 | `Address` | String | Street address (uppercase). | `2842 PANOLA RD` |
| 4 | `City` | String | City name (uppercase). | `LITHONIA` |
| 5 | `State` | String (2-char) | USPS 2-letter state code. | `GA` |
| 6 | `Postal Code` | String (5-digit) | 5-digit ZIP code. Keep as string to preserve leading zeros. | `30058` |
| 7 | `Country` | String | Country code. Constant `US` for all rows. | `US` |
| 8 | `Latitude` | Float | Geographic latitude (decimal degrees). | `33.704706` |
| 9 | `Longitude` | Float | Geographic longitude (decimal degrees). | `-84.169849` |
| 10 | `Phone Number` | String | Store phone, format `(XXX) XXX-XXXX`. Some rows hold two numbers separated by `; `. | `(770) 981-1914` |
| 11 | `Website URL` | String (URL) | Location detail page. **All** contain a malformed `///` segment. | `https://locations.wafflehouse.com///lithonia-ga-100` |
| 12 | `Operated By` | String (categorical) | Operating entity: corporate, fully owned subsidiary, or franchise. 16 distinct values. | `WAFFLE HOUSE, INC` |
| 13 | `Online Order Link` | String (URL) | Online ordering page. | `https://order.wafflehouse.com/menu/waffle-house-100` |
| 14 | `Formatted Business Hours` | String | Operating hours, format `Monday - Sunday\| <hours>`. ~99.6% are 24 hours. | `Monday - Sunday\| 24 hours` |

---

## Data Quality Assessment

### Missing / Blank Values
| Column | Blank Count | Notes |
|--------|-------------|-------|
| `Operated By` | 1 | Store `WH_Museum` (Waffle House Museum, Decatur GA) |
| `Online Order Link` | 3 | Stores `442`, `477`, and `WH_Museum` — no online ordering |
| `Formatted Business Hours` | 2 | Store `3442` (Baton Rouge LA) and `WH_Museum` |

All other 11 columns are **100% populated**.

### Duplicates
- **No duplicate `Store Code` values** — the field is a valid primary key.
- **No fully duplicated rows.**

### Type Problems
- **`Store Code`** is *not* uniformly integer: 2,005 rows are numeric, but **`WH_Museum`** is alphanumeric. Cast to string, not int, or filter the museum record before numeric operations.
- **`Latitude`** / **`Longitude`** parse cleanly as floats; all values are within valid ranges (lat 25.10–41.78, lon −112.34 to −75.34) and consistent with the continental U.S.

### Inconsistencies
1. **Special non-store record:** `Store Code = WH_Museum` ("Waffle House Museum") is not a restaurant. It is the source of most blank/type anomalies above. **Recommend excluding it from location analyses.**
2. **Malformed `Website URL`s:** **All 2,006 rows** contain a `///` triple-slash artifact (e.g. `locations.wafflehouse.com///lithonia-ga-100`). Likely a templating bug at export. Normalize before use.
3. **Multi-value `Phone Number`:** **229 rows** contain two numbers joined by `; ` (e.g. `(706) 956-4560; (706) 356-9921`), breaking the standard `(XXX) XXX-XXXX` pattern. Split or take the first number for single-value needs.
4. **`Country` is constant** (`US` for all rows) — zero analytical value; safe to drop.
5. **`Formatted Business Hours` packs two fields into one** delimited by `|` (day range + time range). Split for analysis. Distribution:
   - `Monday - Sunday\| 24 hours` — 1,998
   - `Monday - Sunday\|  7:00am -  9:00pm` — 3
   - `Monday - Sunday\| Closed` — 2
   - `Monday - Sunday\|  7:00am -  2:00pm` — 1
   - blank — 2
   - Note leading/double spaces in the time strings (`  7:00am`) — trim before parsing.
6. **Uppercase text fields:** `Address`, `City` are all-caps; normalize casing if presentation matters.

### `Operated By` Categories (16 distinct)
| Count | Value |
|-------|-------|
| 1,251 | WAFFLE HOUSE, INC |
| 212 | FULLY OWNED SUBSIDIARY: EAST COAST WAFFLES, INC. |
| 169 | FULLY OWNED SUBSIDIARY: MID SOUTH WAFFLES, INC. |
| 143 | FULLY OWNED SUBSIDIARY: MIDWEST WAFFLES, INC. |
| 103 | FULLY OWNED SUBSIDIARY: OZARK WAFFLES, LLC. |
| 37 | FRANCHISE : ROCKY TOP WAFFLES |
| 23 | FRANCHISE : LOOKOUT WAFFLES |
| 19 | FRANCHISE : J. THOMAS & CO., INC. |
| … | (8 more lower-count entities) |

Values mix prefixes (`FULLY OWNED SUBSIDIARY:`, `FRANCHISE :` — note inconsistent spacing around the colon). Consider deriving a clean `operator_type` (Corporate / Subsidiary / Franchise) column.

---

## Recommended Preprocessing
1. Drop the `WH_Museum` record for restaurant-level analysis.
2. Treat `Store Code` and `Postal Code` as strings.
3. Strip the `///` artifact from `Website URL`.
4. Split `Phone Number` on `; ` (handle the 229 multi-number rows).
5. Split `Formatted Business Hours` into day-range and hours; trim whitespace.
6. Derive `operator_type` from `Operated By`.
7. Drop the constant `Country` column.
