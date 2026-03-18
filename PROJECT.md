# PROJECT.md — Class ז'2 Info Website

## What This Is
A single-page website for the parents and students of class ז'2 at Ben Gurion Middle School, Herzliya.
Homeroom teacher: Nir Oz-Ari.
Static HTML file — no server, no framework. Data comes from Firebase Firestore.

## Hosting
- **GitHub:** https://github.com/niroari/kita2
- **Live URL (Vercel):** https://kita2.vercel.app (auto-deploys on every push to main)
- To update the live site: edit files locally → `git add / commit / push`

## Files
- `index.html` — the entire site
- `admin.html` — password-protected admin panel for editing all content
- `migrate.html` — one-time migration tool (gitignored, local only — run via `python3 -m http.server 8080`)

## Sections (page order)
1. **Hero landing page** — school logo (white), class name, then: 5 compact pill nav buttons, then announcements box (הודעות המחנך), then scroll hint
2. **מערכת שעות** — weekly schedule (Sun–Thu), including break rows
3. **מבחנים ואירועים** — events, one month at a time with prev/next navigation, category filter
4. **מקומות ישיבה** — animated classroom seating chart
5. **צוות המורים** — teacher cards collapsed by default, click to expand contact info
6. **קישורים חשובים** — 9 link cards (4-column grid); last card is full-width featured NotebookLM card

## Admin Panel
- **URL:** `kita2.vercel.app/admin.html`
- Login: email + password (Firebase Auth — Google OAuth not used due to domain authorization complexity)
- Changes go live instantly (no deploy needed)
- Mobile-friendly: scrollable tabs, forms stack vertically, tables scroll horizontally

### Per-section behaviour
| Section | How editing works |
|---|---|
| הודעות | Add form at top; existing list below with Delete button |
| אירועים | Add form at top with date picker + category dropdown; list below with Delete |
| מורים | Add form at top; table below scrolls horizontally; Delete button per row |
| מערכת שעות | Horizontally scrollable inline table — click any cell to edit, auto-saves on blur |
| מקומות ישיבה | Drag-and-drop grid — drag card to swap; click `+` on empty slot to add; click `×` to remove |

### Seating grid details
- לוח bar at bottom, 🚪 door indicator to its right (matches physical classroom)
- Row labels inverted: stored `order=5` → displayed שורה 1 (front), stored `order=1` → displayed שורה 5 (back)
- Local `seatingData` object keeps state; `renderSeating()` rebuilds DOM after every change; Firestore updated async

## Backend — Firebase Firestore
- **Firebase project:** `kita-3017b`
- **SDK:** v10.12.0 compat, loaded from CDN in `<head>`
- **Security rules:** public read (`allow read: if true`), authenticated write only (`allow write: if request.auth != null`)
- **To edit data:** use the admin panel at `/admin.html` — no need to touch Firebase Console directly

### Collections
| Collection | One document per | Key fields |
|---|---|---|
| `announcements` | announcement | `order`, `date`, `title`, `body`, `important` |
| `events` | event | `date` (Timestamp), `title`, `time`, `category`, `endDate` (Timestamp, optional) |
| `schedule` | lesson row | `order`, `period`, `time`, `sun`–`fri`, `type` |
| `seating` | desk row | `order`, `desk1_right/left` … `desk4_right/left` |
| `teachers` | teacher | `order`, `name`, `subject`, `role`, `phone`, `email` |

### Important notes
- `order` field controls sort order — always query with `.orderBy('order')`
- Event dates are stored as Firestore Timestamps → convert with `.toDate()` in JS
- Teachers must keep their `order` field to preserve row order (Nir + רכזת first)

### Re-migrating from Google Sheets
If you ever need to re-sync all data from Google Sheets:
1. Temporarily set Firestore rules to `allow write: if true` and Publish
2. Run `python3 -m http.server 8080` in the project folder
3. Open `http://localhost:8080/migrate.html` and click the button
4. Set rules back to `allow write: if false` and Publish

## Seating Chart — How It Works
- Firestore `seating` collection: 5 documents (one per desk row), ordered by `order`
- Each document: `desk1_right`, `desk1_left`, `desk2_right`, `desk2_left`, `desk3_right`, `desk3_left`, `desk4_right`, `desk4_left`
- `v1 === 'לידור'` → renders Lidor unit with "תומכת למידה" label (golden styling)
- Both cells empty → empty desk (dashed, faded)
- One cell has a name → single seat
- Both cells have names → double seat with 22px gap (no divider line)
- Classroom door is on the RIGHT (first DOM element in RTL)
- Animated on scroll via IntersectionObserver

## Events — How It Works
- Dates stored as Firestore Timestamps; `.toDate()` converts to JS Date
- **Monthly view:** shows one month at a time with `‹ month ›` navigator
  - Globals: `_monthGroups[]`, `_monthIdx`, `_showPast`
  - `renderCurrentMonth()` rebuilds the list on every navigation/filter/toggle
  - `changeMonth(dir)` moves index by ±1
  - Auto-starts on first month with upcoming events; auto-shows past if navigating to an all-past month
