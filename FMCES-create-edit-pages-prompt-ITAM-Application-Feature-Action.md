# Task: redesign 4 create pages and their 4 edit pages in FMCES — ITAM, Application, Feature, Action

You are working in the codebase of **FMCES** (Financial Markets Central Entitlement System), an internal corporate web app shown under "Markets Operations". It uses a dark navy theme with a teal accent. Your job is to **rebuild the look and layout of four "create" pages and their four "edit" pages** so they match the design described in this prompt *exactly*, without breaking anything they do today. Each edit page looks exactly like its create page, just filled with the record's data (section 7).

This prompt is self-contained. You get no images or design files. Everything you need is here: the rules, the colours, every size, the page layouts, the behaviour, a complete stylesheet (Appendix A) and full markup skeletons (Appendix B). **Follow the numbers exactly.** Do not "improve" sizes, colours or spacing by eye. The previous version of these pages looked oversized and badly spaced; the values below fix that.

---

## 0. Ground rules (read before you write any code)

**Change only the presentation of these eight pages:**
- Add ITAM and Edit ITAM
- Add Application and Edit Application
- Add Feature (becomes "Add features") and Edit Feature
- Add Action (becomes "Add actions") and Edit Action

**Keep exactly as they are today:**
- Routes and URLs, and how the user reaches each page.
- Every API call, endpoint, request payload shape and field name sent to the backend.
- The form state library or approach the page uses today (React state, Formik, react-hook-form, whatever exists).
- Validation rules: which fields are required, max lengths, formats, server-side errors. Restyle how errors look, but do not invent new rules or drop existing ones.
- What Save and Submit do: success toasts or messages, and where the user lands after a submit.
- Permission checks, loading states and error handling.
- Any `id`, `name` or `data-testid` that tests may use. Keep them on the equivalent new elements.

**Do not touch:**
- The app header, the left sidebar (dock), list pages, the detail drawer, or any other page.

**Do not add backend work:**
- Do not add API endpoints or change backend code.
- Anything marked **[IF DATA EXISTS]** must only be built when the page already has that data, or an existing API already returns it. Otherwise leave that part out completely. Do not show placeholders or "coming soon" text.

**Get rid of the old look on these eight pages:**
- Glass and blurred panels.
- Blue left bars on section titles.
- ALL-CAPS section titles.
- Floating labels.
- Placeholder text used as labels.
- Double asterisks (`**`).
- Mixed field heights.
- Full-width buttons.
- Big empty panels.

**Sizing rules:**
- Do not use `zoom`, `transform: scale()`, `vw`-based font sizes, or rem values tied to a changed root font size.
- Every size comes from the CSS variables in Appendix A. They are px values that switch by screen size with media queries.

**Styling and structure:**
- Copy Appendix A into a new stylesheet (for example `src/styles/create-forms.css`) **verbatim**. Load it on these pages, or globally — it is fully scoped under the `.cf` class.
- If the app uses CSS modules, styled-components or Tailwind, still put this file in as a plain global stylesheet. Do not translate it; translating loses values.
- Build the shared pieces once as reusable components and use them on all four pages:
  - FormPage
  - PageHeader
  - FormSection
  - Field
  - TextInput
  - SelectField
  - SearchSelect
  - DateField
  - FileDrop
  - ProgressRail
  - ActionBar
  - BulkGrid

  The other create pages (Data Profile, Data Policy, Entitlement, Account) will reuse them later, so keep them generic.

**Fonts:**
- Text uses **Manrope**: weights 400, 500, 600, 700 and 800.
- IDs and codes use **JetBrains Mono**: weights 400, 500 and 600.
- If the app does not load them yet, add this once in the document head:
  `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

### Step 0 — inspect, then write a short mapping

Before you change anything, open the eight page components (four create, four edit) and write down a mapping. Keep it as a comment in your first message or in the PR description:

1. Where each page lives, and which component renders it.
2. Every field on each page, with:
   - its state key or field name,
   - whether it is required today,
   - any max length or format rule,
   - the control type (text, select, date, file).
3. The Save, Submit, Reset and Cancel handlers each page already has.
4. Where the ITAM options come from on Add Application.
5. What the second dropdown ("Select ITAM ID first") on Add Application is, and where its options come from.
6. Whether the selected ITAM object already includes ITAM name, SNOW group and support DL. This decides the **[IF DATA EXISTS]** auto-fill strip.
7. How the OLA upload works:
   - the handler,
   - accepted file types,
   - the max size,
   - whether upload progress is available.
8. How Add Feature and Add Action build their request today. The current pages let the user add several items with "+ Add More …", each with Application, Name and Description. Note the exact array or payload shape.
9. Whether existing feature and action names for an application are already loaded or fetchable with an existing call. This decides the **[IF DATA EXISTS]** duplicate check and the "Already in this application" chips.
10. For each edit page:
    - its route and how it loads the record (the existing fetch call);
    - which fields it disables or does not let the user change;
    - its update handler and payload;
    - whether it has a Save (draft) as well as Submit;
    - whether the record has a status, a version number and "last modified" data;
    - whether editing an approved record creates a new version.

Then build the shared components, then the four create pages, then the four edit pages, then run the acceptance checks in section 10.

---

## 1. Design tokens

All values live as CSS variables on `.cf` (Appendix A). This is a summary, so you know what they mean.

**Colours**

| Token | Value | Use |
|---|---|---|
| `--c-bg` | `#0F1A22` | Page background |
| `--c-field` | `#111E27` | Inside inputs, the grid, drop zones |
| `--c-card` | `#152733` | Section cards, rail cards |
| `--c-card-2` | `#1B3140` | Raised: grid header, hover, number tiles |
| `--c-border` | `#23394A` | Card and input borders |
| `--c-border-hover` | `#2E4A5D` | Input border on hover |
| `--c-hair` | `#1E3240` | Dividers inside cards and the grid |
| `--c-btn-border` | `#2B4556` | Outline buttons, chips, dashed boxes |
| `--c-text` | `#E6EEF2` | Main text, input values |
| `--c-text-2` | `#A7B8C4` | Labels, subtitles |
| `--c-muted` | `#86A0AE` | Help text, meta, icons |
| `--c-placeholder` | `#7890A0` | Placeholders |
| `--c-accent` | `#2DD4BF` | Primary button, focus border, done states |
| `--c-accent-strong` | `#5EEAD4` | Hover on primary; teal text |
| `--c-on-accent` | `#062A26` | Text on teal |
| `--c-link` | `#7DBBFF` | Links ("browse") |
| `--c-err` | `#F4A3A3` | Required star, errors (tint `rgba(244,163,163,.10)`, line `rgba(244,163,163,.55)`) |
| `--c-warn` | `#F6C177` | Draft badge, "now" step in the lifecycle |
| `--c-info` | `#A9D1FF` | Info alerts, "New" badge |

- Focus ring: 3px `rgba(45,212,191,.16)` around a teal border.
- Keyboard focus outline: 2px `#5EEAD4`, offset 2px.
- Easing: `cubic-bezier(.22,1,.36,1)`. Toggle spring: `cubic-bezier(.34,1.36,.64,1)`.

**Density.** Picked automatically by screen size. The tiers are media queries in Appendix A — do not add your own.

| Token | Compact (default) | Comfortable: width ≥ 1440 **and** height ≥ 820 | Large: width ≥ 1880 **and** height ≥ 980 |
|---|---|---|---|
| Body / input text `--fs-md` | 12.5px | 13px | 13.5px |
| Label `--fs-sm` | 11.5px | 12px | 12px |
| Help / error `--fs-xs` | 10.5px | 11px | 11px |
| Mono values `--fs-mono` | 12px | 12.5px | 13px |
| Page title `--fs-h1` | 19px / 800 | 22px / 800 | 24px / 800 |
| Section title `--fs-h2` | 13.5px / 800 | 14.5px / 800 | 15px / 800 |
| Input, select and button height | 30px | 34px | 36px |
| Small button | 26px | 28px | 28px |
| Chip height | 22px | 24px | 24px |
| Grid row height | 38px | 42px | 44px |
| Action bar height | 52px | 60px | 64px |
| Field radius / card radius | 7 / 10px | 8 / 12px | 8 / 12px |
| Column gap / row gap | 12 / 12px | 16 / 14px | 20 / 16px |
| Label → input gap | 5px | 6px | 6px |
| Section padding | 14px 16px 16px | 18px 20px 20px | 20px 24px 24px |
| Gap between sections | 12px | 16px | 20px |
| Page padding left / right | 16 / 24px | 16 / 32px | 16 / 40px |
| Progress rail width | 256px | 288px | 320px |
| Max page width (left aligned) | 1320px | 1360px | 1480px |
| Centred card width (ITAM) | 720px | 780px | 820px |

**Icons**
- 24×24 viewBox, stroke icons, `stroke-width` 1.75, or 2.25 where the skeleton says `bold`. Round caps and joins, `fill: none`, colour `currentColor`.
- If the app already uses **lucide** icons, use these lucide names:
  - `ChevronLeft`, `ChevronDown`, `X`, `Search`
  - `Calendar`, `Upload`, `FileText`, `Eye`, `Trash2`
  - `Check`, `CheckCircle2`, `AlertTriangle`, `Info`
  - `RotateCcw`, `Save`, `Send`, `Plus`, `Clipboard`
  - `User`, `Server`, `AppWindow`, `Lock`
- Otherwise use the inline SVG paths in Appendix C.
- In the skeletons, `[icon:name size]` marks where an icon goes and at what px size.

---

## 2. The page shell (all eight pages)

**Where the page mounts**
- The page renders inside the app's existing content area: right of the dock sidebar, below the header.
- The root element is `<form class="cf" novalidate>`. If the page already renders a `<form>` with an `onSubmit`, put the `cf` class on that form instead of adding a second one. It must fill the content area's height so the action bar can stick to the bottom:
  - Make the content area the scroll container (`overflow: auto`, with a fixed height from the layout).
  - `.cf` has `min-height: 100%`.
  - If the content area has no fixed height, give `.cf` `min-height: calc(100vh - <real header height>px)`.
- `.cf-bar` uses `position: sticky; bottom: 0`. It **only sticks if no element between the bar and the scroll container has `overflow: hidden` or `overflow: auto`**. Check this.
- Test: open Add Application at 1366×768 and scroll. The bar must stay pinned to the bottom of the window the whole time.

**Inside the root**
- `.cf-inner` holds the page content. It is left aligned, with a max width from the tokens; Add ITAM uses `.cf-inner cf-inner--center`.
- `.cf-bar` is the **last child of the form**. It spans the full width, and its contents (`.cf-bar-inner`) line up with the page content.

**Page header** (`.cf-head`)
- Left: a back button, 30/34/36px square with 9px radius and an outline. It goes to the list page, the same as today's back button.
- Then three stacked lines:
  1. Breadcrumb in mono, 10.5–11px, muted, for example `FMCES / App Management / New`. Every part except the last is a link.
  2. Page title, sentence case.
  3. One-line subtitle.
- Right: a status badge — `New` (blue) until the first successful Save, then `Draft` (amber). Bulk pages show no badge.

**Section card** (`.cf-sec`)
- `#152733` with a 1px `#23394A` border and 10 / 12 / 12px radius.
- Header: a 24px number tile, a title (sentence case, 800 weight) and a one-line description (muted).
- The number tile has three states:
  - number (default),
  - teal with a check icon when the section is complete,
  - red "!" when the section has a visible error.
- The body is a 12-column grid (`.cf-grid`). Fields span 4, 6 or 12 columns (`.cf-span-4/6/12`). When the card is narrower than 600px, every field spans 12. This is done with a container query; do not change it.

**Field** (`.cf-field`)
- The label is always above the control. `.cf-label` is 11.5 / 12 / 12px at weight 700, colour `#A7B8C4`, linked to the control with `for` and `id`.
- **Required:** one red `*` straight after the label text (`<span class="cf-req" aria-hidden="true">*</span>`), plus `required` / `aria-required="true"` on the control.
- **Optional:** add `<span class="cf-opt">Optional</span>` after the label text.
- Under the control there is **at most one line**:
  - help text (`.cf-help`, muted), or
  - an error (`.cf-error`, red, with the alert icon). The error *replaces* the help while it is showing.
- Link the line to the control with `aria-describedby`. Invalid controls get `.is-invalid` and `aria-invalid="true"`.
- **Counter** (`.cf-count`, right side of the label row, mono 10.5px):
  - Only on fields that already have a max length. It shows `used / max`.
  - It gets `.is-over` (red) when the value is over the limit.
  - Never invent a max length.
- **Mono:** IDs, codes, distribution lists, owners (bank IDs), ITAM IDs and system names use JetBrains Mono (`.cf-input--mono`, or `.cf-mono` on an inner input). These are long values; mono keeps them readable.
- **Read-only values** (copied from another record) use a dashed border and a transparent background (`readonly`).
- **Disabled** controls show at 50% opacity.

