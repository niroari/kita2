# PROJECT.md — Class ז'2 Info Website

## What This Is
A single-page website for the parents and students of class ז'2 at Ben Gurion Middle School, Herzliya.
Homeroom teacher: Nir Oz-Ari.
Static HTML file — no server, no framework. Opened locally as a `file://` URL.

## Files
- `/Users/nirozari/Class Info Website/index.html` — the entire site
- `/Users/nirozari/Class Info Website/עריכת אתר ז2.xlsx` — Excel file for editing all content

## Sections
1. **Hero landing page** — title, subtitle, 4 nav cards, announcements box
2. **מערכת שעות** — weekly schedule (Sun–Thu), including break rows
3. **צוות המורים** — teacher cards with colored avatars and WhatsApp links
4. **מבחנים ואירועים** — events/exams timeline, grouped by month, color-coded badges
5. **מקומות ישיבה** — animated classroom seating chart

## Google Sheets — Single Sheet for Everything
- **Sheet ID:** `1C3qCuU_SwJvczmGszW52v71aSC6TWtN7xQT1KsOPBlM`
- **Tabs:** `הודעות` | `ישיבה` | `אירועים` | `מערכת שעות` | `צוות מורים`
- All fetched via JSONP (`<script>` tag injection) using Google's `gviz/tq` endpoint
- CORS blocks `fetch()` on `file://` URLs — JSONP is the workaround
- All URLs include `&cb=${Date.now()}` to bust browser cache on every load
- Google's server-side cache can add a few minutes delay after sheet edits — normal behavior

## JSONP Callbacks
| Section | Callback | headers param |
|---|---|---|
| הודעות | `_sheetsCallback` | `headers=1` |
| ישיבה | `_seatingCallback` | `headers=0` |
| אירועים | `_eventsCallback` | `headers=1` |
| מערכת שעות | `_scheduleCallback` | `headers=1` |

## Excel Editing File — Tab Structure
**ישיבה** — 8 columns, 5 data rows, NO header row
- Pairs of columns = one desk: A+B = desk 1 (right), C+D = desk 2, E+F = desk 3, G+H = desk 4 (left)
- Each student gets their own cell (no `/` separator)
- Lidor (תומכת למידה) is in row 1, column A. Column B of that row is empty.
- DO NOT add a header row — the seating JS uses `headers=0` (all rows are desk rows)

**הודעות** — 4 columns: תאריך | כותרת | הודעה | חשוב
- Column D dropdown: TRUE (red/important) or FALSE (normal)
- Conditional formatting: row turns red when D = TRUE
- 50 data rows

**אירועים** — 5 columns: תאריך התחלה | כותרת | שעה | קטגוריה | תאריך סיום
- All holidays for 2025–2026 pre-filled
- Date validation on columns A and E
- Category dropdown with color-coding
- Multi-day events: fill both start and end date → site shows "22–24.9" format

**מערכת שעות** — 9 columns (period | time | Sun–Thu | סוג שורה)
- Break rows: type = "הפסקה" in last column

**צוות מורים** — 5 columns: שם | מקצוע | תפקיד | טלפון | מייל

## Seating Chart — How It Works
- JS reads the sheet with `headers=0` (no header row in the sheet)
- Processes cells in PAIRS: cells 0+1 = desk 1, cells 2+3 = desk 2, etc.
- `v1 === 'לידור'` → renders Lidor unit with "תומכת למידה" label (golden styling)
- Both cells empty → empty desk (dashed, faded)
- One cell has a name → single seat
- Both cells have names → double seat with 22px gap (no divider line)
- Classroom door is on the RIGHT (first DOM element in RTL)
- Animated on scroll via IntersectionObserver

## Events — How It Works
- Past events hidden by default (`.event-row.past { display: none }`)
- "הצג אירועים שעברו" toggle button reveals them (class `show-past` on container)
- Month groups where ALL events are past are also hidden entirely
- Sorted by date regardless of sheet order
- `parseEventDate(cell)` handles: `Date(Y,M,D)` (native Sheets), `DD/MM/YYYY`, `D.M.YYYY`
  - Tries `cell.v` first, falls back to `cell.f` (formatted display value)
- "Past" for multi-day events uses the END date, not start date

## Design
- Hebrew RTL (`dir="rtl"`) throughout
- Dark theme: deep blue-purple gradient (`#07060f → #131129 → #0e1628`) with dot-grid
- Glass morphism panels (`backdrop-filter: blur`)
- Alef font from Google Fonts
- SVG icons only — no emoji (except one 🎉 in the events sheet, added by Nir)
- Staggered CSS animations using `--d` custom property

## Known Issues / Future Work
- Teachers list may still be incomplete
- Site not yet hosted online — opened as a local file
- WhatsApp links format: `https://wa.me/972XXXXXXXXX` (drop leading 0, add 972)