- **Arrow direction (RTL convention):** ‹ (left) = forward/next month; › (right) = back/previous month
  - `month-prev` button (‹) calls `changeMonth(+1)`; disabled at last month
  - `month-next` button (›) calls `changeMonth(-1)`; disabled at first month
- Past events hidden by default; "הצג אירועים שעברו" toggle sets `_showPast` and re-renders
- Sorted by `a.date - b.date` in JS
- "Past" for multi-day events uses the END date, not start date
- **Category filter:** `_activeFilter` global; clicking legend item re-renders current month with filter applied
  - Each event-row has `data-cat`; legend items have `data-cats` (comma-separated for חג,חופש)

## Teachers — How It Works
- Loaded from Firestore `teachers` collection, ordered by `order`
- Fields: `name` | `subject` | `role` | `phone` | `email`
- Cards show name + subject (+ role badge) by default — collapsed
- **All cards are clickable** with chevron; click expands contact panel:
  - Has phone → WhatsApp link
  - Has email → mailto link
  - Neither → "פנייה דרך המשוב" link → `https://web.mashov.info/parents/main/home`
- `toggleTeacher(card)` — closes other open cards, opens clicked one
- WhatsApp number built as: `'972' + phone.replace(/\D/g,'').replace(/^0/,'')`

## Design
- Hebrew RTL (`dir="rtl"`) throughout
- Dark theme: deep blue-purple gradient (`#07060f → #131129 → #0e1628`) with dot-grid
- Glass morphism panels (`backdrop-filter: blur`)
- Alef font from Google Fonts
- SVG icons only — no emoji (except one 🎉 in the events sheet, added by Nir)
- Staggered CSS animations using `--d` custom property

## Quick Links — URLs
| שם | URL | Notes |
|---|---|---|
| גוגל קלאסרום | https://classroom.google.com | |
| משוב תלמידים | https://web.mashov.info/students/main/home | |
| משוב הורים | https://web.mashov.info/parents/main/home | |
| אופק הילקוט הדיגיטלי | https://students.myofek.cet.ac.il/ | |
| גלים פרו | https://pro.galim.org.il | |
| מודל | https://moodlemoe.lms.education.gov.il/ | |
| תקנון בית הספר | https://bengurion-herzliya.mashov.info/wp-content/uploads/sites/225/2024/07/%D7%AA%D7%A7%D7%A0%D7%95%D7%9F-%D7%91%D7%99%D7%AA-%D7%94%D7%A1%D7%A4%D7%A8.pdf-6.pdf | Direct PDF link |
| אתר בית הספר | https://bengurion-herzliya.mashov.info/ | favicon made white |
| שאל את התקנון | https://notebooklm.google.com/notebook/5e283b47-066d-4c88-abdb-16f49da9c3c6 | NotebookLM; featured full-width card |

- Grid is 4 columns (`repeat(4, 1fr)`, `max-width: 740px`); 8 regular cards fill 2 rows, featured card spans row 3
- School/תקנון favicons and NotebookLM logo all use `filter: brightness(0) invert(1)` (white on dark background)
- Featured card style: `.quick-link-featured` — purple gradient, `grid-column: 1/-1`

## Mobile — Key Decisions
- `html, body { overflow-x: hidden }` + `main { overflow: hidden }` — section glow pseudo-elements use negative `right/left` values (e.g. `right: -160px`) which extend past the viewport, causing the body to be wider than the screen. In RTL Chrome on Android, this made the browser show the empty left side, shifting all content right. Clipping at `main` fixes it.
- Schedule table: on mobile, `overflow: visible` overrides the desktop `overflow: hidden` so that `position: sticky` works on the first two columns (שיעור + שעות). They stick to the right when scrolling the table sideways.
- `#month-nav` has `direction: ltr` so arrow positions are physical (‹ = left, › = right), not mirrored by RTL.

## Known Issues / Future Work
- Teachers list may still be incomplete

## Future — Grade-Wide Expansion (not started)
Goal: turn this into a site for the entire grade (שכבה ז׳), with 6 classes.
Note: original plan was Google Sheets-based — needs rethinking now that backend is Firebase.
Architecture would likely be: one Firebase project per class, or one project with sub-collections per class.

**Open questions to resolve when starting:**
1. One Firebase project for all classes, or one per class?
2. Who manages each class's data — each teacher, or just Nir?
3. New Vercel URL for the grade site?

## Workflow — Publishing Updates

### Updating site content (data)
- Go to `kita2.vercel.app/admin.html`, log in, edit
- Changes are live immediately (no deploy needed)

### Updating site design / code
1. Edit `index.html` locally
2. `git add / commit / push`
3. Vercel auto-deploys within ~30 seconds