**Controls** (all exactly one height: 30 / 34 / 36px)
- **Text input:** `.cf-input`.
- **Textarea:** `textarea.cf-input`. Minimum height ≈ 2.2 × the input height, resizes vertically, line-height 1.5.
- **Select:** native `<select class="cf-input">` inside `<span class="cf-select">` with a chevron icon on top. While no value is chosen, add `.is-empty` so it shows in the placeholder colour.
- **Search select (combobox):** `.cf-combo > .cf-control`, with a leading icon, the input, an optional mono sub-label (`.cf-control-sub`), a clear button (`.cf-control-btn`) and a chevron. The dropdown is `.cf-menu` with `.cf-menu-item` rows (the active row gets `.is-active`, and `.cf-menu-meta` holds a mono right-side detail). Behaviour:
  - Typing filters the options the page already loads.
  - ↑ / ↓ move the active row; Enter picks it; Esc closes.
  - Clicking outside closes the menu.
  - With no matches, show `.cf-menu-empty` "No matches".
  - The menu is at most 280px tall and scrolls.
  - Use `role="combobox"`, `aria-expanded`, `aria-controls` and `role="listbox"` / `role="option"`.
  - If the app already has a combobox component (react-select, MUI Autocomplete and so on), restyle that one to look exactly like this instead of writing a new one.
- **Date:** keep the native `<input type="date">` and whatever value format the app already converts to. Put it inside `.cf-control` with a leading calendar icon. The CSS stretches the browser's own picker button invisibly across the field, so clicking anywhere opens the picker.
- **Upload:** see Add Application below.

**Action bar** (`.cf-bar`) — sticky at the bottom; 52 / 60 / 64px tall; background `rgba(15,26,34,.92)` with an 8px blur and a 1px `#23394A` top border.
- **Left side** (`.cf-bar-meta`, 12px, muted), in this order:
  - after a successful Save: `✓ Draft saved · 2 min ago`. Teal check icon; update the relative time every 60 seconds ("just now", "1 min ago" …). Before any save, show nothing here.
  - `* Required` (star in red).
  - when the user has tried to submit and errors remain: `N fields need attention` in red (`.is-error`).
- **Right side:** buttons, the primary always last:
  - Add Application: `Reset` (ghost, rotate icon) · `Save draft` (outline, save icon) · `Submit for approval` (teal, send icon).
  - Add ITAM, Add Features, Add Actions: `Cancel` (ghost) · `Save draft` · primary.
  - Map them to the existing handlers:
    - "Save draft" = today's Save.
    - "Submit for approval" = today's Submit. If today's Submit does not start an approval, label it just "Submit".
    - Show "Save draft" only if the page has a Save today.
    - "Reset" puts the form back to its initial values (use the form library's reset if it has one).
    - "Cancel" does the same as the back button. If the form has unsaved changes and the app already has an unsaved-changes guard, keep using it.
  - While a request is running, that button gets `.is-loading` and the other buttons are disabled.

**Buttons:** 30 / 34 / 36px tall, 7 / 8 / 8px radius, 14px side padding, body-size text (12.5 / 13 / 13.5px) at weight 700, a 15px icon with a 7px gap.
- **Primary:** `#2DD4BF` background with `#062A26` text; hover `#5EEAD4`.
- **Secondary:** transparent with a 1px `#2B4556` border and `#E6EEF2` text; hover background `#152733`.
- **Ghost:** no border, `#A7B8C4` text.
- **Disabled:** 45% opacity.

**Validation timing** (same on every page)
- Do not show errors on fields the user has not touched yet.
- After a field loses focus (blur), validate it and show or hide its error.
- On Submit: validate everything, show all errors, put the "N fields need attention" count in the bar, scroll the first invalid field into view and focus it.
- On Save draft: run only the rules the app runs today for Save.
- Server errors that belong to a field go under that field.
- Other server errors go in a `.cf-alert cf-alert--error` box at the top of the first section, or above the grid on bulk pages.

---

## 3. Page — Add ITAM

**Layout:** one centred card, no progress rail. The header, the card and the action bar contents are all centred on the same column: 720 / 780 / 820px wide.

- Breadcrumb: `FMCES / App Management / ITAM / New`.
- Title: `Add ITAM`.
- Subtitle: `Register a new ITAM entry for application onboarding.`
- Card (`.cf-sec`, no number tile):
  - Title: `ITAM details`.
  - Description: `All five fields are needed to onboard an application under this ITAM.`

| Order | Label | Control | Span | Mono | Required | Help text |
|---|---|---|---|---|---|---|
| 1 | ITAM name | text | 7 | no | yes | — |
| 2 | ITAM ID | text | 5 | **yes** | yes | `The ID from the ITAM register.` |
| 3 | SNOW group | keep today's control type (text, or search select if it already loads options) | 7 | **yes** | yes | `The ServiceNow assignment group that supports this application.` |
| 4 | Support DL | text, placeholder `e.g. DL-FMCES-L2-SUPPORT` | 5 | **yes** | yes | — |
| 5 | Description | textarea, 3 rows | 12 | no | yes | counter only if a max length exists |

- Bar: `* Required` (plus `N fields need attention` after a failed submit) · `Cancel` · `Save draft` · `Submit for approval`.
- **[IF DATA EXISTS]** If the page can already check whether an ITAM ID is taken, show the result under ITAM ID on blur:
  - `.cf-success` with a check-circle icon: "Not used by any other ITAM".
  - `.cf-error`: "This ITAM ID already exists".

Markup: Appendix B.1.

---

## 4. Page — Add Application

**Layout:** "ledger" — section cards on the left, a sticky progress rail on the right (`.cf-ledger`).

- Breadcrumb: `FMCES / App Management / New`.
- Title: `Add application`.
- Subtitle: `Fill in the application details below.`
- Header badge: New / Draft.

**Section 1 — "ITAM & ownership"**
- Description: `Pick the ITAM first; its details fill in automatically.`

| Order | Label | Control | Span | Mono | Notes |
|---|---|---|---|---|---|
| 1 | ITAM ID | search select (`server` icon); sub-label = ITAM name if known | 4 | yes | required (as today) |
| 2 | *(today's second dropdown — keep its real label)* | select or search select; `disabled` with placeholder `Select ITAM ID first` until an ITAM is chosen | 4 | as fits | when ITAM ID changes, clear this value and reload its options the way the page does today |
| 3 | Application name | text | 4 | yes | |
| 4 | Filled from ITAM *{id}* — **[IF DATA EXISTS]** | `.cf-autofill` strip, 3 columns: ITAM name · SNOW group · Support DL (read-only, mono) | 12 | yes | only once an ITAM is chosen and only with the fields the ITAM object really has; hide it otherwise |
| 5 | App owner | today's control (user lookup → `.cf-control` with `user` icon and sub-label `Bank ID`; plain text → `.cf-input--mono`) | 6 | yes | |
| 6 | Application dev DL | text | 6 | yes | |
| 7 | Description | textarea, 2 rows | 12 | no | counter only if a max length exists |

If the page turns out to have **no** second dropdown, use spans 6 + 6 for ITAM ID and Application name instead.

**Section 2 — "Change & onboarding"**
- Description: `Change request and go-live details.`
- Fields:
  - CR number: text, mono, placeholder `e.g. CR-2026-004817`, span 6.
  - FMCES onboard date: date, span 6.
- Mark them Optional or required exactly as today's validation says.

**Section 3 — "OLA document"**
- Description: `The signed operating-level agreement for this application.`
- **Drop zone** (`label.cf-drop` containing the real `<input type="file">`; the input covers the zone invisibly):
  - Upload icon tile.
  - Title: `Drop the OLA here, or browse` ("browse" in link blue).
  - Hint built from the real limits, for example `PDF or Word document, up to 10 MB`. Read the accepted types and max size from the existing code; never make them up.
  - While a file is dragged over it, add `.is-dragover`.
  - Dropping a file calls the same handler as picking one.
- **Each chosen or uploaded file** shows a `.cf-file` row under the zone:
  - file icon, name (ellipsis when long), mono meta `1.2 MB · uploaded`;
  - while uploading, `uploading 45%` plus `.cf-file-bar`, but only if progress is available — otherwise `uploading…`;
  - a remove button that calls the existing remove or clear logic;
  - **[IF DATA EXISTS]** a preview (eye) button when a file URL exists.
  - Upload errors go in `.cf-error` under the zone.
  - If only one OLA is allowed, keep the zone visible and change its title to `Drop a new file to replace it, or browse`.

**Progress rail** (`.cf-rail`, sticky, top 16px)

*Card 1:*
- `PROGRESS` (small caps label) and on the right `X of Y required`. Y = the number of required fields from today's validation; X = how many hold a valid value.
- A 6px teal progress bar.
- A list of the three sections (`.cf-toc-item` buttons). Each has a status dot and right-hand meta:
  - all required fields valid → `.is-done` (teal check). Meta `Done`; for the OLA section, `1 file`.
  - has a visible error → `.is-error` (red "!"). Meta `1 issue` / `2 issues`.
  - no required fields → number dot, meta `Optional`.
  - otherwise → number dot, meta `2 of 3`.
  - while uploading → meta `Uploading…`.
- Clicking an item smoothly scrolls its section into view and focuses the section's first field.
- The section that currently contains focus gets `.is-current`.
- The section number tiles mirror these states.

*Card 2:*
- `AFTER YOU SUBMIT` — a static vertical lifecycle:
  - **Draft** "You can keep editing" (amber dot, `.is-now`)
  - Pending approval "A checker reviews it"
  - Approved "Locked as V1"
  - Active "Ready to use"

**Responsive rail:** below 1100px window width the CSS hides the rail. The bar then shows `· X of Y required` (`.cf-bar-progress`) — keep that element in the bar and fill in the same numbers.

- Bar: saved state · `* Required` · `Reset` · `Save draft` · `Submit for approval`.

Markup: Appendix B.2.

---

## 5. Page — Add Features (bulk grid)

Today this page has "Feature 1 Details" blocks with Application, Feature Name and Description, plus "+ Add More Feature". Replace that with **one shared Application field and a spreadsheet-style grid**. Keep the same request payload.

- Breadcrumb: `FMCES / App Management / Features / New`.
- Title: `Add features`.
- Subtitle: `Define one or more features for an existing application.`
- No badge.

**Card 1** (`.cf-sec`, no header — body only):
- Left, span 6:
  - `Application *`: search select, `app-window` icon, mono value, sub-label `ITAM {id}` if known.
  - Help: `Every row below is added to this application.`
  - Under it a switch (`.cf-switch`): `Use a different application per row`.
- Right, span 6, **[IF DATA EXISTS]**: label `Already in this application`, with the count on the right (`.cf-count`). Below it, mono chips (`.cf-chip cf-chip--mono`) of existing feature names.
  - Show the first 10. If there are 13 or more, add a `+N more` chip button that expands the list. Never hide just 1 or 2: if the total is 12 or fewer, show all.
  - Hide this block when no application is chosen or the per-row switch is on.

**Card 2** (`.cf-card`):

*Head row:*
- Title `Features to add`.
- Mono meta: `4 rows · 2 need attention`, or `· all ready`.
- On the right, small outline buttons: `Add row` (plus icon), `Paste from Excel` (clipboard icon), `Import CSV` (upload icon).

*The grid:* `.cf-bulk` inside `.cf-bulk-scroll`.

| Column | Width |
|---|---|
| # | 44px |
| Feature name * | minmax(180px, .9fr) |
| Description * | minmax(240px, 1.6fr) |
| Check | minmax(170px, 210px) |
| delete | 44px |

- The header row is 6px shorter than a normal row, with a `#1B3140` background and uppercase muted labels (help-text size).
- **Cells are borderless inputs that fill the cell.** The focused cell gets a 2px teal inner ring; an invalid cell gets a red inner line and a red tint.
- Feature name is mono.

*Foot row:* keyboard hints:
- `Tab` next cell
- `Enter` next row
- `Ctrl V` paste rows from Excel: name in column 1, description in column 2

**Grid behaviour**
- **Rows:** state is a list of `{ id, name, description, application?, touched }`.
  - There is **always exactly one empty row at the end** (`.is-new`, status text `New row`, no delete button). It is not counted or submitted.
  - Typing in it turns it into a real row and adds a new empty row.
  - On load: one empty row, with focus in its name cell.
- **Add row** focuses the empty row's name cell.
- **Delete** (trash) removes the row and moves focus to the same column in the next row.
- **Keyboard:**
  - Tab / Shift+Tab follow DOM order.
  - **Enter** moves to the same column in the next row (creating it if needed); **Shift+Enter** moves up.
  - Do not submit the form on Enter inside the grid.
- **Paste into a cell:** if the pasted text contains a tab or a line break, stop the default paste and parse it.
  - Split lines on `\r?\n` and drop trailing empty lines.
  - Split each line on `\t`:
    - 2 columns = name, description.
    - In per-row mode, 3 columns = application, name, description.
  - Skip the first line if its first cell looks like a header (`/^(feature\s*)?name$/i`).
  - Fill downward from the current row: write into empty cells and add rows as needed. Mark the pasted rows as touched.
  - Show a small `.cf-alert cf-alert--info` above the grid for 4 seconds: `Pasted 12 rows`.
  - A single-line paste with no tab behaves normally.
- **Paste from Excel button:** `navigator.clipboard.readText()`, then the same parser, appended after the last real row.
  - If the browser blocks it, show `.cf-alert cf-alert--info` above the grid: `Your browser blocked clipboard access. Click the first empty name cell and press Ctrl+V.` It has a close button.
- **Import CSV:**
  - A hidden `<input type="file" accept=".csv,text/csv">`, read with FileReader.
  - A small CSV parser that handles quotes, `""` escapes, and commas or line breaks inside quotes.
  - Same row filling and header skip as paste.
- **Row check:** only for touched rows, or for every row after a submit attempt. First match wins. The status column shows an icon plus text, with an ellipsis if long.
  1. Name empty → `Name needed`; the name cell is invalid.
  2. Name repeated in the grid (trimmed, case-insensitive) → `Duplicate in this list` on the 2nd and later copies.
  3. **[IF DATA EXISTS]** Name already in the application → `Name already exists`.
  4. Any format or length rule that exists today → its message.
  5. Description empty → `Description needed`; the description cell is invalid.
  6. Otherwise → `Ready`, with the teal check-circle icon (`.is-ok`).
- **Per-row application switch:**
  - **On:** add an Application column after # (`.cf-bulk--per-app`; the grid's min width becomes 980px). Each cell is a borderless search select, pre-filled with the top Application. The top field's label changes to `Default application for new rows`, and the chips block hides.
  - **Off:** if rows have different applications, confirm first (`Set every row to <app>?`), then switch back.
- **Bar:**
  - Left: `4 features · 2 need attention` (red part only when > 0), or `· all ready`.
  - Right: `Cancel` · `Save draft` (only if the page has Save today) · primary `Submit 4 features`. Use "feature" for 1; disable it at 0.
- **Submit:**
  1. Mark all rows touched.
  2. If any errors: focus the first invalid cell and scroll it into view.
  3. Otherwise call the existing submit with the existing payload: one item per real row, `{ application: row.application ?? topApplication, name, description }`, using today's exact field names.
  4. While it runs: grid inputs disabled, primary `.is-loading`.
  5. If the API returns per-item errors, show them in that row's status. Otherwise show the message in an error alert above the grid.
- **Changing the top Application** re-runs the [IF DATA EXISTS] "already exists" check. It does not clear the rows.
- If today's code has a limit on how many items can be sent, enforce it: disable Add row and show `Up to N rows` in the card meta.

**Below 760px** of card width, the grid scrolls sideways inside its card (980px in per-row mode). The page itself never scrolls sideways.

Markup: Appendix B.3. Per-row Application cell: Appendix B.4.

---

## 6. Page — Add Actions

**Identical to Add Features** — same components, layout and behaviour — with these words changed:

| Where | Add Features | Add Actions |
|---|---|---|
| Breadcrumb | `… / Features / New` | `… / Actions / New` |
| Title | `Add features` | `Add actions` |
| Subtitle | `Define one or more features for an existing application.` | `Add one or more actions for an application.` |
| Grid title | `Features to add` | `Actions to add` |
| Name column | `Feature name *` | `Action name *` |
| Description placeholder | `Describe what this feature covers` | `Describe what this action covers` |
| Chips | existing feature names | existing action names |
| Header skip regex | `/^(feature\s*)?name$/i` | `/^(action\s*)?name$/i` |
| Bar / primary | `N features` / `Submit N features` | `N actions` / `Submit N actions` |

Use the existing Add Action submit call and payload.

---

## 7. Edit pages — Edit ITAM, Edit Application, Edit Feature, Edit Action

**Each edit page looks exactly like its create page** — same components, layout, sizes and colours — just filled with the record's data.

**How to build it**
- Build each page once with a `mode` of `'create'` or `'edit'`. Keep both existing routes and render the same component from each.
- In edit mode, load the record with the existing fetch call and put its values into the same form state.
- Submit with the existing update call and **the exact payload the old edit page sends**.

**What changes in edit mode**

| Part | Create | Edit |
|---|---|---|
| Title | `Add application` / `Add ITAM` / `Add features` / `Add actions` | `Edit application` / `Edit ITAM` / `Edit feature` / `Edit action` |
| Breadcrumb | `… / New` | `… / <record name> / Edit`. For example `FMCES / App Management / RATAN_ENTITLEMENT_RULE / Edit`, `FMCES / App Management / ITAM 51358 / Edit`, `FMCES / App Management / Features / Basic_Feature / Edit` |
| Subtitle | the create subtitle | `Update the details below.` If edits go through approval, add ` Changes are reviewed before they replace the live version.` |
| Meta line under the subtitle | — | `.cf-head-meta` (mono, muted, one line, ellipsis). **[IF DATA EXISTS]** Only facts the record really has, joined with ` · `, for example `ITAM 51358 · 5 features · 5 actions · Last modified 24 Sep 2026 by 2023504` |
| Badges on the right | `New` / `Draft` | The record's current status: `cf-badge--ok` (Approved), `cf-badge--draft` (Draft), `cf-badge--pending` (Pending approval). Then **[IF DATA EXISTS]** a version badge `cf-badge--version` such as `V1`. Use the app's own status text. |
| Field values | empty | Prefilled from the record, including dates, selects and the uploaded file |
| Fields that cannot change | — | **Exactly** the fields today's edit page disables or does not let the user change — no more, no fewer. Show each as read-only: a search select or lookup becomes `.cf-control.is-readonly` with a trailing `[icon:lock 14]`; a text field becomes `.cf-input` with `readonly`. No red `*` on read-only fields. Add one help line, for example `The ITAM ID can’t be changed.` or `A feature can’t move to another application.` |
| A field the user changed | — | When a value differs from the loaded value, the label gets `<span class="cf-edited">Edited</span>` (after the `*` / Optional, before the counter) and the control gets `.is-changed` (a soft teal border). Compare trimmed text; compare dates as dates. Both disappear if the user changes the value back. |
| Bar, left side | saved state · `* Required` | `N unsaved changes` in teal (`<span class="is-changed">`, "1 unsaved change" for one), or `No changes` in muted text. Then `* Required`, then the red `N fields need attention` after a failed submit. After a successful Save draft, show `✓ Draft saved · just now` like create. |
| Bar, buttons | `Reset` or `Cancel` · `Save draft` · primary | `Discard changes` (ghost, rotate icon; resets every field to the loaded values; disabled when there are no changes) · `Save draft` (only if today's edit page has Save) · primary. The primary is `Submit for approval` if today's edit submit starts an approval, otherwise `Save changes`. **It is disabled while there are no changes.** The back button in the header still goes back. If the app already has an unsaved-changes guard, keep using it; do not add a new one. |
| While the record loads | — | Render the same page and cards with `.cf-skel` placeholders instead of fields (Appendix B.8), with the bar buttons disabled. If loading fails, show `.cf-alert cf-alert--error` with the existing error message and a `Retry` small outline button that calls the existing load again. Nothing may jump in size when the data arrives. |

**Edit ITAM**
- Same centred card as Add ITAM, with the same five fields filled in.
- ITAM ID is read-only only if today's edit page does not allow changing it.

**Edit Application**
- Same ledger layout and rail as Add Application, filled in.
- **Initial load:** set the ITAM ID first, then the second (dependent) dropdown's value. Do **not** run the "clear the dependent field when the ITAM changes" logic during the initial load — only when the user changes the ITAM.
- **Rail, section list:** status priority is **error > changes > done**.
  - A section with a visible error shows red "!" and `1 issue`.
  - Otherwise, a section with changes shows teal meta `1 change` / `2 changes` (`cf-toc-meta is-changed`).
  - Otherwise use the create rules (`Done`, `Optional`, `1 file`).
- **Rail, lifecycle card:** **[IF DATA EXISTS: versioning]** when the record is approved and editing creates a new version, the steps are:
  - **Draft V2** "You can keep editing" (now)
  - Pending approval "A checker reviews it"
  - Approved "Replaces V1"
  - Active "V1 stays live until then"

  Use the real next version number. Without versioning, keep the create lifecycle.
- **OLA:** the stored file shows as a `.cf-file` row with mono meta `1.2 MB · uploaded 24 Sep 2026`.
  - Only show the parts you have.
  - Add a preview (eye) button if a file URL exists, and a remove button.
  - The drop zone title becomes `Drop a new file to replace it, or browse`.

**Edit Feature and Edit Action**
- The same page as Add Features / Add Actions, with **exactly the row(s) the edit page loads** — normally one. If today's edit page edits several items at once, show them all as rows.
- **Application field:**
  - Read-only if today's edit page does not allow moving the item, with help `A feature can’t move to another application.` (or `An action can’t…`).
  - No "different application per row" switch.
- **Chips:** **[IF DATA EXISTS]** label `Other features in this application` / `Other actions in this application`, without the item being edited.
- **Grid card:**
  - Title singular: `Feature` / `Action`.
  - Meta `1 row · 1 change` (`1 change` in teal) or `1 row · no changes`.
  - **No** Add row / Paste from Excel / Import CSV buttons.
  - **No** trailing empty row.
  - The delete cell stays empty; keep the column so the grid lines up with the create page.
  - The foot only shows `Tab next cell`.
- **Changed cell:** `.cf-bulk-cell.is-changed` (small teal dot in its top-right corner). The status shows `Ready · edited` when changed and valid, and `Ready` when unchanged.
- **Duplicate check:** ignore the record's own original name.
- **Primary button:** follows the edit rule above (`Submit for approval` or `Save changes`), never `Submit N features`.

Markup: Appendix B.5 (Edit ITAM), B.6 (Edit Application), B.7 (Edit Feature; Edit Action uses the words in section 6), B.8 (loading placeholders).

---

## 8. Responsive behaviour (already in the CSS — do not override it)

| Situation | What happens |
|---|---|
| Window < 1440 wide **or** < 820 tall (laptops, 125% Windows scaling) | Compact tier: 30px controls, 12.5px text, 52px bar |
| ≥ 1440 × 820 | Comfortable tier: 34px controls, 13px text, 60px bar |
| ≥ 1880 × 980 | Large tier: 36px controls, 13.5px text, 64px bar |
| Window < 1100 wide | Progress rail hidden; the bar shows `· X of Y required` |
| A section card < 600px wide | Every field goes full width; the auto-fill strip goes to 1 column |
| Grid narrower than 760px (980px per-row) | The grid scrolls sideways inside its card |
| Very wide screens | Content stops at its max width (1320 / 1360 / 1480px), left aligned; ITAM stays centred |

There must never be a horizontal scrollbar on the page itself.

---

## 9. Accessibility

- Every control has a visible `<label for>`. Icon-only buttons have an `aria-label`: back, clear, remove file, delete row.
- Required: native `required` or `aria-required`. Errors: `aria-invalid` plus `aria-describedby` pointing at the error line.
- The bulk grid uses `role="grid"`, `role="row"`, `role="columnheader"` and `role="gridcell"`. Each cell input has an `aria-label` like `Feature name, row 3`.
- The progress bar uses `role="progressbar"` with `aria-valuemin`, `aria-valuemax` and `aria-valuenow`.
- Keyboard focus always shows the 2px `#5EEAD4` outline (`:focus-visible`). Never remove outlines.
- `prefers-reduced-motion` turns off transitions (already in the CSS).

---

## 10. Acceptance checks — do all of these before you finish

Test in Chrome at **1366×768, 1440×900, 1920×1080 and 1024×768** (DevTools device toolbar):

1. **Inputs, selects, search selects, date fields and buttons** are exactly **30px** tall at 1366×768, **34px** at 1440×900 and **36px** at 1920×1080. Body text is 12.5 / 13 / 13.5px. Measure them in DevTools.
2. **The action bar stays pinned to the bottom** of the window while you scroll the Add Application page at 1366×768. Its buttons line up with the right edge of the page content.
3. **No horizontal page scroll** at any size. At 1024×768 the Application rail is hidden and the bar shows `· X of Y required`.
4. **Add ITAM** is one centred card, with header, card and bar contents on the same centred column.
5. **Every label is above its field, with one red `*` for required fields** and `Optional` for optional fields. No `**`, no floating labels, no ALL-CAPS section titles, no coloured left bars, no glass panels.
6. **Choosing an ITAM** on Add Application enables the second dropdown. Changing the ITAM clears it. The auto-fill strip appears only if the data exists.
7. **OLA upload:**
   - drag-over highlights the zone;
   - dropping a file uploads it with the existing handler;
   - the file row shows name, size and status;
   - remove works.
8. **Bulk grid:**
   - typing in the last row adds a new empty row;
   - Enter moves down;
   - pasting 5 rows copied from Excel creates 5 rows;
   - a duplicate name shows `Duplicate in this list`;
   - Submit with errors focuses the first bad cell;
   - a valid submit sends the **same payload shape as before**. Compare in the Network tab with the old page.
9. **Save, Submit, Reset and Cancel** do what they did before. Success and error toasts still appear. Navigation after submit is unchanged.
10. **Edit pages — values:** open each of the four edit pages on a real record. Every field shows the stored value. While it loads you see placeholders, and nothing jumps when the data arrives. Heights and spacing are identical to the create page.
11. **Edit pages — changes:**
    - Changing a field shows `Edited` and updates `N unsaved changes`.
    - Changing it back removes both.
    - `Discard changes` restores every value.
    - The primary button is disabled while there are no changes.
12. **Edit pages — read-only and payload:**
    - Read-only (lock) fields are exactly the ones the old edit page locked.
    - Saving sends the **same payload as the old edit page** (compare in the Network tab).
    - On Edit Application, the dependent dropdown keeps its stored value after load.
13. **No console errors.** Existing tests still pass. Update snapshots only for markup changes.

---

## Appendix A — stylesheet (copy verbatim into `create-forms.css`)

```css
/* ===================== CREATE FORMS — tokens ===================== */
.cf {
  /* colours (dark theme) — same palette as the detail drawer */
  --c-bg: #0F1A22;          /* page background */
  --c-field: #111E27;       /* inside of inputs, grids, drop zones */
  --c-card: #152733;        /* section cards, rail cards */
  --c-card-2: #1B3140;      /* raised: table headers, hover, number tiles */
  --c-border: #23394A;      /* card and input borders */
  --c-border-hover: #2E4A5D;
  --c-hair: #1E3240;        /* dividers inside cards */
  --c-btn-border: #2B4556;  /* outline buttons, chips, dashed boxes */
  --c-text: #E6EEF2;
  --c-text-2: #A7B8C4;
  --c-muted: #86A0AE;
  --c-placeholder: #7890A0;
  --c-accent: #2DD4BF;
  --c-accent-strong: #5EEAD4;
  --c-accent-tint: rgba(45, 212, 191, .12);
  --c-accent-ring: rgba(45, 212, 191, .16);
  --c-on-accent: #062A26;
  --c-link: #7DBBFF;
  --c-err: #F4A3A3;  --c-err-tint: rgba(244, 163, 163, .10);  --c-err-line: rgba(244, 163, 163, .55);
  --c-warn: #F6C177; --c-warn-tint: rgba(246, 193, 119, .13);
  --c-info: #A9D1FF; --c-info-tint: rgba(125, 187, 255, .12);
  --c-bar: rgba(15, 26, 34, .92);
  --c-focus: #5EEAD4;
  --ease: cubic-bezier(.22, 1, .36, 1);
  --spring: cubic-bezier(.34, 1.36, .64, 1);
  --font: 'Manrope', system-ui, -apple-system, 'Segoe UI', sans-serif;
  --mono: 'JetBrains Mono', ui-monospace, 'Cascadia Mono', Consolas, monospace;

  /* density: COMPACT (default — laptops, 125% Windows scaling, short screens) */
  --fs-xs: 10.5px; --fs-sm: 11.5px; --fs-md: 12.5px; --fs-lg: 13.5px; --fs-mono: 12px;
  --fs-h1: 19px; --fs-h2: 13.5px;
  --h-input: 30px; --h-btn: 30px; --h-btn-sm: 26px; --h-chip: 22px; --h-bar: 52px; --h-row: 38px;
  --r: 7px; --r-card: 10px;
  --gap-x: 12px; --gap-y: 12px; --gap-label: 5px; --gap-sec: 12px;
  --sec-pad: 14px 16px 16px; --sec-head-pad: 12px 16px 0;
  --page-pad-l: 16px; --page-pad-r: 24px; --page-pad-top: 16px;
  --rail-w: 256px; --center-w: 720px; --page-max: 1320px;
}
/* density: COMFORTABLE — 1440×900 class screens */
@media (min-width: 1440px) and (min-height: 820px) {
  .cf {
    --fs-xs: 11px; --fs-sm: 12px; --fs-md: 13px; --fs-lg: 14px; --fs-mono: 12.5px;
    --fs-h1: 22px; --fs-h2: 14.5px;
    --h-input: 34px; --h-btn: 34px; --h-btn-sm: 28px; --h-chip: 24px; --h-bar: 60px; --h-row: 42px;
    --r: 8px; --r-card: 12px;
    --gap-x: 16px; --gap-y: 14px; --gap-label: 6px; --gap-sec: 16px;
    --sec-pad: 18px 20px 20px; --sec-head-pad: 16px 20px 0;
    --page-pad-l: 16px; --page-pad-r: 32px; --page-pad-top: 20px;
    --rail-w: 288px; --center-w: 780px; --page-max: 1360px;
  }
}
/* density: LARGE — 1920×1080 at 100% scaling */
@media (min-width: 1880px) and (min-height: 980px) {
  .cf {
    --fs-md: 13.5px; --fs-lg: 14.5px; --fs-mono: 13px; --fs-h1: 24px; --fs-h2: 15px;
    --h-input: 36px; --h-btn: 36px; --h-row: 44px; --h-bar: 64px;
    --gap-x: 20px; --gap-y: 16px; --gap-sec: 20px;
    --sec-pad: 20px 24px 24px; --sec-head-pad: 18px 24px 0;
    --page-pad-r: 40px; --page-pad-top: 24px;
    --rail-w: 320px; --center-w: 820px; --page-max: 1480px;
  }
}

/* ===================== page shell ===================== */
.cf { display: flex; flex-direction: column; min-height: 100%; background: var(--c-bg); color: var(--c-text);
  font-family: var(--font); font-size: var(--fs-md); line-height: 1.45; color-scheme: dark; -webkit-font-smoothing: antialiased; }
.cf *, .cf *::before, .cf *::after { box-sizing: border-box; }
.cf button { font-family: inherit; cursor: pointer; }
.cf h1, .cf h2, .cf h3, .cf p { margin: 0; }
.cf :focus-visible { outline: 2px solid var(--c-focus); outline-offset: 2px; }
.cf a { color: var(--c-link); text-decoration: none; }
.cf a:hover { color: #B3D7FF; }
.cf svg { display: block; flex-shrink: 0; }                 /* icons: 24×24 stroke icons drawn at 14–18px */
.cf .cf-mono { font-family: var(--mono); font-size: var(--fs-mono); }
.cf-hidden { display: none !important; }

.cf-inner { width: 100%; max-width: var(--page-max); display: flex; flex-direction: column; flex: 1 0 auto;
  padding: 0 var(--page-pad-r) 24px var(--page-pad-l); }
.cf-inner--center { max-width: calc(var(--center-w) + var(--page-pad-l) + var(--page-pad-r)); margin: 0 auto; }

/* page header */
.cf-head { display: flex; align-items: flex-start; gap: 14px; padding: var(--page-pad-top) 0 calc(var(--gap-sec) - 2px); }
.cf-back { width: var(--h-btn); height: var(--h-btn); flex-shrink: 0; display: flex; align-items: center; justify-content: center; margin-top: 14px;
  border-radius: 9px; border: 1px solid var(--c-btn-border); background: var(--c-card); color: var(--c-text-2); }
.cf-back:hover { background: var(--c-card-2); color: var(--c-text); }
.cf-head-text { display: flex; flex-direction: column; min-width: 0; }
.cf-crumb { font-family: var(--mono); font-size: var(--fs-xs); color: var(--c-muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-crumb a { color: var(--c-muted); } .cf-crumb a:hover { color: var(--c-text); }
.cf-title { font-size: var(--fs-h1); font-weight: 800; letter-spacing: -.015em; line-height: 1.25; margin-top: 3px; }
.cf-sub { font-size: var(--fs-md); color: var(--c-text-2); margin-top: 2px; }
.cf-head-right { margin-left: auto; display: flex; align-items: center; gap: 10px; margin-top: 14px; }

/* layouts */
.cf-ledger { display: grid; grid-template-columns: minmax(0, 1fr) var(--rail-w); gap: calc(var(--gap-sec) + 8px); align-items: start; }
.cf-stack { display: flex; flex-direction: column; gap: var(--gap-sec); min-width: 0; }
@media (max-width: 1099px) {
  .cf-ledger { grid-template-columns: minmax(0, 1fr); }
  .cf-rail { display: none !important; }
  .cf-bar-progress { display: inline-flex !important; }
}

/* ===================== section card ===================== */
.cf-sec { background: var(--c-card); border: 1px solid var(--c-border); border-radius: var(--r-card); container-type: inline-size; scroll-margin-top: 16px; }
.cf-sec-head { display: flex; align-items: flex-start; gap: 12px; padding: var(--sec-head-pad); }
.cf-sec-num { width: 24px; height: 24px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; margin-top: 1px; border-radius: 7px;
  background: var(--c-card-2); border: 1px solid var(--c-btn-border); color: var(--c-accent-strong); font: 600 11.5px var(--mono);
  transition: background-color .25s, border-color .25s, color .25s; }
.cf-sec-num.is-done { background: var(--c-accent); border-color: var(--c-accent); color: var(--c-on-accent); }
.cf-sec-num.is-error { background: var(--c-err-tint); border-color: var(--c-err-line); color: var(--c-err); }
.cf-sec-titles { flex: 1; min-width: 0; }
.cf-sec-title { font-size: var(--fs-h2); font-weight: 800; line-height: 1.35; }
.cf-sec-desc { font-size: var(--fs-sm); color: var(--c-muted); margin-top: 1px; }
.cf-sec-body { padding: var(--sec-pad); }
.cf-sec-foot { display: flex; align-items: center; gap: 10px; padding: 12px 20px; border-top: 1px solid var(--c-hair); font-size: var(--fs-xs); color: var(--c-muted); }

/* 12-column field grid */
.cf-grid { display: grid; grid-template-columns: repeat(12, minmax(0, 1fr)); gap: var(--gap-y) var(--gap-x); }
.cf-span-4 { grid-column: span 4; } .cf-span-5 { grid-column: span 5; } .cf-span-6 { grid-column: span 6; }
.cf-span-7 { grid-column: span 7; } .cf-span-8 { grid-column: span 8; } .cf-span-12 { grid-column: span 12; }
@container (max-width: 600px) {
  .cf-grid > [class*="cf-span-"] { grid-column: span 12; }
}

/* ===================== field ===================== */
.cf-field { display: flex; flex-direction: column; gap: var(--gap-label); min-width: 0; }
.cf-label { display: flex; align-items: center; gap: 6px; font-size: var(--fs-sm); font-weight: 700; color: var(--c-text-2); line-height: 1.3; }
.cf-req { color: var(--c-err); font-weight: 800; margin-left: -3px; }
.cf-opt { font-size: var(--fs-xs); font-weight: 600; color: var(--c-muted); }
.cf-count { margin-left: auto; font: 500 10.5px var(--mono); color: var(--c-muted); }
.cf-count.is-over { color: var(--c-err); }

.cf-input { display: block; width: 100%; height: var(--h-input); padding: 0 10px; margin: 0; border-radius: var(--r); border: 1px solid var(--c-border);
  background: var(--c-field); color: var(--c-text); font: inherit; font-size: var(--fs-md); outline: none;
  transition: border-color .15s, box-shadow .15s, background-color .15s; }
.cf-input:hover { border-color: var(--c-border-hover); }
.cf-input::placeholder { color: var(--c-placeholder); opacity: 1; }
.cf-input:focus, .cf-control:focus-within { border-color: var(--c-accent); box-shadow: 0 0 0 3px var(--c-accent-ring); }
.cf-input--mono, .cf-input--mono input { font-family: var(--mono); font-size: var(--fs-mono); }
.cf-input.is-invalid, .cf-control.is-invalid { border-color: var(--c-err-line); background: linear-gradient(var(--c-err-tint), var(--c-err-tint)), var(--c-field); }
.cf-input.is-invalid:focus, .cf-control.is-invalid:focus-within { box-shadow: 0 0 0 3px var(--c-err-tint); }
.cf-input[readonly], .cf-input.is-readonly { background: transparent; border-style: dashed; color: var(--c-text-2); }
.cf-input:disabled, .cf-control.is-disabled { opacity: .5; cursor: not-allowed; }
textarea.cf-input { height: auto; min-height: calc(var(--h-input) * 2.2); padding: 8px 10px; line-height: 1.5; resize: vertical; }

/* select — native <select> with a drawn chevron */
.cf-select { position: relative; display: block; }
.cf-select select.cf-input { appearance: none; -webkit-appearance: none; padding-right: 32px; cursor: pointer; }
.cf-select select.cf-input:invalid, .cf-select select.cf-input.is-empty { color: var(--c-placeholder); }
.cf-select > svg { position: absolute; right: 10px; top: 50%; margin-top: -8px; color: var(--c-muted); pointer-events: none; }
.cf-select option { background: #172A36; color: var(--c-text); }

/* control = input with icons / sub-label / clear button (search select, date, user lookup) */
.cf-control { display: flex; align-items: center; gap: 8px; width: 100%; height: var(--h-input); padding: 0 8px 0 10px; border-radius: var(--r);
  border: 1px solid var(--c-border); background: var(--c-field); color: var(--c-text); transition: border-color .15s, box-shadow .15s; }
.cf-control:hover { border-color: var(--c-border-hover); }
.cf-control > svg { color: var(--c-muted); }
.cf-control input { flex: 1; min-width: 0; height: 100%; padding: 0; border: 0; outline: 0; background: transparent; color: inherit; font: inherit; font-size: var(--fs-md); }
.cf-control input::placeholder { color: var(--c-placeholder); opacity: 1; }
.cf-control input[type="date"] { position: relative; }
.cf-control input[type="date"]::-webkit-calendar-picker-indicator { position: absolute; inset: 0; width: auto; height: auto; opacity: 0; cursor: pointer; } /* whole field opens the picker; our calendar icon stays */
.cf-control-sub { font: 500 11px/1.5 var(--mono); color: var(--c-muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 45%; }
.cf-control-btn { width: 22px; height: 22px; flex-shrink: 0; display: inline-flex; align-items: center; justify-content: center; padding: 0; border: 0; border-radius: 5px; background: transparent; color: var(--c-muted); }
.cf-control-btn:hover { background: var(--c-card-2); color: var(--c-text); }

/* dropdown list for search selects */
.cf-menu { position: absolute; z-index: 60; left: 0; right: 0; top: calc(100% + 4px); max-height: 280px; overflow: auto; padding: 6px;
  background: #172A36; border: 1px solid #2B4556; border-radius: 10px; box-shadow: 0 20px 44px -16px rgba(0, 0, 0, .75); }
.cf-menu-item { display: flex; align-items: center; gap: 10px; width: 100%; min-height: var(--h-input); padding: 0 10px; border: 0; border-radius: 7px;
  background: transparent; color: var(--c-text); font-size: var(--fs-md); font-weight: 600; text-align: left; }
.cf-menu-item:hover, .cf-menu-item.is-active { background: var(--c-card-2); }
.cf-menu-item > svg { color: var(--c-accent); }
.cf-menu-item .cf-menu-meta { margin-left: auto; font: 500 11px/1.5 var(--mono); color: var(--c-muted); white-space: nowrap; }
.cf-menu-empty { padding: 10px; font-size: var(--fs-sm); color: var(--c-muted); }
.cf-combo { position: relative; }

/* help / error / success line under a field (one at a time) */
.cf-help { font-size: var(--fs-xs); color: var(--c-muted); line-height: 1.4; }
.cf-error { display: flex; align-items: center; gap: 5px; font-size: var(--fs-xs); color: var(--c-err); line-height: 1.4; }
.cf-success { display: flex; align-items: center; gap: 5px; font-size: var(--fs-xs); color: var(--c-accent-strong); line-height: 1.4; }

/* values copied from a parent record (e.g. from the selected ITAM) */
.cf-autofill { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px; padding: 10px 12px; border-radius: 9px;
  background: var(--c-field); border: 1px dashed var(--c-btn-border); }
.cf-autofill-k { display: block; font-size: var(--fs-xs); font-weight: 600; color: var(--c-muted); }
.cf-autofill-v { display: block; font: 500 var(--fs-sm)/1.5 var(--mono); color: var(--c-text); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; margin-top: 3px; }
@container (max-width: 600px) { .cf-autofill { grid-template-columns: minmax(0, 1fr); } }

/* file upload */
.cf-upload { display: flex; flex-direction: column; gap: 8px; }
.cf-drop { position: relative; display: flex; align-items: center; gap: 14px; padding: 14px 16px; border: 1.5px dashed var(--c-btn-border); border-radius: 10px;
  background: var(--c-field); transition: border-color .15s, background-color .15s; }
.cf-drop.is-dragover { border-color: var(--c-accent); background: var(--c-accent-tint); }
.cf-drop.is-invalid { border-color: var(--c-err-line); }
.cf-drop input[type="file"] { position: absolute; inset: 0; opacity: 0; cursor: pointer; }
.cf-drop-icon { width: 36px; height: 36px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; border-radius: 9px; background: var(--c-card-2); color: var(--c-accent-strong); }
.cf-drop-title { font-weight: 700; }
.cf-drop-title span { color: var(--c-link); }
.cf-drop-hint { font-size: var(--fs-xs); color: var(--c-muted); }
.cf-file { display: flex; align-items: center; gap: 10px; min-height: 44px; padding: 6px 8px 6px 10px; border: 1px solid var(--c-border); border-radius: 9px; background: var(--c-field); }
.cf-file-icon { width: 28px; height: 28px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; border-radius: 7px; background: var(--c-info-tint); color: var(--c-info); }
.cf-file-text { flex: 1; min-width: 0; display: flex; flex-direction: column; }
.cf-file-name { font-size: var(--fs-sm); font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-file-meta { font: 500 var(--fs-xs)/1.5 var(--mono); color: var(--c-muted); }
.cf-file-bar { height: 3px; border-radius: 3px; background: var(--c-hair); overflow: hidden; margin-top: 4px; }
.cf-file-bar i { display: block; height: 100%; background: var(--c-accent); }

/* chips */
.cf-chips { display: flex; flex-wrap: wrap; gap: 6px; }
.cf-chip { display: inline-flex; align-items: center; height: var(--h-chip); max-width: 100%; padding: 0 9px; border-radius: 6px; background: var(--c-card-2);
  border: 1px solid var(--c-btn-border); color: var(--c-text); font-size: var(--fs-sm); font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-chip--mono { font-family: var(--mono); font-size: 11.5px; }

/* switch */
.cf-switch { position: relative; display: inline-flex; align-items: center; gap: 9px; font-size: var(--fs-sm); font-weight: 600; color: var(--c-text-2); cursor: pointer; white-space: nowrap; }
.cf-switch input { position: absolute; opacity: 0; width: 1px; height: 1px; }
.cf-switch-track { position: relative; width: 30px; height: 18px; flex-shrink: 0; border-radius: 999px; background: #2A4253; transition: background-color .2s; }
.cf-switch-track::after { content: ""; position: absolute; left: 2px; top: 2px; width: 14px; height: 14px; border-radius: 50%; background: #C9D6DD; transition: transform .22s var(--spring), background-color .2s; }
.cf-switch input:checked + .cf-switch-track { background: var(--c-accent); }
.cf-switch input:checked + .cf-switch-track::after { transform: translateX(12px); background: #fff; }
.cf-switch input:focus-visible + .cf-switch-track { outline: 2px solid var(--c-focus); outline-offset: 2px; }

/* ===================== buttons ===================== */
.cf-btn { display: inline-flex; align-items: center; justify-content: center; gap: 7px; height: var(--h-btn); padding: 0 14px; border-radius: var(--r);
  border: 1px solid transparent; font-size: var(--fs-md); font-weight: 700; white-space: nowrap; transition: background-color .15s, border-color .15s, color .15s; }
.cf-btn--primary { background: var(--c-accent); color: var(--c-on-accent); }
.cf-btn--primary:hover { background: var(--c-accent-strong); }
.cf-btn--secondary { background: transparent; border-color: var(--c-btn-border); color: var(--c-text); }
.cf-btn--secondary:hover { background: var(--c-card); }
.cf-btn--ghost { background: transparent; color: var(--c-text-2); padding: 0 10px; }
.cf-btn--ghost:hover { background: var(--c-card); color: var(--c-text); }
.cf-btn--danger { background: transparent; color: var(--c-err); padding: 0 10px; }
.cf-btn--danger:hover { background: var(--c-err-tint); }
.cf-btn--sm { height: var(--h-btn-sm); padding: 0 10px; font-size: var(--fs-sm); border-radius: 7px; }
.cf-btn:disabled { opacity: .45; cursor: not-allowed; }
.cf-btn.is-loading { pointer-events: none; opacity: .8; }
.cf-icon-btn { width: 28px; height: 28px; flex-shrink: 0; display: inline-flex; align-items: center; justify-content: center; padding: 0; border: 0; border-radius: 7px;
  background: transparent; color: var(--c-muted); }
.cf-icon-btn:hover { background: var(--c-card-2); color: var(--c-text); }
.cf-icon-btn--danger:hover { background: var(--c-err-tint); color: var(--c-err); }
.cf-kbd { font: 500 10.5px var(--mono); padding: 1px 5px; border-radius: 4px; border: 1px solid var(--c-btn-border); color: var(--c-muted); }

/* badges & alerts */
.cf-badge { display: inline-flex; align-items: center; gap: 5px; height: 20px; padding: 0 7px; border-radius: 5px; font-size: 10.5px; font-weight: 800;
  letter-spacing: .06em; text-transform: uppercase; white-space: nowrap; }
.cf-badge::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: currentColor; }
.cf-badge--draft { background: var(--c-warn-tint); color: var(--c-warn); }
.cf-badge--new { background: var(--c-info-tint); color: var(--c-info); }
.cf-alert { display: flex; align-items: flex-start; gap: 10px; padding: 10px 12px; border-radius: 9px; font-size: var(--fs-sm); line-height: 1.45; }
.cf-alert > svg { margin-top: 1px; }
.cf-alert--info { background: var(--c-info-tint); color: #CFE4FF; } .cf-alert--info > svg { color: var(--c-info); }
.cf-alert--error { background: var(--c-err-tint); color: #F8CACA; } .cf-alert--error > svg { color: var(--c-err); }

/* ===================== progress rail (Application page) ===================== */
.cf-rail { position: sticky; top: 16px; display: flex; flex-direction: column; gap: 12px; }
.cf-rail-card { display: flex; flex-direction: column; gap: 12px; padding: 14px 16px; background: var(--c-card); border: 1px solid var(--c-border); border-radius: var(--r-card); }
.cf-rail-row { display: flex; align-items: center; justify-content: space-between; gap: 8px; }
.cf-rail-title { font-size: var(--fs-xs); font-weight: 700; letter-spacing: .1em; text-transform: uppercase; color: var(--c-muted); }
.cf-rail-count { font-size: var(--fs-sm); font-weight: 700; }
.cf-progress { height: 6px; border-radius: 999px; background: var(--c-field); border: 1px solid var(--c-hair); overflow: hidden; }
.cf-progress i { display: block; height: 100%; border-radius: 999px; background: var(--c-accent); transition: width .4s var(--ease); }
.cf-toc { display: flex; flex-direction: column; gap: 2px; margin: 0 -8px; }
.cf-toc-item { display: flex; align-items: center; gap: 10px; min-height: 36px; padding: 4px 8px; border: 0; border-radius: 8px; background: transparent;
  color: var(--c-text); font-size: var(--fs-md); font-weight: 600; text-align: left; width: 100%; }
.cf-toc-item:hover, .cf-toc-item.is-current { background: var(--c-card-2); }
.cf-toc-dot { width: 18px; height: 18px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; border-radius: 50%;
  border: 1.5px solid #3D5A6C; color: var(--c-muted); font: 600 10px var(--mono); }
.cf-toc-item.is-done .cf-toc-dot { background: var(--c-accent); border-color: var(--c-accent); color: var(--c-on-accent); }
.cf-toc-item.is-error .cf-toc-dot { background: var(--c-err-tint); border-color: var(--c-err-line); color: var(--c-err); }
.cf-toc-name { flex: 1; min-width: 0; line-height: 1.3; }
.cf-toc-meta { font-size: var(--fs-xs); font-weight: 600; color: var(--c-muted); white-space: nowrap; }
.cf-toc-item.is-error .cf-toc-meta { color: var(--c-err); }
.cf-life { display: flex; flex-direction: column; }
.cf-life-step { display: grid; grid-template-columns: 18px minmax(0, 1fr); gap: 10px; font-size: var(--fs-sm); }
.cf-life-node { display: flex; flex-direction: column; align-items: center; }
.cf-life-node i { width: 10px; height: 10px; margin-top: 4px; flex-shrink: 0; border-radius: 50%; border: 2px solid #3A5566; background: var(--c-card); }
.cf-life-node b { flex: 1; width: 2px; min-height: 14px; background: var(--c-hair); }
.cf-life-step:last-child .cf-life-node b { display: none; }
.cf-life-step.is-now .cf-life-node i { border-color: var(--c-warn); background: var(--c-warn); }
.cf-life-text { padding-bottom: 10px; color: var(--c-text-2); }
.cf-life-step.is-now .cf-life-text { color: var(--c-text); font-weight: 700; }
.cf-life-text small { display: block; font-size: var(--fs-xs); font-weight: 500; color: var(--c-muted); }

/* ===================== sticky action bar ===================== */
.cf-bar { position: sticky; bottom: 0; z-index: 30; margin-top: auto; background: var(--c-bar); border-top: 1px solid var(--c-border);
  -webkit-backdrop-filter: blur(8px); backdrop-filter: blur(8px); }
.cf-bar-inner { display: flex; align-items: center; gap: 10px; width: 100%; max-width: var(--page-max); height: var(--h-bar);
  padding: 0 var(--page-pad-r) 0 calc(var(--page-pad-l) + 8px); }
.cf-bar-inner--center { max-width: calc(var(--center-w) + var(--page-pad-l) + var(--page-pad-r)); margin: 0 auto; padding-left: var(--page-pad-l); }
.cf-bar-meta { display: flex; align-items: center; gap: 8px; min-width: 0; font-size: var(--fs-sm); color: var(--c-muted); white-space: nowrap; }
.cf-bar-meta .is-error { color: var(--c-err); }
.cf-bar-spacer { flex: 1; }
.cf-saved { display: inline-flex; align-items: center; gap: 6px; }
.cf-saved > svg { color: var(--c-accent); }
.cf-bar-progress { display: none; align-items: center; gap: 6px; }

/* ===================== bulk grid (Features / Actions) ===================== */
.cf-card { background: var(--c-card); border: 1px solid var(--c-border); border-radius: var(--r-card); min-width: 0; }
.cf-card-head { display: flex; align-items: center; flex-wrap: wrap; gap: 10px 12px; padding: 12px 16px; border-bottom: 1px solid var(--c-hair); }
.cf-card-title { font-size: var(--fs-h2); font-weight: 800; }
.cf-card-meta { flex: 1; min-width: 0; font: 500 11.5px/1.5 var(--mono); color: var(--c-muted); }
.cf-card-meta .is-error { color: var(--c-err); }
.cf-card-tools { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.cf-card-body { padding: var(--sec-pad); }
.cf-card-foot { display: flex; align-items: center; flex-wrap: wrap; gap: 8px 14px; padding: 10px 16px; border-top: 1px solid var(--c-hair); font-size: var(--fs-xs); color: var(--c-muted); }
.cf-card-foot span { display: inline-flex; align-items: center; gap: 6px; }

.cf-bulk-scroll { overflow-x: auto; }
.cf-bulk { --cols: 44px minmax(180px, .9fr) minmax(240px, 1.6fr) minmax(170px, 210px) 44px; min-width: 760px; }
.cf-bulk--per-app { --cols: 44px minmax(200px, 1fr) minmax(170px, .9fr) minmax(220px, 1.4fr) minmax(170px, 200px) 44px; min-width: 980px; }
.cf-bulk-row { display: grid; grid-template-columns: var(--cols); align-items: stretch; min-height: var(--h-row); border-top: 1px solid var(--c-hair); }
.cf-bulk-row:first-child { border-top: 0; }
.cf-bulk-row > * { min-width: 0; display: flex; align-items: center; }
.cf-bulk-head { min-height: calc(var(--h-row) - 6px); background: var(--c-card-2); font-size: var(--fs-xs); font-weight: 700; letter-spacing: .08em; text-transform: uppercase; color: var(--c-muted); }
.cf-bulk-head > * { padding: 0 10px; gap: 4px; }
.cf-bulk-head .cf-req { margin-left: 0; }
.cf-bulk-num { justify-content: center; font: 500 11.5px var(--mono); color: var(--c-muted); border-right: 1px solid var(--c-hair); }
.cf-bulk-cell { position: relative; }
.cf-bulk-cell + .cf-bulk-cell { border-left: 1px solid var(--c-hair); }
.cf-bulk-cell > input { width: 100%; height: 100%; min-height: var(--h-row); padding: 0 10px; border: 0; border-radius: 2px; outline: 0;
  background: transparent; color: var(--c-text); font: inherit; font-size: var(--fs-md); }
.cf-bulk-cell > input.cf-mono { font-family: var(--mono); font-size: var(--fs-mono); }
.cf-bulk-cell > input::placeholder { color: var(--c-placeholder); opacity: 1; }
.cf-bulk-cell > input:focus { box-shadow: inset 0 0 0 2px var(--c-accent); background: rgba(45, 212, 191, .05); }
.cf-bulk-cell.is-invalid > input { box-shadow: inset 0 0 0 1.5px var(--c-err-line); background: var(--c-err-tint); }
.cf-bulk-cell.is-invalid > input:focus { box-shadow: inset 0 0 0 2px var(--c-err); }
.cf-bulk-status { gap: 6px; padding: 0 10px; font-size: var(--fs-sm); border-left: 1px solid var(--c-hair); }
.cf-bulk-status > span { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-bulk-status.is-ok { color: var(--c-accent-strong); }
.cf-bulk-status.is-error { color: var(--c-err); }
.cf-bulk-status.is-new { color: var(--c-muted); }
.cf-bulk-del { justify-content: center; }
.cf-bulk-row.is-new { background: rgba(255, 255, 255, .012); }
.cf-bulk-cell > .cf-combo { flex: 1; min-width: 0; height: 100%; }
.cf-bulk-cell > .cf-combo > .cf-control { height: 100%; min-height: var(--h-row); border: 0; border-radius: 2px; background: transparent; }
.cf-bulk-cell > .cf-combo > .cf-control:focus-within { box-shadow: inset 0 0 0 2px var(--c-accent); background: rgba(45, 212, 191, .05); }

/* ===================== edit mode (same pages, prefilled) ===================== */
.cf-head-meta { font: 500 var(--fs-xs)/1.5 var(--mono); color: var(--c-muted); margin-top: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-badge--ok { background: var(--c-accent-tint); color: var(--c-accent-strong); }
.cf-badge--pending { background: rgba(170, 150, 255, .12); color: #C9B6FF; }
.cf-badge--version { background: var(--c-card-2); color: var(--c-text-2); font-family: var(--mono); font-weight: 600; letter-spacing: 0; text-transform: none; }
.cf-badge--version::before { display: none; }
.cf-edited { display: inline-flex; align-items: center; gap: 4px; font-size: var(--fs-xs); font-weight: 700; color: var(--c-accent-strong); }
.cf-edited::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: var(--c-accent); }
.cf-input.is-changed, .cf-control.is-changed { border-color: rgba(45, 212, 191, .45); }
.cf-control.is-readonly { background: transparent; border-style: dashed; color: var(--c-text-2); }
.cf-control.is-readonly:hover { border-color: var(--c-border); }
.cf-control.is-readonly input { cursor: default; }
.cf-toc-meta.is-changed, .cf-bar-meta .is-changed, .cf-card-meta .is-changed { color: var(--c-accent-strong); }
.cf-bulk-cell.is-changed::after { content: ""; position: absolute; top: 6px; right: 6px; width: 6px; height: 6px; border-radius: 50%; background: var(--c-accent); pointer-events: none; }
/* loading placeholders while the record loads */
.cf-skel { display: block; height: var(--h-input); border-radius: var(--r); background: linear-gradient(90deg, #142430 0%, #1B3140 50%, #142430 100%);
  background-size: 200% 100%; animation: cf-shimmer 1.4s ease-in-out infinite; }
.cf-skel--label { height: 10px; width: 35%; border-radius: 5px; }
@keyframes cf-shimmer { from { background-position: 100% 0; } to { background-position: -100% 0; } }

/* ===================== motion ===================== */
@media (prefers-reduced-motion: reduce) {
  .cf *, .cf *::before, .cf *::after { transition-duration: .01ms !important; animation-duration: .01ms !important; }
}
```

---

## Appendix B — markup skeletons

These skeletons show the exact structure and class names. Character counters (`63 / 500`) appear only where a max length exists today. **The values in them are sample data** — bind every value, state class (`is-done`, `is-invalid`, …) and text to real state. Write them in the app's own framework: JSX, Vue or Angular templates — for example `class` → `className` in React. `[icon:name size]` = an icon at that px size (section 1, Appendix C).

### B.1 Add ITAM

```html
<form class="cf" novalidate>
  <div class="cf-inner cf-inner--center">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <a href="#">ITAM</a> / <span>New</span></nav>
        <h1 class="cf-title">Add ITAM</h1>
        <p class="cf-sub">Register a new ITAM entry for application onboarding.</p>
      </div>
      <div class="cf-head-right"></div>
    </header>
    <section class="cf-sec">
      <div class="cf-sec-head">
        <div class="cf-sec-titles">
          <h2 class="cf-sec-title">ITAM details</h2>
          <p class="cf-sec-desc">All five fields are needed to onboard an application under this ITAM.</p>
        </div>
      </div>
      <div class="cf-sec-body">
        <div class="cf-grid">
          <div class="cf-field cf-span-7">
            <label class="cf-label" for="itam-name">ITAM name<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input" id="itam-name" type="text" value="RATAN Entitlement Rules" required>
          </div>
          <div class="cf-field cf-span-5">
            <label class="cf-label" for="itam-id">ITAM ID<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input cf-input--mono" id="itam-id" type="text" value="51358" required aria-describedby="itam-id-help">
            <span class="cf-help" id="itam-id-help">The ID from the ITAM register.</span>
          </div>
          <div class="cf-field cf-span-7">
            <label class="cf-label" for="itam-snow">SNOW group<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input cf-input--mono" id="itam-snow" type="text" value="FM-CES-L2-SUPPORT" required aria-describedby="itam-snow-help">
            <span class="cf-help" id="itam-snow-help">The ServiceNow assignment group that supports this application.</span>
          </div>
          <div class="cf-field cf-span-5">
            <label class="cf-label" for="itam-dl">Support DL<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input cf-input--mono is-invalid" id="itam-dl" type="text" placeholder="e.g. DL-FMCES-L2-SUPPORT" required aria-invalid="true" aria-describedby="itam-dl-err">
            <span class="cf-error" id="itam-dl-err">[icon:alert-triangle 13]Enter the support distribution list</span>
          </div>
          <div class="cf-field cf-span-12">
            <label class="cf-label" for="itam-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-count">63 / 500</span></label>
            <textarea class="cf-input" id="itam-desc" rows="3" required>Entitlement rule engine for the RATAN markets operations desks.</textarea>
          </div>
        </div>
      </div>
    </section>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner cf-bar-inner--center">
      <div class="cf-bar-meta"><span><span class="cf-req">*</span> Required</span><span>·</span><span class="is-error">1 field needs attention</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">Cancel</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
    </div>
  </footer>
</form>
```

### B.2 Add Application

`[Existing dependent field]` = today's second dropdown, with its real label.

```html
<form class="cf" novalidate>
  <div class="cf-inner">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <span>New</span></nav>
        <h1 class="cf-title">Add application</h1>
        <p class="cf-sub">Fill in the application details below.</p>
      </div>
      <div class="cf-head-right"><span class="cf-badge cf-badge--draft">Draft</span></div>
    </header>
    <div class="cf-ledger">
      <div class="cf-stack">

        <section class="cf-sec" id="sec-1" aria-labelledby="sec-1-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-1-t">ITAM &amp; ownership</h2><p class="cf-sec-desc">Pick the ITAM first; its details fill in automatically.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-grid">
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-itam">ITAM ID<span class="cf-req" aria-hidden="true">*</span></label>
                <div class="cf-combo">
                  <div class="cf-control">[icon:server 15]<input id="app-itam" class="cf-mono" type="text" value="51358" role="combobox" aria-expanded="false" aria-autocomplete="list"><span class="cf-control-sub">RATAN Entitlement Rules</span><button type="button" class="cf-control-btn" aria-label="Clear ITAM ID">[icon:x 13]</button>[icon:chevron-down 16]</div>
                </div>
              </div>
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-dep">[Existing dependent field]<span class="cf-req" aria-hidden="true">*</span></label>
                <span class="cf-select"><select class="cf-input" id="app-dep"><option>RATAN Entitlement Rules</option></select>[icon:chevron-down 16]</span>
              </div>
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-name">Application name<span class="cf-req" aria-hidden="true">*</span></label>
                <input class="cf-input cf-input--mono" id="app-name" type="text" value="RATAN_ENTITLEMENT_RULE" required>
              </div>
              <div class="cf-field cf-span-12">
                <span class="cf-label">Filled from ITAM 51358</span>
                <div class="cf-autofill">
                  <div><span class="cf-autofill-k">ITAM name</span><span class="cf-autofill-v">RATAN Entitlement Rules</span></div>
                  <div><span class="cf-autofill-k">SNOW group</span><span class="cf-autofill-v">FM-CES-L2-SUPPORT</span></div>
                  <div><span class="cf-autofill-k">Support DL</span><span class="cf-autofill-v">DL-FMCES-L2-SUPPORT</span></div>
                </div>
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-owner">App owner<span class="cf-req" aria-hidden="true">*</span></label>
                <div class="cf-control">[icon:user 15]<input id="app-owner" class="cf-mono" type="text" value="2023504"><span class="cf-control-sub">Bank ID</span>[icon:chevron-down 16]</div>
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-dl">Application dev DL<span class="cf-req" aria-hidden="true">*</span></label>
                <input class="cf-input cf-input--mono" id="app-dl" type="text" value="DL-RATAN-ENT-DEV" required>
              </div>
              <div class="cf-field cf-span-12">
                <label class="cf-label" for="app-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-count">98 / 500</span></label>
                <textarea class="cf-input" id="app-desc" rows="2" required>Rule engine that maps RATAN business roles to FMCES entitlements for the markets operations desks.</textarea>
              </div>
            </div>
          </div>
        </section>

        <section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num" aria-hidden="true">2</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Change &amp; onboarding</h2><p class="cf-sec-desc">Change request and go-live details.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-grid">
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-cr">CR number<span class="cf-opt">Optional</span></label>
                <input class="cf-input cf-input--mono" id="app-cr" type="text" placeholder="e.g. CR-2026-004817">
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-date">FMCES onboard date<span class="cf-opt">Optional</span></label>
                <div class="cf-control">[icon:calendar 15]<input id="app-date" type="date" value="2026-09-24"></div>
              </div>
            </div>
          </div>
        </section>

        <section class="cf-sec" id="sec-3" aria-labelledby="sec-3-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num" aria-hidden="true">3</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-3-t">OLA document</h2><p class="cf-sec-desc">The signed operating-level agreement for this application.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-upload">
              <label class="cf-drop">
                <input type="file" accept=".pdf,.doc,.docx" aria-label="Upload OLA document">
                <span class="cf-drop-icon">[icon:upload 18]</span>
                <span class="cf-file-text"><span class="cf-drop-title">Drop the OLA here, or <span>browse</span></span><span class="cf-drop-hint">PDF or Word document, up to 10 MB</span></span>
              </label>
              <div class="cf-file">
                <span class="cf-file-icon">[icon:file-text 15]</span>
                <span class="cf-file-text"><span class="cf-file-name">RATAN_ENTITLEMENT_RULE_OLA_v2.pdf</span><span class="cf-file-meta">1.2 MB · uploading 45%</span><span class="cf-file-bar"><i style="width: 45%"></i></span></span>
                <button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Remove file">[icon:trash-2 15]</button>
              </div>
            </div>
          </div>
        </section>

      </div>
      <aside class="cf-rail" aria-label="Form progress">
        <div class="cf-rail-card">
          <div class="cf-rail-row"><span class="cf-rail-title">Progress</span><span class="cf-rail-count">6 of 7 required</span></div>
          <div class="cf-progress" role="progressbar" aria-label="Required fields complete" aria-valuemin="0" aria-valuemax="7" aria-valuenow="6"><i style="width: 86%"></i></div>
          <nav class="cf-toc" aria-label="Sections">
            <button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">ITAM &amp; ownership</span><span class="cf-toc-meta">Done</span></button>
            <button type="button" class="cf-toc-item"><span class="cf-toc-dot">2</span><span class="cf-toc-name">Change &amp; onboarding</span><span class="cf-toc-meta">Optional</span></button>
            <button type="button" class="cf-toc-item is-current"><span class="cf-toc-dot">3</span><span class="cf-toc-name">OLA document</span><span class="cf-toc-meta">Uploading…</span></button>
          </nav>
        </div>
        <div class="cf-rail-card">
          <span class="cf-rail-title">After you submit</span>
          <div class="cf-life">
          <div class="cf-life-step is-now"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Draft<small>You can keep editing</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Pending approval<small>A checker reviews it</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Approved<small>Locked as V1</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Active<small>Ready to use</small></span></div>
          </div>
        </div>
      </aside>
    </div>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner">
      <div class="cf-bar-meta"><span class="cf-saved">[icon:check-circle 15]Draft saved · 2 min ago</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 6 of 7 required</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Reset</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
    </div>
  </footer>
</form>
```

### B.3 Add Features (Add Actions is the same with the words in section 6)

```html
<form class="cf" novalidate>
  <div class="cf-inner">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <a href="#">Features</a> / <span>New</span></nav>
        <h1 class="cf-title">Add features</h1>
        <p class="cf-sub">Define one or more features for an existing application.</p>
      </div>
      <div class="cf-head-right"></div>
    </header>
    <div class="cf-stack">

      <section class="cf-sec">
        <div class="cf-sec-body">
          <div class="cf-grid">
            <div class="cf-field cf-span-6">
              <label class="cf-label" for="bulk-app">Application<span class="cf-req" aria-hidden="true">*</span></label>
              <div class="cf-combo">
                <div class="cf-control">[icon:app-window 15]<input id="bulk-app" class="cf-mono" type="text" value="RATAN_ENTITLEMENT_RULE" role="combobox" aria-expanded="false" aria-autocomplete="list"><span class="cf-control-sub">ITAM 51358</span><button type="button" class="cf-control-btn" aria-label="Clear application">[icon:x 13]</button>[icon:chevron-down 16]</div>
              </div>
              <span class="cf-help">Every row below is added to this application.</span>
              <label class="cf-switch"><input type="checkbox"><span class="cf-switch-track"></span>Use a different application per row</label>
            </div>
            <div class="cf-field cf-span-6">
              <span class="cf-label">Already in this application<span class="cf-count">5</span></span>
              <div class="cf-chips"><span class="cf-chip cf-chip--mono">Feature</span><span class="cf-chip cf-chip--mono">Action</span><span class="cf-chip cf-chip--mono">Entitlement</span><span class="cf-chip cf-chip--mono">Application</span><span class="cf-chip cf-chip--mono">Basic_Feature</span></div>
            </div>
          </div>
        </div>
      </section>

      <section class="cf-card">
        <div class="cf-card-head">
          <h2 class="cf-card-title">Features to add</h2>
          <span class="cf-card-meta">4 rows · <span class="is-error">2 need attention</span></span>
          <div class="cf-card-tools">
            <button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add row</button>
            <button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:clipboard 14]Paste from Excel</button>
            <button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:upload 14]Import CSV</button>
          </div>
        </div>
        <div class="cf-bulk-scroll">
          <div class="cf-bulk" role="grid" aria-label="Features to add">
            <div class="cf-bulk-row cf-bulk-head" role="row">
              <span role="columnheader" style="justify-content: center">#</span>
              <span role="columnheader">Feature name <span class="cf-req">*</span></span>
              <span role="columnheader">Description <span class="cf-req">*</span></span>
              <span role="columnheader">Check</span>
              <span role="columnheader"><span class="cf-hidden">Delete</span></span>
            </div>
            <div class="cf-bulk-row" role="row">
              <span class="cf-bulk-num" role="rowheader">1</span>
              <span class="cf-bulk-cell" role="gridcell"><input class="cf-mono" type="text" value="Trade_Capture" aria-label="Feature name, row 1"></span>
              <span class="cf-bulk-cell" role="gridcell"><input type="text" value="Capture and amend trades for the RATAN desks." placeholder="Describe what this feature covers" aria-label="Description, row 1"></span>
              <span class="cf-bulk-status is-ok" role="gridcell">[icon:check-circle 14]<span>Ready</span></span>
              <span class="cf-bulk-del" role="gridcell"><button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Delete row 1">[icon:trash-2 15]</button></span>
            </div>
            <div class="cf-bulk-row" role="row">
              <span class="cf-bulk-num" role="rowheader">2</span>
              <span class="cf-bulk-cell" role="gridcell"><input class="cf-mono" type="text" value="Position_Keeping" aria-label="Feature name, row 2"></span>
              <span class="cf-bulk-cell" role="gridcell"><input type="text" value="View and maintain end-of-day positions." placeholder="Describe what this feature covers" aria-label="Description, row 2"></span>
              <span class="cf-bulk-status is-ok" role="gridcell">[icon:check-circle 14]<span>Ready</span></span>
              <span class="cf-bulk-del" role="gridcell"><button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Delete row 2">[icon:trash-2 15]</button></span>
            </div>
            <div class="cf-bulk-row" role="row">
              <span class="cf-bulk-num" role="rowheader">3</span>
              <span class="cf-bulk-cell is-invalid" role="gridcell"><input class="cf-mono" type="text" value="Basic_Feature" aria-label="Feature name, row 3" aria-invalid="true"></span>
              <span class="cf-bulk-cell" role="gridcell"><input type="text" value="Basic access to the application." placeholder="Describe what this feature covers" aria-label="Description, row 3"></span>
              <span class="cf-bulk-status is-error" role="gridcell">[icon:alert-triangle 14]<span>Name already exists</span></span>
              <span class="cf-bulk-del" role="gridcell"><button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Delete row 3">[icon:trash-2 15]</button></span>
            </div>
            <div class="cf-bulk-row" role="row">
              <span class="cf-bulk-num" role="rowheader">4</span>
              <span class="cf-bulk-cell" role="gridcell"><input class="cf-mono" type="text" value="Risk_Limits_Monitoring" aria-label="Feature name, row 4"></span>
              <span class="cf-bulk-cell is-invalid" role="gridcell"><input type="text" value="" placeholder="Describe what this feature covers" aria-label="Description, row 4" aria-invalid="true"></span>
              <span class="cf-bulk-status is-error" role="gridcell">[icon:alert-triangle 14]<span>Description needed</span></span>
              <span class="cf-bulk-del" role="gridcell"><button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Delete row 4">[icon:trash-2 15]</button></span>
            </div>
            <div class="cf-bulk-row is-new" role="row">
              <span class="cf-bulk-num" role="rowheader">5</span>
              <span class="cf-bulk-cell" role="gridcell"><input class="cf-mono" type="text" placeholder="Type a name…" aria-label="Feature name, new row"></span>
              <span class="cf-bulk-cell" role="gridcell"><input type="text" placeholder="…and a description" aria-label="Description, new row"></span>
              <span class="cf-bulk-status is-new" role="gridcell"><span>New row</span></span>
              <span class="cf-bulk-del" role="gridcell"></span>
            </div>
          </div>
        </div>
        <div class="cf-card-foot">
          <span><span class="cf-kbd">Tab</span>next cell</span>
          <span><span class="cf-kbd">Enter</span>next row</span>
          <span><span class="cf-kbd">Ctrl V</span>paste rows from Excel: name in column 1, description in column 2</span>
        </div>
      </section>

    </div>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner">
      <div class="cf-bar-meta"><span>4 features</span><span>·</span><span class="is-error">2 need attention</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">Cancel</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit 4 features</button>
    </div>
  </footer>
</form>
```

### B.4 Per-row Application cell (only when the per-row switch is on)

The grid root gets `cf-bulk--per-app`; the header gets `<span role="columnheader">Application <span class="cf-req">*</span></span>` after `#`; and each row gets this cell after `.cf-bulk-num`:

```html
<span class="cf-bulk-cell" role="gridcell"><span class="cf-combo"><span class="cf-control"><input class="cf-mono" type="text" value="RATAN_ENTITLEMENT_RULE" role="combobox" aria-expanded="false" aria-label="Application, row 1">[icon:chevron-down 15]</span></span></span>
```

### B.5 Edit ITAM

```html
<form class="cf" novalidate>
  <div class="cf-inner cf-inner--center">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <a href="#">ITAM 51358</a> / <span>Edit</span></nav>
        <h1 class="cf-title">Edit ITAM</h1>
        <p class="cf-sub">Update the details below. Changes are reviewed before they replace the live version.</p>
        <p class="cf-head-meta">ITAM 51358 · Last modified 24 Sep 2026 by 2023504</p>
      </div>
      <div class="cf-head-right"><span class="cf-badge cf-badge--ok">Approved</span><span class="cf-badge cf-badge--version">V1</span></div>
    </header>
    <section class="cf-sec">
      <div class="cf-sec-head">
        <div class="cf-sec-titles">
          <h2 class="cf-sec-title">ITAM details</h2>
          <p class="cf-sec-desc">All five fields are needed to onboard an application under this ITAM.</p>
        </div>
      </div>
      <div class="cf-sec-body">
        <div class="cf-grid">
          <div class="cf-field cf-span-7">
            <label class="cf-label" for="itam-name">ITAM name<span class="cf-req" aria-hidden="true">*</span><span class="cf-edited">Edited</span></label>
            <input class="cf-input is-changed" id="itam-name" type="text" value="RATAN Entitlement Rules – Markets Ops" required>
          </div>
          <div class="cf-field cf-span-5">
                <label class="cf-label" for="itam-id">ITAM ID</label>
                <div class="cf-control is-readonly">[icon:server 15]<input id="itam-id" class="cf-mono" type="text" value="51358" readonly aria-describedby="itam-id-help">[icon:lock 14]</div>
                <span class="cf-help" id="itam-id-help">The ITAM ID can’t be changed.</span>
              </div>
          <div class="cf-field cf-span-7">
            <label class="cf-label" for="itam-snow">SNOW group<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input cf-input--mono" id="itam-snow" type="text" value="FM-CES-L2-SUPPORT" required>
          </div>
          <div class="cf-field cf-span-5">
            <label class="cf-label" for="itam-dl">Support DL<span class="cf-req" aria-hidden="true">*</span></label>
            <input class="cf-input cf-input--mono" id="itam-dl" type="text" value="DL-FMCES-L2-SUPPORT" required>
          </div>
          <div class="cf-field cf-span-12">
            <label class="cf-label" for="itam-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-count">63 / 500</span></label>
            <textarea class="cf-input" id="itam-desc" rows="3" required>Entitlement rule engine for the RATAN markets operations desks.</textarea>
          </div>
        </div>
      </div>
    </section>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner cf-bar-inner--center">
      <div class="cf-bar-meta"><span class="is-changed">1 unsaved change</span><span>·</span><span><span class="cf-req">*</span> Required</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
    </div>
  </footer>
</form>
```

### B.6 Edit Application

```html
<form class="cf" novalidate>
  <div class="cf-inner">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <a href="#">RATAN_ENTITLEMENT_RULE</a> / <span>Edit</span></nav>
        <h1 class="cf-title">Edit application</h1>
        <p class="cf-sub">Update the details below. Changes are reviewed before they replace the live version.</p>
        <p class="cf-head-meta">ITAM 51358 · 5 features · 5 actions · Last modified 24 Sep 2026</p>
      </div>
      <div class="cf-head-right"><span class="cf-badge cf-badge--ok">Approved</span><span class="cf-badge cf-badge--version">V1</span></div>
    </header>
    <div class="cf-ledger">
      <div class="cf-stack">

        <section class="cf-sec" id="sec-1" aria-labelledby="sec-1-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-1-t">ITAM &amp; ownership</h2><p class="cf-sec-desc">Pick the ITAM first; its details fill in automatically.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-grid">
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-itam">ITAM ID</label>
                <div class="cf-control is-readonly">[icon:server 15]<input id="app-itam" class="cf-mono" type="text" value="51358" readonly><span class="cf-control-sub">RATAN Entitlement Rules</span>[icon:lock 14]</div>
              </div>
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-dep">[Existing dependent field]<span class="cf-req" aria-hidden="true">*</span></label>
                <span class="cf-select"><select class="cf-input" id="app-dep"><option>RATAN Entitlement Rules</option></select>[icon:chevron-down 16]</span>
              </div>
              <div class="cf-field cf-span-4">
                <label class="cf-label" for="app-name">Application name</label>
                <div class="cf-control is-readonly">[icon:app-window 15]<input id="app-name" class="cf-mono" type="text" value="RATAN_ENTITLEMENT_RULE" readonly>[icon:lock 14]</div>
              </div>
              <div class="cf-field cf-span-12">
                <span class="cf-label">Filled from ITAM 51358</span>
                <div class="cf-autofill">
                  <div><span class="cf-autofill-k">ITAM name</span><span class="cf-autofill-v">RATAN Entitlement Rules</span></div>
                  <div><span class="cf-autofill-k">SNOW group</span><span class="cf-autofill-v">FM-CES-L2-SUPPORT</span></div>
                  <div><span class="cf-autofill-k">Support DL</span><span class="cf-autofill-v">DL-FMCES-L2-SUPPORT</span></div>
                </div>
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-owner">App owner<span class="cf-req" aria-hidden="true">*</span></label>
                <div class="cf-control">[icon:user 15]<input id="app-owner" class="cf-mono" type="text" value="2023504"><span class="cf-control-sub">Bank ID</span>[icon:chevron-down 16]</div>
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-dl">Application dev DL<span class="cf-req" aria-hidden="true">*</span></label>
                <input class="cf-input cf-input--mono" id="app-dl" type="text" value="DL-RATAN-ENT-DEV" required>
              </div>
              <div class="cf-field cf-span-12">
                <label class="cf-label" for="app-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-edited">Edited</span><span class="cf-count">98 / 500</span></label>
                <textarea class="cf-input is-changed" id="app-desc" rows="2" required>Rule engine that maps RATAN business roles to FMCES entitlements for the markets operations desks.</textarea>
              </div>
            </div>
          </div>
        </section>

        <section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Change &amp; onboarding</h2><p class="cf-sec-desc">Change request and go-live details.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-grid">
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-cr">CR number<span class="cf-opt">Optional</span><span class="cf-edited">Edited</span></label>
                <input class="cf-input cf-input--mono is-changed" id="app-cr" type="text" value="CR-2026-004817">
              </div>
              <div class="cf-field cf-span-6">
                <label class="cf-label" for="app-date">FMCES onboard date<span class="cf-opt">Optional</span></label>
                <div class="cf-control">[icon:calendar 15]<input id="app-date" type="date" value="2026-09-24"></div>
              </div>
            </div>
          </div>
        </section>

        <section class="cf-sec" id="sec-3" aria-labelledby="sec-3-t">
          <div class="cf-sec-head">
            <span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
            <div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-3-t">OLA document</h2><p class="cf-sec-desc">The signed operating-level agreement for this application.</p></div>
          </div>
          <div class="cf-sec-body">
            <div class="cf-upload">
              <label class="cf-drop">
                <input type="file" accept=".pdf,.doc,.docx" aria-label="Replace OLA document">
                <span class="cf-drop-icon">[icon:upload 18]</span>
                <span class="cf-file-text"><span class="cf-drop-title">Drop a new file to replace it, or <span>browse</span></span><span class="cf-drop-hint">PDF or Word document, up to 10 MB</span></span>
              </label>
              <div class="cf-file">
                <span class="cf-file-icon">[icon:file-text 15]</span>
                <span class="cf-file-text"><span class="cf-file-name">RATAN_ENTITLEMENT_RULE_OLA_v2.pdf</span><span class="cf-file-meta">1.2 MB · uploaded 24 Sep 2026</span></span>
                <button type="button" class="cf-icon-btn" aria-label="Preview file">[icon:eye 15]</button>
                <button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Remove file">[icon:trash-2 15]</button>
              </div>
            </div>
          </div>
        </section>

      </div>
      <aside class="cf-rail" aria-label="Form progress">
        <div class="cf-rail-card">
          <div class="cf-rail-row"><span class="cf-rail-title">Progress</span><span class="cf-rail-count">7 of 7 required</span></div>
          <div class="cf-progress" role="progressbar" aria-label="Required fields complete" aria-valuemin="0" aria-valuemax="7" aria-valuenow="7"><i style="width: 100%"></i></div>
          <nav class="cf-toc" aria-label="Sections">
            <button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">ITAM &amp; ownership</span><span class="cf-toc-meta is-changed">1 change</span></button>
            <button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Change &amp; onboarding</span><span class="cf-toc-meta is-changed">1 change</span></button>
            <button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">OLA document</span><span class="cf-toc-meta">1 file</span></button>
          </nav>
        </div>
        <div class="cf-rail-card">
          <span class="cf-rail-title">After you submit</span>
          <div class="cf-life">
          <div class="cf-life-step is-now"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Draft V2<small>You can keep editing</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Pending approval<small>A checker reviews it</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Approved<small>Replaces V1</small></span></div>
          <div class="cf-life-step"><span class="cf-life-node"><i></i><b></b></span><span class="cf-life-text">Active<small>V1 stays live until then</small></span></div>
          </div>
        </div>
      </aside>
    </div>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner">
      <div class="cf-bar-meta"><span class="is-changed">2 unsaved changes</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 7 of 7 required</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
    </div>
  </footer>
</form>
```

### B.7 Edit Feature (Edit Action is the same with the words in section 6)

```html
<form class="cf" novalidate>
  <div class="cf-inner">
    <header class="cf-head">
      <button type="button" class="cf-back" aria-label="Back to App Management">[icon:chevron-left 17]</button>
      <div class="cf-head-text">
        <nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">App Management</a> / <a href="#">Features</a> / <a href="#">Basic_Feature</a> / <span>Edit</span></nav>
        <h1 class="cf-title">Edit feature</h1>
        <p class="cf-sub">Update the details below. Changes are reviewed before they replace the live version.</p>
        <p class="cf-head-meta">RATAN_ENTITLEMENT_RULE · Basic_Feature · Last modified 24 Sep 2026</p>
      </div>
      <div class="cf-head-right"><span class="cf-badge cf-badge--ok">Approved</span><span class="cf-badge cf-badge--version">V1</span></div>
    </header>
    <div class="cf-stack">

      <section class="cf-sec">
        <div class="cf-sec-body">
          <div class="cf-grid">
            <div class="cf-field cf-span-6">
                <label class="cf-label" for="bulk-app">Application</label>
                <div class="cf-control is-readonly">[icon:app-window 15]<input id="bulk-app" class="cf-mono" type="text" value="RATAN_ENTITLEMENT_RULE" readonly aria-describedby="bulk-app-help"><span class="cf-control-sub">ITAM 51358</span>[icon:lock 14]</div>
                <span class="cf-help" id="bulk-app-help">A feature can’t move to another application.</span>
              </div>
            <div class="cf-field cf-span-6">
              <span class="cf-label">Other features in this application<span class="cf-count">4</span></span>
              <div class="cf-chips"><span class="cf-chip cf-chip--mono">Feature</span><span class="cf-chip cf-chip--mono">Action</span><span class="cf-chip cf-chip--mono">Entitlement</span><span class="cf-chip cf-chip--mono">Application</span></div>
            </div>
          </div>
        </div>
      </section>

      <section class="cf-card">
        <div class="cf-card-head">
          <h2 class="cf-card-title">Feature</h2>
          <span class="cf-card-meta">1 row · <span class="is-changed">1 change</span></span>
        </div>
        <div class="cf-bulk-scroll">
          <div class="cf-bulk" role="grid" aria-label="Feature">
            <div class="cf-bulk-row cf-bulk-head" role="row">
              <span role="columnheader" style="justify-content: center">#</span>
              <span role="columnheader">Feature name <span class="cf-req">*</span></span>
              <span role="columnheader">Description <span class="cf-req">*</span></span>
              <span role="columnheader">Check</span>
              <span role="columnheader"><span class="cf-hidden">Delete</span></span>
            </div>
            <div class="cf-bulk-row" role="row">
              <span class="cf-bulk-num" role="rowheader">1</span>
              <span class="cf-bulk-cell" role="gridcell"><input class="cf-mono" type="text" value="Basic_Feature" aria-label="Feature name, row 1"></span>
              <span class="cf-bulk-cell is-changed" role="gridcell"><input type="text" value="Basic access to the RATAN entitlement rule screens." aria-label="Description, row 1"></span>
              <span class="cf-bulk-status is-ok" role="gridcell">[icon:check-circle 14]<span>Ready · edited</span></span>
              <span class="cf-bulk-del" role="gridcell"></span>
            </div>
          </div>
        </div>
        <div class="cf-card-foot">
          <span><span class="cf-kbd">Tab</span>next cell</span>
        </div>
      </section>

    </div>
  </div>
  <footer class="cf-bar">
    <div class="cf-bar-inner">
      <div class="cf-bar-meta"><span class="is-changed">1 unsaved change</span><span>·</span><span><span class="cf-req">*</span> Required</span></div>
      <span class="cf-bar-spacer"></span>
      <button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
    </div>
  </footer>
</form>
```

### B.8 Loading placeholders (inside each `.cf-sec-body` while the record loads; one placeholder per field, same spans as the real fields)

```html
<div class="cf-grid" aria-busy="true" aria-label="Loading">
  <div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
  <div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
  <div class="cf-field cf-span-12"><span class="cf-skel cf-skel--label"></span><span class="cf-skel" style="height: 76px"></span></div>
</div>
```

---

## Appendix C — icon paths (24×24, stroke `currentColor`, width 1.75, round caps and joins, no fill)

| Name | Path `d` |
|---|---|
| `chevron-left` | `M15 6l-6 6 6 6` |
| `chevron-down` | `M6 9l6 6 6-6` |
| `x` | `M6 6l12 12M18 6 6 18` |
| `search` | `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4` |
| `calendar` | `M4 6h16v14H4zM4 10h16M8 3v4M16 3v4` |
| `upload` | `M12 16V5M7 10l5-5 5 5M5 20h14` |
| `file-text` | `M6 3h9l4 4v14H6zM14 3v5h5M9 13h7M9 17h7` |
| `eye` | `M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12zM12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6z` |
| `trash-2` | `M4 7h16M9 7V4h6v3M6 7l1 13h10l1-13` |
| `check` | `M5 12.5l4.5 4.5L19 7.5` |
| `check-circle` | `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM8 12l3 3 5-6` |
| `alert-triangle` | `M12 4 2.8 20h18.4zM12 10v4.5M12 17.2h.01` |
| `info` | `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM12 11v6M12 7.5h.01` |
| `rotate-ccw` | `M9 14 4 9l5-5M4 9h10a6 6 0 0 1 0 12h-3` |
| `save` | `M5 4h11l3 3v13H5zM8 4v5h7V4M8 20v-6h8v6` |
| `send` | `M4 12 20 4l-6 16-3-7z` |
| `plus` | `M12 5v14M5 12h14` |
| `clipboard` | `M9 4h6v3H9zM8 5H5v16h14V5h-3M9 12h6M9 16h6` |
| `user` | `M12 12a4 4 0 1 0 0-8 4 4 0 0 0 0 8zM4 21a8 8 0 0 1 16 0` |
| `server` | `M4 4h16v6H4zM4 14h16v6H4zM8 7h.01M8 17h.01` |
| `app-window` | `M3 5h18v14H3zM3 9h18M6.5 7h.01M9 7h.01` |
| `lock` | `M6 11h12v10H6zM8.5 11V7.5a3.5 3.5 0 0 1 7 0V11` |

Inline form: `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="…"/></svg>` (use `stroke-width="2.25"` where the skeleton says `bold`).
