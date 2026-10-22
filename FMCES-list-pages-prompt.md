# FMCES: restyle the list and report pages (styling only, keep every feature)

You are working in the FMCES codebase. FMCES is an internal "Markets Operations One" web app with a dark navy theme and a teal accent. I have already restyled the Home dashboard and the record drawers. Now I want the **list and report pages** restyled to match.

**This is a styling job.** Every feature, column, button, field, label, handler, API call and permission check that exists today must still exist and work the same way afterwards. You are changing how things look and move, not what they do. If a change in this prompt would remove or alter behaviour, stop and ask me first.

I am not sending screenshots. This prompt is the complete specification:
- The stylesheet is in **Appendix A**. Copy it verbatim.
- The markup for every new visual piece is in **Appendix B**.
- The icons are in **Appendix C**.
- Everything else (rules, mapping onto the existing code, wording, behaviour, motion) is described below.

Use the numbers and class names given here exactly. Do not adjust them by eye.

---

## Pages in scope

The left sidebar of the app is called **the dock**. These are the pages to restyle, in dock order:

| # | Page (dock name stays exactly as it is today) | What is on it |
|---|---|---|
| 1 | Application Management | Tabs: ITAM records (only for some roles), Applications, Features, Actions. Features and Actions need an application to be chosen first ("Select Application"). |
| 2 | Data Profiles | One list. |
| 3 | Data Policy Management | One list, plus Upload (Excel). |
| 4 | Entitlement Management | One list. |
| 5 | Entitlement Users | Finder (ITAM Name → Application, or a quick search across all ITAMs), then Accounts / Entitlements tabs. |
| 6 | Reports | Choose an application and options, Generate Preview, Download Report. |
| 7 | **FMID Reports** | Search parameters (ITAM ID, Application, Bank IDs), Fetch Report, Download Security Matrix, results in a Detail view and a Matrix view. |

Important about page 7:
- The page is **FMID Reports**. In my design notes I sometimes called it "Entitlement Report". That was my mistake. **Do not rename anything to "Entitlement Report".**
- The dock name, the route and the page title stay exactly what they are today.

The names in the table are how I refer to the pages. If any of them differs from what the dock or the page heading says today, the app wins: keep the app's text, and tell me about the difference.

---

## How we work together (read this first)

**You may ask me questions at any time. I want you to.** If you are not sure whether something is possible, how my data maps to a field, or whether a change would affect behaviour, stop and ask. Don't guess. I would rather answer ten questions than lose a feature.

**Before you write any code, send me a Step 0 report and wait for my reply.** It has four parts.

**Part 1: map of the current code.** For each of the seven pages:
- The page component and its route.
- The table component it uses. Is it a native `<table>`, a div grid you wrote, or a library (AG Grid, MUI DataGrid, TanStack, PrimeNG, Angular Material…)? Name it and its version.
- How column **resize**, column **drag / reorder**, column **show / hide**, **sort**, **pin**, **pagination** and the **row click that opens the drawer** are implemented, and where their state lives.
- How **Presets** are stored and applied (save, page defaults, star the default, delete, the toasts).
- Whether each list is paged on the server or loaded fully into the browser.
- Where the "Updated …" text and the refresh action live today.
- The framework and styling approach (React, Angular, Vue; CSS modules, SCSS, Tailwind…), and any global CSS (UI library theme, resets, `button` / `table` / `input` styles) that could bleed into the new classes.

**Part 2: feature inventory (this becomes your checklist).** For each page, list **every** control, column, label, button, menu item, tooltip, empty message and state that exists today, exactly as worded in the app. Include the column list of every table, in order, with its header text, including the FMID Reports **Matrix view** (every column). At the end of the job, every line of this inventory must still be present and working.

**Part 3: feasibility table.** For each new visual element below, give one status:
- ✅ **possible now** with data the page already has;
- ⚠️ **possible, but needs something** (say exactly what);
- ❌ **not possible**, and why.

Cover at least:
- facts row counts and the mix bar (section 6.1);
- the search match count (6.2);
- the on-screen column chips replacing the column menu (6.3);
- the table adapter on your table component, including the resize handle and drag styles (6.4);
- relative dates under dates (6.5);
- row reorder animation (FLIP) (13.3);
- the dock glide (12);
- the FMID Reports fetching stages and progress bar (10.4).

**Part 4: questions.** Everything you are unsure about, numbered, so I can answer in one go. Always include your answer or proposal for these:
1. **FMID Reports Matrix view:** list every column it shows today, in order. Confirm you will keep all of them.
2. **Edit pencil:** today it sits in its own column. My default is to keep it there, always visible, restyled (6.5). Confirm, or tell me if you see a reason to do otherwise.
3. **"Updated …" text:** where it is today, and confirm the header refresh line (3.4) will reuse the same text and the same refresh function.
4. **Facts counts:** if a list is server-paged, the counts per status need the totals. Does the API return them? If not, say so; do not count only the visible page.
5. **Module numbers** (the `01`–`07` in the header eyebrow, 3.2): confirm the dock order gives the numbers I listed.
6. **Column menu:** what it can do today (show/hide, reorder, required columns that cannot be hidden, reset to default).
7. **Reports page:** which settings the page has today (application, fields, format, others?), so we can map them onto the step layout (9).
8. **FMID Reports fetching:** does the app know real progress or stages while fetching? If not, we use the simple version (10.4).
9. **Status values:** list every status value each list can show (for example `APPROVED`, `DRAFT`, `PENDING_APPROVAL`, `INACTIVE`…), so I can confirm the colour mapping (6.5).

Do not build anything marked ⚠️ or ❌ until I answer. Build ✅ items only after I reply to Step 0.

**While you build:**
- Work in the phases listed in section 17.
- After each phase, send a short report:
  - what you built;
  - the files you touched;
  - how I can test it;
  - the inventory lines you re-checked;
  - anything that didn't match this spec;
  - new questions.
- If you hit something unclear mid-phase, stop and ask. Do not quietly skip a feature, remove a control, or change behaviour.

**Never invent data.**
- **[IF DATA EXISTS]** in this prompt means: build it only if the page already has the data, or an existing API returns it. Otherwise follow the fallback written next to it, and tell me.
- My mockup used sample data: application names like REPORTING_LAKE_BA, SAWFL and AppTest; people names; Bank IDs; counts like "17 users"; my own column sets. **None of that may appear in the app.** Every value shown comes from the app's real data.
- Appendix B contains mockup wording in places, such as placeholders and helper sentences. Where the app already has its own text for the same thing, **the app's text wins**. Section 0.3 lists the labels I know.

---

## 0. Rules

### 0.1 Styling only

You may:
- add classes to existing elements;
- wrap existing elements in new styled containers;
- replace an element's visual content, for example turning a plain status word into a dot plus the word;
- add the purely visual elements described here (facts row, mix bar, eyebrow, watermark number, glass tab indicator, skeletons, empty-state art, toast check and countdown bar, dock glide);
- add small view-only state for animations, such as a flag that flips to replay an entrance.

You may not:
- change routes, API calls, payloads, query parameters, auth or role checks;
- change existing handlers' logic: resize, drag, sort, pin, pagination, presets, export, upload, fetch, download, reset, clear, the role gating of ITAM, or row click opening the drawer;
- remove or rename any column, field, button, menu item, tab, filter, tooltip, label or message;
- change default sort, default page size, default visible columns, or column order;
- change ids, names or `data-testid`s that tests rely on.

Do not add endpoints or change backend code without asking me.

### 0.2 Keep everything (non-exhaustive; your Part 2 inventory is the full list)

**Every list table:**
- column **resize** (drag the header edge);
- column **drag / reorder**;
- column **show / hide**;
- the **pin** column and pinned rows;
- the **`#`** (row number) column;
- the **edit pencil**;
- **Active** and **Status** columns;
- **OLA**, **CII Number** and **Onboard Date** columns where they exist;
- the **filter (funnel)** button and whatever it opens;
- **sorting** by header click;
- **pagination** with **Rows 15 / 25 / 50**;
- the **"Updated just now"** text;
- **row click opens the existing drawer**. The drawers were restyled in an earlier prompt. Do not touch them.

**Header actions:**
- **Presets**: save, page defaults, star a default, delete, and the toasts they show today;
- **Export** (the menu shows "N rows · N cols", then CSV and Excel);
- **Upload** (Excel modal) on Data Policy Management;
- each page's create button (see 0.3).

**Application Management:**
- the four tabs;
- the **New** dropdown. Its ITAM item and the **ITAM records tab** stay visible **only to the roles that see them today**. Reuse the existing role check unchanged. **Do not show any text such as "restricted" or "role required" anywhere.** Users without the role simply don't see those items.
- Features / Actions **"Select Application"**.

**Entitlement Users:**
- ITAM Name and Application selects;
- the quick search across ITAMs;
- the Accounts / Entitlements tabs;
- the empty message;
- **Create Account**.

**Reports:**
- "Loading applications…";
- the "N accounts loaded · Generate Preview" bar;
- **Generate Preview** and **Download Report**;
- the results table with its search;
- "Showing 10 of 20";
- **Show 10 more** / **Show all**.

**FMID Reports:**
- required-field asterisks;
- the "All users … one request" hint;
- **Fetch Report**, **Reset**, **Clear all**;
- the security matrix format select and **Download Security Matrix**;
- the fetching state;
- the result chips;
- **Export CSV / Excel / JSON**;
- the **Detail / Matrix** toggle;
- the filters **with their info tooltips**;
- search across all columns;
- the counts line (for example "17 users · 16 cols · 0 FMIDs");
- the **COLUMNS** group toggles;
- the grouped Detail table;
- the **Matrix view with ALL of its existing columns**. In my mockup the matrix had fewer columns. **That was wrong. Never drop, merge or rename a Matrix column.** Style it only. If unsure, ask.

### 0.3 Names and words

- **Dock:** do not change any dock item's name, icon, order or link. The only dock change is the gliding highlight (section 12).
- **Page titles** stay as they are in the app today. The ones I know:
  - "Application Management";
  - "Data Profiles";
  - "Data Policy Management";
  - "Entitlement Management";
  - "Entitlement Users";
  - "Reports";
  - FMID Reports keeps its current heading. In my notes it reads "Report"; check the code.
- **Subtitles** under the titles: keep the app's text.
- **Button labels:** keep the app's exact text. The ones I know:
  - Presets
  - Add New
  - New
  - Upload
  - New Policy
  - Create Account
  - Generate Preview
  - Download Report
  - Fetch Report
  - Reset
  - Clear all
  - Download Security Matrix
- **Column headers, status words, filter names, tooltips, empty messages, toast messages:** keep the app's text. You may change letter case only through CSS. The header style uppercases headers with `text-transform`; the text in the code stays the same.

### 0.4 Do not touch

- The app header (top bar).
- The workspace tabs.
- The dock items (only add the glide, section 12).
- The record drawers.
- The New / Edit forms and pages.
- The Home dashboard.

### 0.5 CSS

- Copy **Appendix A verbatim** into one new stylesheet, `fmces-pages.css`, and load it globally after your other styles.
- Everything is scoped: every class starts with `lp-`, and the design tokens live on `.lp-root`.
- Do not translate it to Tailwind, CSS modules or inline styles. Do not rename classes. Do not edit values. If something looks wrong, ask.
- The only inline styles allowed are:
  - the ones written in Appendix B (custom properties such as `--i`, `--d`, `--k`, `--c`, `--w`, `--n`, `--p`, `--cols`, `--minw`, `--gc`, the `transform` on sliding indicators, `width` on skeleton lines, `grid-column` on group headers, and the few one-off layout styles shown there);
  - your table library's own inline widths, which it needs for resizing. Leave those alone.
- Section 12 has a small extra CSS block for the dock. It lives outside `.lp-root`, so put it in the same file below Appendix A.
- If your UI library injects styles that fight these, for example a theme that sets table row heights or button fonts, do not edit Appendix A. Tell me, and we will add a scoped override.

### 0.6 The one replacement I asked for: the column menu becomes on-screen chips

Today the columns are chosen from a dropdown menu. I asked for the columns to be **always visible as chips** above the table (section 6.3). This is the **only** control that changes form, and only its form:
- The chips use **the same state and the same handlers** as the menu.
- Everything the menu can do must still be possible:
  - show / hide;
  - reorder by drag, if the menu supports it today;
  - columns that cannot be hidden stay locked;
  - reset to the default columns, if the menu has it today.
- Remove the dropdown trigger only after the chips do everything the menu did. If the menu does something the chips can't, keep the menu and ask me.

### 0.7 Fonts

The design needs Manrope weights 200–800 and JetBrains Mono. If the existing link lacks any weight, replace it with:
`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@200;300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

---

## 1. How to read Appendix B

Appendix B is framework-neutral HTML. Translate it to your template syntax:

| In Appendix B | Means |
|---|---|
| `{name}` | A value you compute or already have. The sections below explain each one. |
| `<!-- if cond -->…<!-- /if -->` | Render only when `cond` is true. |
| `<!-- for each x in list -->…<!-- /for -->` | Repeat for each item. |
| `<!-- TABLE ROW (B.5), once for each r in list -->` | Render the B.5 row content for each item. |
| `onClick={fn}`, `onChange={fn}`, `aria-x={v}` | Bind the handler or attribute. Always bind **your existing handler** for that action. |
| `class="lp-a {lp-b [lp-c]\|lp-d}"` | Static classes, plus a computed part in braces. `[x]` is an optional modifier. `a\|b` means one of the alternatives. |
| `<Icon name="search" size="15" bold class="lp-x"/>` | Your small Icon component (below). |
| `ref={srRef}` | Keep a reference to the element. Used for focus, for example `/` focuses the search. |

Appendix B shows **where things go and which classes they carry**. Its element structure is a guide, not a requirement. For example, if your Presets popover is a library popover component, keep the component and put the classes on its parts.

**Icon component:**

```html
<svg class="lp-i [lp-b if bold] [extra classes]" width="{size}" height="{size}" viewBox="0 0 24 24" aria-hidden="true"><path d="{path from Appendix C}"></path></svg>
```

- Stroke icons only. The CSS sets fill none, stroke `currentColor`, stroke width 1.75 (2.25 when bold).
- If the app already uses an icon set, keep it for existing icons where you prefer. Use the Appendix C paths for the new elements, so they match the design.

---

## 2. Colours, type, motion

All colours are tokens on `.lp-root` (Appendix A, first rule). They are the same tokens as the drawers, plus a few for rows and popovers. Use only these.

| Token | Value | Use |
|---|---|---|
| `--bg` | `#0F1A22` | Page background |
| `--surface` | `#111E27` | Inputs, segmented controls |
| `--card` | `#152733` | Cards |
| `--card2` | `#1B3140` | Raised fills, active segment, dock highlight |
| `--border` | `#23394A` | Card borders |
| `--hair` | `#1E3240` | Hairline dividers, header underline |
| `--btn` | `#2B4556` | Button and input borders |
| `--text` | `#E6EEF2` | Main text |
| `--text2` | `#A7B8C4` | Secondary text, table cells |
| `--muted` | `#86A0AE` | Labels, table headers, meta |
| `--faint` | `#6F8896` | Placeholders, empty values, row numbers |
| `--accent` | `#2DD4BF` | Teal: primary button, Approved/Active dots, checkmarks |
| `--accent2` | `#5EEAD4` | Light teal: links, active sort header, numbers in the eyebrow |
| `--on` | `#062A26` | Text on teal buttons |
| `--draft` | `#F6C177` | Amber: Draft |
| `--pend` | `#C9B6FF` | Lavender: Pending approval |
| `--err` | `#F4A3A3` | Errors, required asterisks |
| `--blue` | `#93C5FD` | Informational (OLA button, service accounts, security matrix card) |
| `--pop` / `--pop-b` | `#172A36` / `#2E4A5C` | Popover fill / border |
| `--row-h` / `--row-hs` / `--row-on` | `rgba(23,43,56,.85)` / `#162A37` / `#183140` | Row hover (pinned), row hover, selected row |
| `--mg-bg` / `--mg-b` | gradient / `#2C4C5D` | Monogram tile fill / border |
| `--tint` / `--tint2` | teal at 10% / 16% | Selected chips, active filters |
| `--ease` | `cubic-bezier(.22,1,.36,1)` | Entrances |
| `--spring` | `cubic-bezier(.34,1.36,.64,1)` | Indicators, presses, pops |

**Fonts:**
- Manrope for everything.
- JetBrains Mono (`lp-mono`) for:
  - IDs, Bank IDs, FMIDs;
  - account / service names;
  - version numbers;
  - counts in pills;
  - the `01`–`07` module numbers;
  - the rows-per-page buttons and page numbers.

**Key sizes** (the CSS has them all):

| Element | Size / weight |
|---|---|
| Page title | 36px / 300, letter-spacing −0.028em |
| Watermark number behind the title | 150px / 200 at 3.5% opacity |
| Eyebrow (`01 — FMCES`) | 11px / 700 uppercase |
| Facts numbers | 30px / 300 |
| Table header | 11px / 700 uppercase, letter-spacing .065em, row 42px |
| Table row | 54px high, cells 13px |
| Buttons | 38px high, radius 10px (small: 32px / 9px) |
| Cards | radius 18px |
| Popovers | 300px wide, radius 14px |

**Motion:** everything is in the CSS, apart from the few JavaScript hooks in section 13. Everything respects `prefers-reduced-motion` (last rule of Appendix A).

---

## 3. Page wrapper and page header (B.1, B.2)

### 3.1 Wrapper (B.1)

- Wrap the content area of each of the seven pages in `<div class="lp-root"><div class="lp-pg">…</div></div>`. That is everything to the right of the dock and below the workspace tabs.
- `.lp-pg` adds the page padding (`30px 44px 72px 14px`). If your layout container already pads the content, remove one of the two so the content is not double-padded, and tell me which you removed.
- Inside `.lp-pg`:
  - the soft background glow `.lp-aur` (two blurred teal / blue blobs that drift slowly);
  - then a content wrapper with class `lp-pga`. Its direct blocks carrying `lp-rz` rise in one after another; `--d` is the order.
- **Replaying the entrance:** if your router reuses the same component when moving between these pages, swap `lp-pga` ↔ `lp-pgb` on each page change. If each page mounts fresh, `lp-pga` alone is enough.
- **Popover scrim:** while any popover from this prompt is open, render the invisible full-screen `.lp-pscrim` button that closes it on click. If your popovers already close on outside click, skip it.

### 3.2 Header (B.2)

- **`.lp-bign`**: big faint watermark of the module number behind the title (aria-hidden).
- **`.lp-eb` eyebrow:** module number in mono teal, a short line, then `FMCES`.
  - Module numbers follow dock order: Application Management `01`, Data Profiles `02`, Data Policy Management `03`, Entitlement Management `04`, Entitlement Users `05`, Reports `06`, FMID Reports `07`.
  - Confirm in Step 0.
- **`.lp-pt` title:** the app's page title (0.3).
- **`.lp-ps` subtitle line:** the app's subtitle, plus the refresh line (3.4) on list pages.
- **`.lp-pa` actions**, right-aligned, in this order:
  1. Presets (3.3);
  2. Upload, only on Data Policy Management;
  3. the create button (`lp-btn lp-btn-p`, with the plus icon) and its existing label and handler.

  On Application Management, the create button is the **New** dropdown:
  - Use the `.lp-nwm` menu with one `.lp-nwi` row per existing item.
  - Each row has an icon, a bold title and a small description. The description is optional: use the app's text if it has one, otherwise omit the `<small>`.
  - Use the existing menu items, wording, order and handlers.
  - The ITAM item is behind the **existing role check**, with a separator above it.
  - The row for the currently open tab gets `lp-on`.
  - Icons: applications `grid`, features `box`, actions `zap`, ITAM `database`.
- Pages without a create button today don't get one.

### 3.3 Presets popover (B.2)

The **Presets** button (`lp-btn lp-btn-g`):
- Shows the layers icon, the word "Presets", then `.lp-sub` with the name of the preset currently applied, then `.lp-bdg` with the number of saved presets.
- The chevron rotates while the popover is open. The button gets `lp-on` while open.

The popover (`.lp-pop`) contains:
- The heading "Presets".
- `.lp-pin2`: the name input and the Save button.
  - Bind your existing save logic.
  - After a successful save, the button shows a drawn check and "Saved" for 1.5s (`lp-okc` icon, CSS draws it). Then it shows "Save" again.
  - Keep your existing save toast too.
- One `.lp-vw` row per preset, including the page defaults:
  - The name, plus `<small>` meta if you have it (for example "Page default", or the number of columns). Otherwise omit it.
  - The row for the currently applied preset gets `lp-on`.
  - **Star** button (`.lp-vwi`) marks the default, using your existing handler. It gets `lp-on` when this preset is the default.
  - **Delete** button (`.lp-vwi.lp-del`) appears only where deleting is allowed today, with your existing handler and existing confirmation, if any.
- The footer note `.lp-pnote`. Use your own wording if the app has one; otherwise use "The starred preset opens when you come to this page."

Rows stagger in (`--i` = index).

### 3.4 Refresh line (`.lp-sync`)

On the list pages (1–5), the subtitle line ends with a small pill button:
- It shows the **same "Updated …" text the page has today** and calls **the existing refresh**.
- While refreshing, it gets `lp-busy` (its dot turns amber).
- Keep the existing "Updated …" text in its current place too, unless you tell me it is the same element.
- Do not show it on Reports and FMID Reports.

---

## 4. Application Management tabs (B.3)

The four tabs become one **glass segmented control** (`.lp-gtw > .lp-gt`):
- `--n` = number of visible tabs: 4, or 3 when the user cannot see ITAM records.
- One `button[role=tab]` per visible tab, in today's order, with the app's tab names.
  - `<em>` shows the tab's record count **[IF DATA EXISTS]**; otherwise omit the `<em>`.
  - The active tab gets `lp-on` and `aria-selected="true"`.
- The sliding indicator `.lp-ind` gets `style="transform: translateX({index × 100}%)"`, where index is the active tab's position among the **visible** tabs. It glides with a spring.
- Switching tabs uses your existing tab logic and route, if any.
- The tab content slides in from the side it came from (13.2).

The ITAM tab is hidden by the **existing** role check. No placeholder, no disabled tab, no message.

---

## 5. Application band for Features and Actions (in B.4)

On the Features and Actions tabs, the top of the list card is the **application band** (`.lp-scx`). It replaces the look of today's "Select Application" control and keeps its logic.

- **Label:** "Application". Use the app's label if it has one, for example "Select Application".
- **The trigger `.lp-scf`** (50px high):
  - a monogram tile with the app's initials (`.lp-mgx`, plus `lp-dr` if the application is a draft);
  - the application name in bold;
  - a chevron.
  - It gets `lp-on` while open.
- **The popover `.lp-pop.lp-l.lp-scp`:**
  - a search field `.lp-psr`, filtering by application name or ITAM;
  - the group label "Applications";
  - one `.lp-opt` row per application, with the monogram, name, `<small>` meta (for example "ITAM 51515"), a status dot (`.lp-lc` variant, 6.5), and a check on the selected one.
  - Use your existing application list and your existing select handler.
  - If the list is long and you already load it lazily, keep that.
  - If nothing matches, show the `.lp-pnote` "No application matches “…”".
- **Facts to the right of the trigger** (`.lp-scx-m`): `.lp-kv3` pills for App UID, ITAM and Status, **[IF DATA EXISTS]** for the chosen application. Omit any you don't have.
- **`.lp-scx-n`:** one muted line under the band. Use the app's helper text if it has one; otherwise omit it.
- If no application is selected yet and the app shows a message today, keep that message. Show it in the list card's empty state (6.6) instead of the table.

---

## 6. The list card (B.4, B.5, B.6)

One card (`.lp-cd`) holds, top to bottom:
1. the facts row and mix bar, plus the toolbar on the right;
2. the column chips;
3. the table;
4. the footer.

This applies to Application Management (all tabs), Data Profiles, Data Policy Management, Entitlement Management, and the Accounts / Entitlements lists on Entitlement Users.

### 6.1 Facts row and mix bar (`.lp-cd-h`, left side) **[IF DATA EXISTS]**

A row of big numbers that double as quick filters:
- One `.lp-fb` button per fact: the number (`.lp-fb-n`, 30px / 300), and under it the label (`.lp-fb-l`) with a small coloured dot (`.lp-fb-d`, `--c`). The first fact, "All", has no dot.
- **Facts per list:**
  - Lists with a status: All, then one fact per status the list really has, in this order: Approved, Draft, Pending approval, Not active. Use the app's status words.
  - A status with no rows still shows, with `lp-zero` (dimmed). **Not active** shows only if the list has any.
  - Entitlement Users, Accounts: All, People, Service accounts, Enabled. Enabled is not part of the mix bar.
  - Entitlement Users, Entitlements: All, Privileged, Standard.
  - If a list has other natural groups, ask me.
- **Colours:**

  | Fact | `--c` |
  |---|---|
  | Approved / Active / Live / Enabled / Standard | `#2DD4BF` |
  | Draft / Privileged | `#F6C177` |
  | Pending approval | `#C9B6FF` |
  | Not active / Inactive / Rejected | `#46636F` |
  | People | `#5EEAD4` |
  | Service accounts | `#93C5FD` |

- **Clicking a fact** filters the list to it. Click again, or click All, to clear.
  - If the app already has a status filter (in the funnel menu, for example), **drive that same filter**, so both stay in sync.
  - If it doesn't, filter client-side on the loaded rows. On server-paged lists, ask me first.
  - The active fact gets `lp-on` (teal number, which bumps).
- **The mix bar** `.lp-dist` under the facts: one segment per fact except All and Enabled. `--c` is the colour, `--w` the count, `--k` the index. Segments grow in one after another. Skip facts with 0.
- **Counts:**
  - The counts must be for the **whole list**, not the visible page.
  - Client-side lists: count the loaded rows.
  - Server-paged lists: only if the API gives totals per status (Step 0, question 4). Otherwise hide the facts row and mix bar for that list and tell me.
- **Count-up:** when the list loads, refreshes, or the tab / application changes, the numbers count up from 0 over 750ms (13.4).

### 6.2 Toolbar (`.lp-tl`, right side of `.lp-cd-h`)

Left to right:
1. **Search** `.lp-sr`:
   - The magnifier icon, then the input with your existing search binding and placeholder.
   - While there is a query:
     - `.lp-hits` shows the number of matches, for example "3 matches". This is the number of rows after filtering **[IF DATA EXISTS]**; for server search, use the total the API returns, or omit it.
     - `.lp-x` clears the query.
   - With no query, a `<kbd>/</kbd>` hint shows. Pressing `/` anywhere on the page, outside an input, focuses the search. Add this key handler only if nothing else uses `/`.
   - The field widens from 280px to 320px on focus, and the icon nudges.
   - Matching text in the primary column is wrapped in `<mark>` [IF the search is client-side; otherwise skip the highlighting].
2. **Active filter chip** `.lp-fchip`: shown while a fact (6.1) is active, for example "Showing drafts", with an × that clears it.
3. **Your existing filter (funnel) button.**
   - Restyle it as `.lp-ibx`: a 38px square icon button. It gets `lp-on` while its panel is open **or** any of its filters is applied.
   - Its panel keeps its content. Give the panel container `.lp-pop` styling if it is a simple popover; if it's a library component, ask me.
4. A thin separator `.lp-tsep`.
5. **Refresh** `.lp-ibx`, with your existing refresh. It gets `lp-spin` while loading. If the page has no refresh button today, use the existing refresh function. If there is no refresh function at all, omit this button.
6. **Export** `lp-btn lp-btn-g` with the download icon, "Export" and a chevron. It opens `.lp-pop[role=menu]`:
   - heading "Export" with `.lp-mono` "N rows · N cols" (the numbers your export uses today);
   - one `.lp-xi` row per format you have today (CSV, Excel), each with a file badge (`.lp-ft.lp-csv` / `.lp-ft.lp-xls`), a bold name and a small description;
   - the note `.lp-pnote`. Use the app's text if it has one; otherwise "Only the rows and columns you can see are exported." Only use that sentence if it is true for your export. Otherwise ask me.

   After an export starts:
   - the button shows a drawn check and "Exported" for 1.8s (`lp-ok`);
   - your existing toast still shows.

### 6.3 Column chips (`.lp-colbar`)

A row under the card header:
- **Label:** the columns icon, "Columns", then `<b>` with the number of visible columns.
- **One `.lp-cch` chip per column** (`role="checkbox"`), in the current column order, including hidden ones.
  - Visible columns: `lp-on`, with a check.
  - Hidden columns: no `lp-on`, with a plus.
  - Click toggles visibility, through **your existing column-visibility handler**.
- **Locked columns:** columns that cannot be hidden today stay on, and get `lp-req` and `aria-disabled="true"`. Clicking does nothing.
- **Reorder:** if your column menu can reorder columns today, make the chips draggable with your existing reorder handler:
  - `draggable="true"`;
  - `lp-drag` on the dragged chip;
  - `lp-drop-l` / `lp-drop-r` on the target, to show the insert side.

  If reordering happens by dragging table headers today, keep that too.
- **Reset:** the `.lp-colrs` link appears when the columns differ from the default, **if a reset exists today**.
- When columns change, the table fades its columns in (13.5).
- On narrow screens the chip row wraps. That's fine.

### 6.4 The table: keep your component, restyle it with the adapter

**Do not replace your table component.** Resize, drag, sort, pin, pagination and row click live in it. Instead, add the **adapter classes** (end of Appendix A, the block starting `Table adapter`) to its existing elements.

Appendix A also contains `lp-tw`, `lp-tb`, `lp-tr`, `lp-th`, `lp-thc`, `lp-td` and `lp-tbd` from my div-grid mockup. **Never put them on your table elements.** They set `display: grid` and would break column widths. Use these equivalents. B.4 and B.5 show them in place:

| Part of your table | Add this class | Notes |
|---|---|---|
| Scroll container around the table | `lp-xt-wrap` | It already scrolls horizontally? Keep its overflow. Add `lp-xt-tall` + `lp-xt-sticky` (on the table) if the header should stick while the body scrolls. |
| The table / grid root | `lp-xt` | Add `lp-xt-click` if a row click opens the drawer (pointer cursor). |
| Header cell | `lp-xt-th` | Sorted column: add `lp-on`; descending: add `lp-desc`. |
| Sort indicator | the `arrow-up` icon with class `lp-sa`, after the header text | If the library draws its own indicator and you can't replace it, colour it `var(--accent2)` and tell me. |
| **Column resize handle** | `lp-xt-rz` on the handle element | While resizing: `lp-on` on the handle and `lp-resizing` on `.lp-xt`. The handle is a 9px hit area on the header's right edge, drawn as a 1px hairline that turns into a 2px glowing teal line on hover and while dragging. Keep your resize logic and min/max widths. |
| **Header being dragged** (reorder) | `lp-drag` on it; `lp-drop-l` / `lp-drop-r` on the target | Keep your drag logic. |
| Body (tbody / rows container) | `lp-xt-body` | Swap `lp-lb` on/off when the rows are replaced (13.1). |
| Body row | `lp-xt-r`, plus `style="--i: {index on page}"` and `data-k="{record id}"` | Selected row (its drawer is open): `lp-sel`. Pinned row: `lp-pinned`. Just-changed row: `lp-flash`, for 1.2s. |
| Body cell | `lp-xt-td` | 54px row height, 13px text, padding 0 12px. |
| `#` column cells | add `lp-xt-num` | Mono, faint. |
| Pin column | `lp-xt-td` with the `.lp-pin` button inside | B.5. |
| Edit pencil column | `lp-xt-act`, with the pencil as `.lp-rab` | B.5. |
| A full-width label row ("Pinned 2") | `<tr><td colspan="{all}" class="lp-xt-grp"><div class="lp-tgr">…</div></td></tr>` | Only if pinned rows already show as a group at the top today. Don't change pin ordering. |

**Rules:**
- Do not change column widths, min / max widths, the resize logic or the default order.
- If the library sets row height through a prop (for example `rowHeight`), set it to **54** and the header to **42**. Tell me which prop you changed. That is a visual setting, so it is allowed.
- If the library renders its own cell wrappers with backgrounds, borders or padding, neutralise them **only inside `.lp-xt`**. Write that override in a small separate file, `fmces-pages-overrides.css`, and show it to me.
- Long text truncates with an ellipsis. The full value goes in the `title` attribute.
- A row click keeps opening the existing drawer. Buttons inside a row (pin, OLA, pencil) must not open the drawer: keep your existing `stopPropagation`.

### 6.5 Cell content (B.5)

Restyle what is inside each cell by **column type**, keeping the value and its formatting:

| Column | Content | Classes |
|---|---|---|
| Pin | Pin button. Hidden until row hover; stays visible and tilted teal when pinned. A drop animation plays when pinned. | `.lp-pin` (+ `lp-on`) |
| `#` | Row number, as today | cell gets `lp-xt-num` |
| **Primary name** (Application, Feature, Action, ITAM name, Profile, Policy, Entitlement, Account) | Monogram tile + name. The monogram is the first letters of the first two words, or a user / server icon for accounts. | `.lp-nmb` > `.lp-mgx` (`lp-dr` draft, `lp-pd` pending, `lp-sv` service account) + `.lp-nmt` (+ `lp-mono` for code-style names) |
| IDs, codes, owners, CII Number, DLs, Bank IDs | Text | `.lp-tt.lp-mono` |
| Plain text, descriptions | Text, ellipsis, `title` = full value | `.lp-tt` |
| Empty value | Keep whatever the app shows today (for example "—") | `.lp-tt.lp-dim` |
| **Status** | Coloured dot + the app's status word | `.lp-lc` + variant (below) + `<span class="lp-lct">` |
| **Active** / Enabled | Dot + the app's word ("Active" / "Inactive", "Enabled" / "Disabled", or Yes / No) | `.lp-st` (teal) or `.lp-st.lp-off` (grey) |
| Version | Small mono pill | `.lp-vr` |
| Counts | Mono number, faint when 0 | `.lp-cn > b` (+ `lp-z`) |
| **Dates** (Onboard Date, Last modified, Created) | The date exactly as the app formats it, plus a small relative line under it (for example "3 days ago") | `.lp-dt` > `b` + `small` |
| **OLA** | Document icon button, your existing OLA handler | `.lp-ola` |
| Chips (application names, roles) | Small chip(s) | `.lp-chp` (+ `lp-mono`); several go in `.lp-chps` |
| Type (User / Service) | Icon + word | `.lp-ty` |
| Yes / No flags | Small pill, amber for Yes | `.lp-yn` (+ `lp-y`) |
| **Edit pencil** | Pencil icon button, your existing handler, always visible | `.lp-rab` |

**Status colour mapping** (`.lp-lc` variant). Confirm with your status list in Step 0:

| App status | Variant |
|---|---|
| Approved / Active / Live / Published | `lp-lc-live` (teal) |
| Draft | `lp-lc-draft` (amber) |
| Pending / Pending approval / Submitted | `lp-lc-pend` (lavender) |
| Inactive / Not active / Rejected / Retired / anything else | `lp-lc-off` (grey) |

If a row has an approved version **and** a newer draft, and the app shows that today, add the small amber `.lp-lc-x` pill after the word, with the app's text (for example "v3 draft").

Appendix A has a hover overlay for row actions (`.lp-ra`). **Do not use it.** The pencil stays in its column, and row click already opens the drawer.

### 6.6 Loading, empty, footer

- **Loading** (B.6): while rows load, show 6 shimmer rows (`.lp-skr` with `.lp-sk` bars) inside `.lp-xt-wrap` instead of the body. Use the widths in B.6.
  - If your table library has its own loading overlay, replace its look with these rows if possible. Otherwise ask.
  - Keep any existing "Loading…" text for screen readers, as `.lp-sr-only`.
- **Empty** (`.lp-emp`):
  - a floating icon tile, a 22px / 300 title, a muted sentence, and an optional small button;
  - use the app's existing empty messages;
  - when the emptiness comes from a search or a fact filter, the button clears it. Label it "Clear search" or "Show all", unless the app has its own label.
- **Footer** (`.lp-tf`):
  - Left: the app's existing footer text, for example "Showing 1–15 of 48" and "Updated just now".
  - Right: "Rows per page", then the `.lp-seg` segmented buttons **15 / 25 / 50**, with the active one `lp-on`, using your existing page-size handler. Then `.lp-pgr` page buttons (`.lp-pgb2`: previous, numbers, next; `lp-on` for the current page, `lp-dis` when disabled), with your existing pagination handlers.
  - If your pagination shows more controls (first / last, a page input), keep them and style them as `.lp-pgb2`.

---

## 7. Upload dialog (B.7), Data Policy Management

Restyle the existing **Upload Excel** modal. Keep its logic, accepted file types, validation and API call.

**Backdrop and panel:**
- `.lp-dw` is the fixed full-screen layer.
- Inside it, `.lp-dscrim` is the dimmed, blurred backdrop button (click = close, if your modal closes on backdrop click today).
- Then `.lp-dlg`, the panel: 560px, radius 20px, springs in.
- If your modal is a library component, keep it and put these classes on its backdrop and panel.

**Header `.lp-dlg-h`:**
- the upload icon tile `.lp-dlg-ic`;
- the title and sub-line (the app's text; mine are fallbacks);
- the close button `.lp-xclose`.

**Body `.lp-dlg-b`:** a numbered list `.lp-steps` of `.lp-stp` rows. Each row has a number `.lp-stp-n`, which turns teal with `lp-ok` when done, a bold line and a small line. Map the modal's existing content onto the steps:

1. **Template** (only if the modal offers a template download today): the link button `.lp-lk` with the download icon and the template file name, using your handler.
2. **Instructions** (only if the modal shows instructions today): their text.
3. **The file:**
   - With no file, the dashed drop zone `.lp-drop`: icon, "Drop your Excel file here", and "or browse your computer · .xlsx only" (use the app's text and accepted types). Clicking opens your file picker. Keep drag-and-drop if you have it.
   - With a file, the file card `.lp-fil`:
     - the XLSX badge, the file name, the size / status text `{upPT}`, and the remove button `.lp-xclose`;
     - while uploading, the thin `.lp-prog` bar with `--p` = percent **[IF DATA EXISTS]**, otherwise omit the bar;
     - no other progress decoration.

Renumber if a step is omitted.

**Footer `.lp-dlg-f`:**
- Cancel (`lp-btn lp-btn-g`);
- the primary Upload button (`lp-btn lp-btn-p`, with the upload icon). It gets `lp-dis` until a file is chosen, and while uploading.

**Errors:** keep the app's messages. Show them as a `.lp-err` line under the drop zone / file card, and tell me where you put them.

---

## 8. Entitlement Users (B.8)

**Finder card `.lp-fnd2`**, three columns:
- **Left `.lp-fz`, "Browse by application":** two large selects `.lp-sel2` joined by a chevron.
  - Each select has a numbered badge (`.lp-sel2-n`, which turns teal with `lp-ok` once chosen), a small label, the value (placeholder style `lp-ph` until chosen) and a chevron.
  - **Select 1** is **ITAM Name** (the app's label) with your existing options and handler.
  - **Select 2** is **Application**. It gets `lp-dis` until an ITAM is chosen, if that is how it works today.
  - Their popovers are `.lp-pop.lp-l` lists of `.lp-opt` rows, with a check on the selected row.
- **Middle `.lp-fz-or`:** a vertical "or" divider.
- **Right `.lp-fz`, "Find one person":** the existing quick search across ITAMs, as `.lp-qs` (56px), with the app's placeholder and a small primary **Search** button, plus Enter to search. Use your existing search handler.

The panel headings ("Browse by application", "Find one person") and their small lines are new helper text. Use them unless the app has its own wording.

**Before anything is chosen** (`.lp-hero0`):
- floating ghost rows (`.lp-ghost`, decorative);
- a 30px / 300 title and a muted line.
- Use **the app's existing empty message** as the title. My text is the fallback: "Pick an ITAM and an application to see who has access."

**While loading:** a card with skeleton rows (B.8, `uLoading`).

**When loaded:**
- **Glass tabs** `.lp-gt` (`--n: 2`): **Accounts** and **Entitlements**, with counts in `<em>`, and the sliding `.lp-ind`. Use your existing tab state.
- **Context card** `.lp-ctx` to the right of the tabs:
  - for an application: the monogram, the application name, and "ITAM {id}" (meta);
  - for a quick search: a search icon, “Results for “{query}””, and "across all ITAMs".
- Then the **list card** (section 6) with that tab's existing columns.
- **Create Account** stays in the header actions (3.2) with its existing handler.

---

## 9. Reports (B.9)

Two columns (`.lp-rb`): a sticky settings panel `.lp-rp` on the left (320–390px), the preview `.lp-rv` on the right.

**Settings panel:** numbered steps `.lp-rs2`. Each has a number badge (teal check once done), a title and its control. A vertical line joins the steps. The last step gets `lp-end`. Map **your existing controls** onto the steps (Step 0, question 7):

1. **Application:**
   - `.lp-sel2` select, with your existing options, handler and label.
   - While the list loads, show the app's "Loading applications…" text inside the select, in placeholder style.
2. **Fields** **[only if the page has field selection today]**:
   - the toggles `.lp-tg` in a two-column grid, with a check box that draws its check;
   - the step title shows the selected count in mono;
   - a "Select all" link `.lp-lk`, if that exists today.
3. **Format** **[only if the page has a format choice today]**: three radio cards `.lp-fmt` (CSV / Excel / JSON, as the app offers), each with a coloured file badge (`.lp-ft.lp-csv` / `.lp-xls` / `.lp-json`), a name and a small description.

If a step has no matching control, omit it and renumber. Never add a control that has no logic behind it.

**Footer `.lp-rp-f`:**
- **Generate Preview** (`lp-btn lp-btn-g`, eye icon);
- **Download Report** (`lp-btn lp-btn-p`, download icon, `lp-dis` until possible).
- Both use your existing handlers and labels.

If the app shows "N accounts loaded · Generate Preview" as a bar today, keep that text and place it as a `.lp-hintx` line above the two buttons. Keep the button inside it working.

**Preview area:**
- **Before a preview:** `.lp-hero0` with ghost rows, "Your report preview appears here" (or the app's text) and a muted line.
- **While generating:** a skeleton header bar and 6 skeleton rows.
- **After:**
  - The document header `.lp-doc-h`:
    - a file tab with the format badge and the file name, **[IF DATA EXISTS]**, otherwise omit;
    - `.lp-kv3` pills with the row and field counts;
    - the existing **search** as `.lp-sr` (240px).
  - The **results table**: your existing table, with the adapter classes (6.4), the `#` column mono and faint, and the same columns as today.
  - Below it, `.lp-more`: the app's "Showing 10 of 20" text and the **Show 10 more** / **Show all** buttons (`lp-btn lp-btn-g lp-btn-sm`), with your existing handlers.

---

## 10. FMID Reports (B.10)

### 10.1 Parameters (`.lp-pgrid`: parameters card + security matrix card)

**Parameters card `.lp-prm`.**

Heading: "Search parameters", or the app's text.

One row with three fields joined by chevrons:
1. **ITAM ID** `.lp-sel2`, with the numbered badge.
2. **Application** `.lp-sel2`, with the numbered badge.
3. **Bank IDs:**
   - The existing mode choice ("All users" / specific IDs, with the app's wording) as the `.lp-bseg` segmented control with a sliding `.lp-ind`.
   - When specific IDs are chosen, the existing input appears below as `.lp-ids` (textarea + helper line). Keep your parsing and validation.

**Required fields:**
- Keep the **asterisks**: add `<span class="lp-req-star">*</span>` after the label of each required field.
- Keep your validation messages, in `--err` colour.

**Footer `.lp-prm-f`:**
- **Fetch Report** (`lp-btn lp-btn-p`, arrow icon; `lp-dis` when the form is invalid or a fetch is running);
- **Reset** and **Clear all** (`lp-btn lp-btn-g`), each with its existing handler. If the app has both, keep both;
- then the app's hint text (for example "All users … one request") as `.lp-hintx`.

**Security matrix card `.lp-mxk`** (300px, blue-tinted):
- the XLSX badge;
- the title "Security matrix";
- a line naming the selected application;
- the **existing format select**, styled as `.lp-mini`;
- the **Download Security Matrix** button (`lp-btn lp-btn-g lp-btn-sm`, download icon, the app's label). Use your existing handler, and the existing enabled / disabled rule.

### 10.2 Before fetching

`.lp-hero0` with ghost rows, title "Nothing fetched yet" (or the app's text) and a muted line.

### 10.3 Results

**Result header `.lp-rhd`** (one rounded bar):
- The big number of users (`.lp-rhd-b`, 34px / 300, teal), with the label "users fetched", or the app's wording.
- The existing **result chips** as `.lp-kv3` pills, for example ITAM and Application, with the app's labels and values.
- The **Detail / Matrix** toggle as `.lp-vseg`, a two-segment glass control with a sliding `.lp-ind`. Use your existing view state.
- The **Export** group `.lp-xg`: a label plus joined buttons **CSV / Excel / JSON**, with your existing handlers.

**Card `.lp-cd` under it.**

Toolbar `.lp-cd-h`:
- The existing **search across all columns** as `.lp-sr`.
- The existing **filters**, each as a `.lp-dd` button with a `<small>` label and the current value. Their menus keep your options and logic.
- **Every existing info tooltip stays.** Put its trigger as an `.lp-inf` icon button (the `info` icon) next to the filter label, using your existing tooltip component and text.
- Use your filters, names and order, not mine. In my mockup they were Domicile, Entity, Bank ID and FMID; use whatever the app has.
- The **counts line**, right-aligned, mono, muted: exactly the app's text, for example "17 users · 16 cols · 0 FMIDs".

Detail view only: the **COLUMNS** group toggles `.lp-cgs`:
- the label, then one `.lp-tg` per existing group, with the drawn check;
- keep your rule that at least one group stays on.

### 10.4 Fetching

`.lp-fet` panel (`role=status`):
- the title "Fetching entitlement data…" (or the app's text);
- the muted line.
- **[IF the app knows real stages / progress]:** the stage pills `.lp-fstep` (`lp-now` amber blinking, `lp-done` teal) and the thin bar `.lp-ibar` with `--p`.
- **Otherwise:** omit the pills and the bar.

Never fake progress.

### 10.5 Detail view

Keep your existing grouped table and add the adapter (6.4) in its **dense** variant:
- `lp-xt lp-xt-dense lp-xt-sticky` on the table, plus `lp-xt-grouped` when it has the group header row (so the second header row sticks 34px lower);
- `lp-xt-wrap lp-xt-tall` on the scroller (max 560px high, scrolls both ways).

**Group header row** (the row with User / Supervisor / … group names):
- each cell `lp-xt-gh`, with `style="--c: {group colour}"`;
- a small square `<i></i>` before the name;
- a 2px coloured top line.

Group colours, in group order: `#5EEAD4`, `#93C5FD`, `#C9B6FF`, `#F6C177`, `#2DD4BF`, `#F4A3A3`. Continue the cycle for further groups.

**Column header row:** `lp-xt-th`. The first column of each group gets `lp-xt-gs` (hairline on its left), in both header rows and the body.

**First column** (the user: monogram + name + Bank ID in mono, `.lp-who`):
- sticky: `lp-xt-stk` on its header cells and body cells;
- entitlement lists inside a cell use `.lp-ent` with `.lp-chp` chips;
- Yes / No flags use `.lp-yn`.

Rows animate in. Hover tints the whole row.

### 10.6 Matrix view: keep ALL columns

- Use your **existing matrix table** with **every column it has today, in the same order, with the same headers and values**. I will check this against your Step 0 list.
- Style it with the same dense adapter as 10.5:
  - `lp-xt lp-xt-dense lp-xt-sticky` on the table, `lp-xt-wrap lp-xt-tall` on the scroller;
  - the first (user) column sticky with `lp-xt-stk`;
  - group header cells `lp-xt-gh`, if it has groups;
  - Yes / No or tick values as `.lp-yn` (+ `lp-y`), if that is how it shows them today;
  - IDs in mono.
- Do not add a totals row or column. Do not hide, merge or narrow columns to make it fit; it scrolls horizontally.
- If anything about the matrix is unclear, **ask before changing it**.

Switching Detail ↔ Matrix replays the row entrance (13.1).

---

## 11. Toast (B.11)

Restyle the existing toasts. Keep their messages and triggers.
- A light pill (`.lp-toast`), bottom centre, that springs up.
- A round teal check (`.lp-tck`) that pops and draws itself.
- The text.
- A thin countdown line (`.lp-tbar`) that runs for 2.6s, the toast's lifetime.

Add `lp-show` to show the toast and remove it to hide. If your toast library controls the lifetime, set it to 2.6s, or tell me its value so the countdown matches.

Error toasts, if you have them: keep them. Ask me before styling them.

---

## 12. Dock: the gliding highlight (only change to the dock)

Today the active dock item has a highlight: a filled tile plus a small teal bar on its left. Make that highlight **glide** to the newly active item instead of jumping. Don't change anything else in the dock.

- Inside the dock's item container (`position: relative`), add one `<span class="lp-dk-sl" aria-hidden="true"></span>` behind the items.
- Set its transform to `translateY({active item's offsetTop}px)`. Recompute on route change and on resize.
- Match its `left`, `width`, `height` and `border-radius` to the current active highlight, using `--dk-x` and the size values below. If yours differ, use yours and tell me.
- Remove the old static highlight's **background and left bar** from the active item, so it isn't drawn twice. The active item's icon colour and other states stay.
- The items need `position: relative; z-index: 1` so they sit above the slider.

Add this to `fmces-pages.css`, below Appendix A:

```css
/* FMCES dock: gliding active highlight (outside .lp-root) */
.lp-dk-sl{position:absolute;left:var(--dk-x,7px);top:0;width:44px;height:44px;border-radius:12px;background:#1B3140;box-shadow:inset 0 1px 0 rgba(255,255,255,.05);transition:transform .6s cubic-bezier(.34,1.36,.64,1);pointer-events:none;z-index:0}
.lp-dk-sl::before{content:"";position:absolute;left:-8px;top:12px;width:3px;height:20px;border-radius:0 3px 3px 0;background:#2DD4BF;box-shadow:0 0 10px rgba(45,212,191,.8)}
@media (prefers-reduced-motion:reduce){.lp-dk-sl{transition:none}}
```

If no dock item is active (for example on Home), hide the slider.

---

## 13. Motion: the JavaScript hooks

Most motion is pure CSS and already in Appendix A. These hooks need a few lines of JavaScript.

| Feature | How |
|---|---|
| Page entrance | Blocks with `lp-rz` and `--d` rise in on mount (3.1). |
| Row entrance | Rows rise in with a 38ms stagger (`--i`). |
| Glass tab / segment indicators | `transform: translateX(index × 100%)` on `.lp-ind`. |
| Button feedback | Lift on hover, press on click, a soft sheen on primary buttons, the plus icon turns, the download icon nudges. All CSS. |
| Popovers | Pop in from their corner, with rows staggering. CSS. |
| Checks | Checkboxes, toggles and "Saved" / "Exported" draw their check. CSS. |
| Facts bump | The active fact's number bumps. CSS. |
| Search | The icon nudges on focus, and the match count pops. CSS. |
| Pin | The pin drops in when pinned. CSS. |
| Skeleton shimmer, ghost float, empty icon float | CSS. |

### 13.1 Replay the row entrance when rows are replaced

Toggle `lp-lb` on `.lp-xt-body` (on → off → on…) each time the **whole row set** changes:
- tab change;
- fact or filter change;
- preset applied;
- refresh finished;
- application changed;
- results loaded;
- Detail ↔ Matrix.

Do **not** toggle it on sort, on pin, or when one row updates. Those use FLIP (13.3).

### 13.2 Directional tab slide

When the App Management tab or the Entitlement Users tab changes:
- add `lp-slr1` / `lp-slr2` to `.lp-xt-wrap` if the new tab is to the **right** of the old one, or `lp-sll1` / `lp-sll2` if it is to the **left**;
- alternate 1 and 2 on each change, so the animation restarts.

### 13.3 FLIP when rows move (sort, pin / unpin, fact filter)

Before the state change, record each row's `getBoundingClientRect().top` by its `data-k`. After the DOM updates:
- rows that still exist animate from their old position: `el.animate([{transform:'translateY(dy px)'},{transform:'none'}], {duration:600, easing:'cubic-bezier(.22,1,.36,1)'})`;
- new rows fade up: `[{opacity:0, transform:'translateY(8px)'},{opacity:1, transform:'none'}]`, 450ms, delay `60 + n×30ms`, fill `backwards`.

Skip it when `prefers-reduced-motion` is on.

If your table library virtualises rows or re-creates row elements in a way that breaks this, tell me. In that case use only 13.1.

### 13.4 Count-up

When facts appear or change, animate each number from 0 to its value over **750ms**. Use `requestAnimationFrame`, round to an integer, and finish on the exact value. Do the same for the FMID Reports user number. Skip it with reduced motion.

### 13.5 Column fade

When a column is shown or hidden, swap `lp-cf1` ↔ `lp-cf2` on `.lp-xt`. The table fades from 35% to full.

### 13.6 Flash

After an action changes one row in place (for example pin, or a save that returns to the list), add `lp-flash` to that row for 1.2s. **[IF DATA EXISTS]**: only when you know which row changed.

### 13.7 Not wanted (I tried these and removed them; do not add them)

- a loading progress line along the top of the page;
- a pulsing / pinging sync dot or a spinning sync icon in the header;
- moving stripes or an animated check on the upload progress;
- progress bars inside table cells (for example in Features / Actions counts);
- lifecycle bars or rings in the list;
- a cursor-following glow;
- a big hero panel replacing the header;
- any "restricted" / "role required" text.

---

## 14. States and errors

- Every async area has a loading look:
  - skeleton rows for tables;
  - a placeholder style inside selects;
  - the `.lp-fet` panel for FMID Reports.
- Error messages: keep the app's text and logic.
  - Inline errors go under the field as `.lp-err`.
  - A page-level error (for example, the list failed to load) uses the empty-state block `.lp-emp` inside the card. Put the app's error text in `.lp-emp-t` / `.lp-emp-s`, and add a Retry button (`lp-btn lp-btn-g lp-btn-sm`) only if a retry exists today.
  - Tell me where each error appears.
- Disabled buttons use `lp-dis` (or the native `disabled` attribute, which the CSS also styles), and keep their current enable rules.

---

## 15. Responsive

The CSS has the breakpoints at **900px**:
- The header actions wrap under the title.
- Search goes full width.
- The finder, the Reports two-column layout and the FMID parameter grid stack into one column.
- The parameter chevrons hide.

Tables keep horizontal scroll. Check every page at 1440, 1280, 1024 and 900px wide.

---

## 16. Accessibility

- Tabs: `role=tablist` / `tab`, with `aria-selected`.
- Toggles and chips: `aria-pressed` / `role=checkbox` + `aria-checked`.
- Selects: `aria-haspopup=listbox` + `aria-expanded`, and the options carry `aria-selected`.
- Menus: `role=menu` / `menuitem`.
- Popovers close on **Esc** and return focus to their trigger.
- Every icon-only button has an `aria-label` (pin, refresh, OLA, pencil, clear, close, info).
- Focus rings come from the CSS (`:focus-visible`, light teal). Don't remove them.
- Sorted headers carry `aria-sort`.
- The toast is `role=status` with `aria-live=polite`.
- Keep any existing aria attributes.

---

## 17. Build phases (report after each)

1. **Foundation:**
   - load `fmces-pages.css` and the fonts;
   - add the `.lp-root` / `.lp-pg` wrapper and the header (title, eyebrow, watermark, subtitle, refresh line) to all seven pages;
   - add the dock glide;
   - add the toast restyle.

   Check that nothing else in the app changed.
2. **One list end to end: Data Profiles.**
   - facts + mix bar;
   - toolbar (search, filter button, refresh, export menu);
   - column chips;
   - the table adapter with resize handle, drag, sort, pin, `#`, pencil, all cell types;
   - skeleton, empty state, footer;
   - Presets popover;
   - FLIP and the replays.

   **Stop and let me review before continuing.**
3. **Other list pages:**
   - Data Policy Management, including the Upload dialog;
   - Entitlement Management;
   - Application Management: tabs, New menu with role gating, application band for Features / Actions.
4. **Entitlement Users:** finder, start state, tabs, context card, lists.
5. **Reports.**
6. **FMID Reports:** parameters, matrix card, fetching, results header, filters with tooltips, Detail view, **Matrix view with all columns**.
7. **Polish pass:**
   - responsive widths;
   - reduced motion;
   - keyboard and screen reader checks;
   - re-run your Part 2 inventory line by line.

---

## 18. Acceptance checklist

**Behaviour unchanged:**
- [ ] Every line of the Step 0 Part 2 inventory is present and works, on every page.
- [ ] Column **resize** works on every table, and the handle shows the hairline → teal line on hover and while dragging.
- [ ] Column **reorder** works as before (headers and / or chips).
- [ ] Column show / hide works from the chips. Locked columns can't be hidden. Reset works, if it existed.
- [ ] Sort, pin, `#`, pencil, OLA, pagination (15 / 25 / 50), the filter funnel, Presets (save, defaults, star, delete, toasts), Export (CSV / Excel), Upload: all work exactly as before.
- [ ] Clicking a row opens the existing drawer. Clicking pin / OLA / pencil does not.

**Names and gating:**
- [ ] No dock name, page title, subtitle, column header, button label, tooltip or message text changed.
- [ ] The page is still **FMID Reports**. Nothing was renamed to "Entitlement Report".
- [ ] The ITAM tab and New → ITAM are visible only to the same roles as before, with no "restricted" text anywhere.

**Report pages:**
- [ ] FMID Reports Matrix view shows **every** column it showed before, in the same order.
- [ ] FMID Reports: asterisks, hint, Fetch / Reset / Clear all, the matrix format select + Download Security Matrix, result chips, Export CSV / Excel / JSON, filters **with info tooltips**, search, the counts line, COLUMNS groups: all present and working.
- [ ] Reports: Loading applications…, the accounts-loaded line, Generate Preview, Download Report, search, Showing X of Y, Show 10 more / Show all: all present and working.
- [ ] Entitlement Users: ITAM Name → Application, quick search, Accounts / Entitlements, the empty message, Create Account: all present and working.

**Data and styling:**
- [ ] No mockup sample data anywhere.
- [ ] Facts counts are correct for the whole list (or hidden, as agreed).
- [ ] The mockup grid classes (`lp-tr`, `lp-td`, `lp-tb`, `lp-thc`, `lp-tbd`, `lp-tw`) are not used on real table elements.
- [ ] Appendix A was copied verbatim. Any library overrides are in `fmces-pages-overrides.css` and were shown to me.

**Motion, layout, accessibility:**
- [ ] Motion matches section 13. Nothing from 13.7 was added. Reduced motion turns animations off.
- [ ] Checked at 1440 / 1280 / 1024 / 900px. Keyboard works, Esc closes popovers, and focus is visible.

---

# Appendix A — `fmces-pages.css` (copy verbatim)

One file. The last block before the reduced-motion rule is the **table adapter** (section 6.4). Add the dock CSS from section 12 below it.

```css
.lp-root{--bg:#0F1A22;--hdr:#0B141B;--surface:#111E27;--card:#152733;--card2:#1B3140;--border:#23394A;--hair:#1E3240;--btn:#2B4556;--text:#E6EEF2;--text2:#A7B8C4;--muted:#86A0AE;--faint:#6F8896;--accent:#2DD4BF;--accent2:#5EEAD4;--on:#062A26;--draft:#F6C177;--pend:#C9B6FF;--err:#F4A3A3;--blue:#93C5FD;--pop:#172A36;--pop-b:#2E4A5C;--dock-bg:#13222D;--row-h:rgba(23,43,56,.85);--row-hs:#162A37;--row-on:#183140;--mg-bg:linear-gradient(145deg,#1F3C4A,#142833);--mg-b:#2C4C5D;--tint:rgba(45,212,191,.10);--tint2:rgba(45,212,191,.16);--ease:cubic-bezier(.22,1,.36,1);--spring:cubic-bezier(.34,1.36,.64,1);font-family:'Manrope',system-ui,sans-serif;font-size:13px;line-height:1.45;color:var(--text);-webkit-font-smoothing:antialiased;color-scheme:dark}
.lp-root,.lp-root *,.lp-root *::before,.lp-root *::after{box-sizing:border-box}
:where(.lp-root) button{font:inherit;cursor:pointer;color:inherit}
:where(.lp-root) :is(h1,h2,h3,p,ol,ul,dl,dd){margin:0;padding:0}
:where(.lp-root) :is(ol,ul){list-style:none}
:where(.lp-root) :focus-visible{outline:2px solid var(--accent2);outline-offset:2px}
.lp-root kbd{display:inline-flex;align-items:center;justify-content:center;min-width:22px;height:22px;padding:0 6px;border-radius:6px;border:1px solid var(--btn);background:var(--surface);color:var(--text2);font:600 11px/1 'JetBrains Mono',monospace}
.lp-root mark{background:rgba(45,212,191,.28);color:#fff;border-radius:3px;padding:0 1px}
svg.lp-i{display:block;flex-shrink:0;fill:none;stroke:currentColor;stroke-width:1.75;stroke-linecap:round;stroke-linejoin:round}
svg.lp-i.lp-b{stroke-width:2.25}
.lp-mono{font-family:'JetBrains Mono',ui-monospace,monospace}
.lp-grow{flex:1 1 auto;min-width:0}
.lp-pg{position:relative;min-height:100%;padding:30px 44px 72px 14px}
.lp-aur{position:absolute;inset:0 0 auto 0;height:420px;overflow:hidden;pointer-events:none;z-index:0}
.lp-aur i{position:absolute;border-radius:50%;filter:blur(70px)}
.lp-aur .lp-a1{width:560px;height:420px;right:6%;top:-260px;background:radial-gradient(circle,#2DD4BF,transparent 65%);opacity:.11;animation:lp-drift1 28s ease-in-out infinite alternate}
.lp-aur .lp-a2{width:460px;height:380px;left:22%;top:-300px;background:radial-gradient(circle,#60A5FA,transparent 65%);opacity:.07;animation:lp-drift2 34s ease-in-out infinite alternate}
@keyframes lp-drift1{to{transform:translate(-180px,80px) scale(1.12)}}
@keyframes lp-drift2{to{transform:translate(160px,60px) scale(.92)}}
.lp-pga,.lp-pgb{position:relative;z-index:1}
@keyframes lp-rise{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:none}}
@keyframes lp-rise2{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:none}}
.lp-pga .lp-rz{animation:lp-rise .8s var(--ease) backwards;animation-delay:calc(var(--d,0)*80ms)}
.lp-pgb .lp-rz{animation:lp-rise2 .8s var(--ease) backwards;animation-delay:calc(var(--d,0)*80ms)}
.lp-ph{position:relative;display:flex;align-items:flex-end;gap:24px;flex-wrap:wrap}
.lp-ph-l{flex:1 1 420px;min-width:0}
.lp-eb{display:flex;align-items:center;gap:10px;font-size:11px;font-weight:700;letter-spacing:.09em;text-transform:uppercase;color:var(--muted)}
.lp-eb b{font:600 11px/1 'JetBrains Mono',monospace;letter-spacing:0;color:var(--accent2)}
.lp-eb i{width:18px;height:1px;background:var(--btn)}
.lp-pt{margin-top:10px;font-size:36px;font-weight:300;letter-spacing:-.028em;line-height:1.1}
.lp-ps{margin-top:10px;display:flex;align-items:center;flex-wrap:wrap;gap:6px 14px;color:var(--text2);font-size:13.5px}
.lp-sync{display:inline-flex;align-items:center;gap:8px;height:26px;padding:0 10px 0 8px;border:0;border-radius:999px;background:rgba(255,255,255,.035);color:var(--muted);font-size:12px;font-weight:600}
.lp-sync:hover{color:var(--text)}
.lp-pulse{width:7px;height:7px;border-radius:50%;background:var(--accent);animation:lp-breathe 2.4s ease-in-out infinite;flex-shrink:0}
.lp-sync.lp-busy .lp-pulse{background:var(--draft);animation:none}
@keyframes lp-breathe{0%,100%{box-shadow:0 0 0 0 rgba(45,212,191,.55)}60%{box-shadow:0 0 0 7px rgba(45,212,191,0)}}
.lp-bign{position:absolute;right:300px;top:-46px;font-size:150px;font-weight:200;letter-spacing:-.06em;line-height:1;color:rgba(230,238,242,.035);pointer-events:none;user-select:none}
.lp-pa{display:flex;align-items:center;gap:8px;flex-wrap:wrap}
.lp-btn{position:relative;display:inline-flex;align-items:center;gap:8px;height:38px;padding:0 14px;border-radius:10px;border:1px solid transparent;font-size:13px;font-weight:700;white-space:nowrap;transition:background-color .2s,border-color .2s,color .2s,box-shadow .25s,transform .3s var(--spring)}
.lp-btn:active{transform:scale(.97)}
.lp-btn-g{border-color:var(--btn);background:rgba(17,30,39,.6);color:var(--text)}
.lp-btn-g:hover{border-color:#3A5A6C;background:var(--card)}
.lp-btn-g.lp-on{border-color:var(--accent);color:var(--accent2);background:var(--tint)}
.lp-btn-p{background:var(--accent);color:var(--on);box-shadow:0 10px 26px -12px rgba(45,212,191,.7)}
.lp-btn-p:hover{background:var(--accent2);box-shadow:0 14px 30px -10px rgba(45,212,191,.75)}
.lp-btn.lp-dis{opacity:.45;pointer-events:none}
.lp-btn .lp-sub{font-weight:600;color:var(--muted)}
.lp-btn .lp-bdg{min-width:18px;height:18px;padding:0 5px;border-radius:9px;background:var(--card2);color:var(--accent2);font:600 10.5px/18px 'JetBrains Mono',monospace;text-align:center}
.lp-btn-sm{height:32px;padding:0 11px;font-size:12.5px;border-radius:9px}
.lp-ibx{width:38px;height:38px;display:inline-flex;align-items:center;justify-content:center;border-radius:10px;border:1px solid var(--btn);background:rgba(17,30,39,.6);color:var(--text2);padding:0;transition:border-color .2s,color .2s,background-color .2s,transform .3s var(--spring)}
.lp-ibx:hover{color:var(--text);border-color:#3A5A6C}
.lp-ibx.lp-on{border-color:var(--accent);color:var(--accent2);background:var(--tint)}
.lp-ibx.lp-spin svg{animation:lp-spin .8s linear infinite}
@keyframes lp-spin{to{transform:rotate(360deg)}}
.lp-lk{display:inline-flex;align-items:center;gap:6px;border:0;background:transparent;padding:0;color:var(--accent2);font-weight:700;font-size:12.5px}
.lp-lk:hover{color:#99F6E4}
.lp-anc{position:relative;display:inline-flex}
.lp-gtw{margin-top:26px;display:flex;align-items:center;gap:16px;flex-wrap:wrap}
.lp-gt{position:relative;display:inline-grid;grid-template-columns:repeat(var(--n),minmax(136px,1fr));padding:4px;border-radius:14px;background:rgba(255,255,255,.035);border:1px solid rgba(255,255,255,.07);max-width:100%;overflow-x:auto}
.lp-gt .lp-ind{position:absolute;top:4px;bottom:4px;left:4px;width:calc((100% - 8px) / var(--n));border-radius:10px;background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.12);box-shadow:inset 0 1px 0 rgba(255,255,255,.06);transition:transform .55s var(--spring)}
.lp-gt button{position:relative;z-index:1;height:38px;padding:0 20px;border:0;border-radius:10px;background:transparent;color:var(--muted);font-size:13.5px;font-weight:700;display:flex;align-items:center;justify-content:center;gap:8px;white-space:nowrap;transition:color .2s}
.lp-gt button:hover{color:var(--text2)}
.lp-gt button.lp-on{color:var(--text)}
.lp-gt em{font-style:normal;font:600 11px/1 'JetBrains Mono',monospace;color:var(--faint)}
.lp-gt button.lp-on em{color:var(--accent2)}
.lp-cd{margin-top:18px;border-radius:18px;border:1px solid var(--border);background:linear-gradient(180deg,rgba(21,39,51,.72),rgba(17,30,39,.5));box-shadow:0 40px 70px -50px rgba(0,0,0,.9)}
.lp-cd-h{display:flex;align-items:center;gap:18px 24px;padding:18px 18px 16px 22px;flex-wrap:wrap}
.lp-cd-h .lp-tl{margin-left:auto}
.lp-fx{display:flex;flex-direction:column;gap:12px;min-width:0;width:max-content;max-width:100%}
.lp-fx-r{display:flex;align-items:stretch;flex-wrap:wrap;row-gap:8px}
.lp-fb{position:relative;display:flex;flex-direction:column;align-items:flex-start;gap:3px;padding:2px 22px 4px 0;border:0;background:transparent;color:var(--text);text-align:left;transition:color .2s}
.lp-fb+.lp-fb{padding-left:22px;border-left:1px solid var(--hair)}
.lp-fb-n{font-size:30px;font-weight:300;letter-spacing:-.03em;line-height:1;font-variant-numeric:tabular-nums;transition:color .25s}
.lp-fb-l{display:flex;align-items:center;gap:7px;font-size:11.5px;font-weight:700;color:var(--muted);white-space:nowrap;transition:color .2s}
.lp-fb-d{width:7px;height:7px;border-radius:50%;background:var(--c,#3E5C6C)}
.lp-fb:hover .lp-fb-l{color:var(--text2)}
.lp-fb.lp-on .lp-fb-n{color:var(--accent2)}
.lp-fb.lp-on .lp-fb-l{color:var(--text)}
.lp-fb.lp-zero .lp-fb-n{color:var(--faint)}
.lp-dist{display:flex;gap:3px;height:4px;width:100%;margin-top:2px}
.lp-dist i{display:block;height:100%;border-radius:2px;background:var(--c);flex:var(--w) 1 0;transform-origin:left;animation:lp-grow 1s var(--ease) backwards;animation-delay:calc(var(--k) * 120ms + 250ms)}
@keyframes lp-grow{from{transform:scaleX(0);opacity:0}to{transform:none;opacity:1}}
.lp-tl{display:flex;align-items:center;gap:8px;flex-wrap:wrap}
.lp-sr{display:flex;align-items:center;gap:8px;width:280px;height:38px;padding:0 8px 0 12px;border-radius:10px;border:1px solid var(--btn);background:rgba(15,26,34,.65);color:var(--muted);transition:border-color .2s,box-shadow .25s,width .4s var(--ease)}
.lp-sr:focus-within{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint);width:320px;color:var(--accent2)}
.lp-sr input{flex:1;min-width:0;height:100%;border:0;background:transparent;color:var(--text);font:inherit;font-size:13px;outline:none}
.lp-sr input::placeholder{color:var(--faint)}
.lp-sr input::-webkit-search-cancel-button{display:none}
.lp-sr .lp-x{width:22px;height:22px;display:flex;align-items:center;justify-content:center;border:0;border-radius:6px;background:var(--card2);color:var(--text2);padding:0}
.lp-fchip{display:inline-flex;align-items:center;gap:6px;height:30px;padding:0 6px 0 11px;border-radius:999px;background:var(--tint);border:1px solid var(--tint2);color:var(--accent2);font-size:12px;font-weight:700;animation:lp-pop .3s var(--spring)}
.lp-fchip button{width:20px;height:20px;display:flex;align-items:center;justify-content:center;border:0;border-radius:50%;background:transparent;color:var(--accent2);padding:0}
.lp-fchip button:hover{background:var(--tint2)}
.lp-tsep{width:1px;height:22px;background:var(--hair);margin:0 2px}
.lp-tw{overflow-x:auto;border-top:1px solid var(--hair)}
.lp-tb{width:100%;min-width:var(--minw)}
.lp-tr{position:relative;display:grid;grid-template-columns:var(--cols);align-items:center;min-height:54px;padding:0 10px 0 6px;border-bottom:1px solid rgba(30,50,64,.65);transition:background-color .16s}
.lp-tr.lp-th{min-height:42px;border-bottom-color:var(--hair);position:relative}
.lp-thc{display:flex;align-items:center;padding:0 12px;min-width:0}
.lp-sb{display:inline-flex;align-items:center;gap:6px;height:28px;padding:0 6px;margin-left:-6px;border:0;border-radius:7px;background:transparent;color:var(--muted);font-size:11px;font-weight:700;letter-spacing:.065em;text-transform:uppercase;white-space:nowrap;transition:color .2s,background-color .2s}
.lp-sb:hover{color:var(--text2);background:rgba(255,255,255,.04)}
.lp-sb .lp-sa{opacity:0;transition:transform .35s var(--spring),opacity .2s}
.lp-sb:hover .lp-sa{opacity:.6}
.lp-thc.lp-on .lp-sb{color:var(--accent2)}
.lp-thc.lp-on .lp-sa{opacity:1}
.lp-thc.lp-desc .lp-sa{transform:rotate(180deg)}
.lp-tbd .lp-tr{animation:lp-row .55s var(--ease) backwards;animation-delay:calc(var(--i) * 38ms + 60ms)}
.lp-tbd.lp-lb .lp-tr{animation-name:lp-row2}
@keyframes lp-row{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
@keyframes lp-row2{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
.lp-tbd .lp-tr:hover{background:var(--row-hs)}
.lp-tbd .lp-tr.lp-sel{background:var(--row-on);box-shadow:inset 3px 0 0 var(--accent)}
.lp-tbd .lp-tr.lp-pinned{background:linear-gradient(90deg,rgba(45,212,191,.06),transparent 40%)}
.lp-tbd .lp-tr.lp-pinned:hover{background:var(--row-h)}
.lp-tbd .lp-tr.lp-flash{animation:lp-flash 1.2s var(--ease)}
@keyframes lp-flash{0%{background:rgba(45,212,191,.22)}100%{background:transparent}}
.lp-td{display:flex;align-items:center;gap:8px;padding:0 12px;min-width:0;color:var(--text2);font-size:13px}
.lp-tt{overflow:hidden;text-overflow:ellipsis;white-space:nowrap;min-width:0}
.lp-tt.lp-mono{font-size:12.5px}
.lp-tt.lp-dim{color:var(--faint)}
.lp-tgr{display:flex;align-items:center;gap:10px;height:34px;padding:0 22px;font-size:11px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--muted);border-bottom:1px solid rgba(30,50,64,.65);background:rgba(15,26,34,.35)}
.lp-tgr b{font:600 11px/1 'JetBrains Mono',monospace;letter-spacing:0;color:var(--accent2)}
.lp-tgr svg{color:var(--accent)}
.lp-pin{width:30px;height:30px;display:flex;align-items:center;justify-content:center;border:0;border-radius:8px;background:transparent;color:var(--faint);padding:0;opacity:0;transition:opacity .2s,color .2s,background-color .2s,transform .35s var(--spring)}
.lp-tr:hover .lp-pin,.lp-pin:focus-visible{opacity:1}
.lp-pin:hover{color:var(--text);background:rgba(255,255,255,.05)}
.lp-pin.lp-on{opacity:1;color:var(--accent);transform:rotate(-18deg)}
.lp-pin.lp-on svg{fill:rgba(45,212,191,.25)}
.lp-thpin{display:flex;justify-content:center;color:var(--faint)}
.lp-nmb{display:flex;align-items:center;gap:12px;min-width:0;width:100%;height:54px;padding:0;border:0;background:transparent;color:var(--text);text-align:left}
.lp-mgx{width:32px;height:32px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:9px;background:var(--mg-bg);border:1px solid var(--mg-b);color:var(--accent2);font-size:11px;font-weight:800;letter-spacing:.02em;transition:transform .35s var(--spring),border-color .2s}
.lp-tr:hover .lp-mgx{transform:scale(1.06);border-color:#3D6476}
.lp-mgx.lp-dr{color:var(--draft);border-color:rgba(246,193,119,.35);background:linear-gradient(145deg,#3A3424,#1F2A2C)}
.lp-mgx.lp-pd{color:var(--pend);border-color:rgba(201,182,255,.35)}
.lp-mgx.lp-sv{color:var(--blue)}
.lp-nmt{display:block;min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;font-size:13.5px;font-weight:700}
.lp-nmt.lp-mono{font-size:12.5px;font-weight:600}
.lp-nmb:hover .lp-nmt{color:#fff;text-decoration:underline;text-decoration-color:rgba(94,234,212,.5);text-underline-offset:3px}
.lp-lc{display:inline-flex;align-items:center;gap:8px;white-space:nowrap}
.lp-lc>i{width:7px;height:7px;border-radius:50%;flex-shrink:0;background:var(--accent);box-shadow:0 0 0 3px rgba(45,212,191,.14);animation:lp-pop .4s var(--spring) backwards;animation-delay:calc(var(--i,0) * 38ms + 260ms)}
.lp-lct{font-size:12.5px;font-weight:700}
.lp-lc-live .lp-lct{color:var(--accent2)}
.lp-lc-draft>i{background:var(--draft);box-shadow:0 0 0 3px rgba(246,193,119,.14)}
.lp-lc-draft .lp-lct{color:var(--draft)}
.lp-lc-pend>i{background:var(--pend);box-shadow:0 0 0 3px rgba(201,182,255,.14)}
.lp-lc-pend .lp-lct{color:var(--pend)}
.lp-lc-off>i{background:var(--faint);box-shadow:none}
.lp-lc-off .lp-lct{color:var(--muted)}
.lp-lc-x{display:inline-flex;align-items:center;gap:5px;height:20px;padding:0 7px;border-radius:6px;background:rgba(246,193,119,.12);color:var(--draft);font:600 10.5px/1 'JetBrains Mono',monospace}
.lp-vr{display:inline-flex;align-items:center;height:22px;padding:0 7px;border-radius:6px;background:rgba(255,255,255,.05);border:1px solid rgba(255,255,255,.06);color:var(--text2);font:600 11.5px/1 'JetBrains Mono',monospace}
.lp-cn{display:flex;align-items:center;gap:10px;width:100%}
.lp-cn b{font:500 13px/1 'JetBrains Mono',monospace;color:var(--text)}
.lp-cn b.lp-z{color:var(--faint)}
.lp-dt{display:flex;flex-direction:column;line-height:1.25}
.lp-dt b{font-weight:600;color:var(--text2);font-size:12.5px}
.lp-dt small{font-size:11px;color:var(--faint)}
.lp-ola{width:32px;height:32px;display:flex;align-items:center;justify-content:center;border:1px solid transparent;border-radius:9px;background:rgba(147,197,253,.08);color:var(--blue);padding:0;transition:background-color .2s,border-color .2s,transform .3s var(--spring)}
.lp-ola:hover{background:rgba(147,197,253,.16);border-color:rgba(147,197,253,.35);transform:translateY(-1px)}
.lp-chp{display:inline-flex;align-items:center;height:24px;padding:0 9px;border-radius:7px;background:rgba(255,255,255,.045);border:1px solid rgba(255,255,255,.06);color:var(--text2);font-size:12px;font-weight:600;white-space:nowrap;max-width:240px;overflow:hidden;text-overflow:ellipsis}
.lp-chp.lp-mono{font-family:'JetBrains Mono',monospace;font-size:11.5px}
.lp-chp em{font-style:normal;color:var(--faint);margin-right:5px}
.lp-chps{display:flex;gap:6px;min-width:0;overflow:hidden}
.lp-st{display:inline-flex;align-items:center;gap:7px;font-size:12.5px;font-weight:700;color:var(--accent2)}
.lp-st i{width:7px;height:7px;border-radius:50%;background:var(--accent);box-shadow:0 0 0 3px rgba(45,212,191,.14)}
.lp-st.lp-off{color:var(--muted)}
.lp-st.lp-off i{background:var(--faint);box-shadow:none}
.lp-st.lp-warn{color:var(--draft)}
.lp-st.lp-warn i{background:var(--draft);box-shadow:0 0 0 3px rgba(246,193,119,.14)}
.lp-ty{display:inline-flex;align-items:center;gap:7px;color:var(--text2);font-size:12.5px;font-weight:600}
.lp-ty svg{color:var(--muted)}
.lp-ra{position:absolute;right:8px;top:50%;display:flex;gap:4px;padding:4px 4px 4px 36px;background:linear-gradient(90deg,rgba(23,43,56,0),var(--row-hs) 34px);opacity:0;transform:translate(8px,-50%);transition:opacity .2s,transform .3s var(--ease);pointer-events:none}
.lp-tr:hover .lp-ra,.lp-tr:focus-within .lp-ra{opacity:1;transform:translate(0,-50%);pointer-events:auto}
.lp-tr.lp-sel .lp-ra{--row-hs:#183140}
.lp-rab{width:32px;height:32px;display:flex;align-items:center;justify-content:center;border:0;border-radius:9px;background:transparent;color:var(--text2);padding:0}
.lp-rab:hover{background:var(--card2);color:var(--text)}
.lp-sk{display:block;height:10px;border-radius:5px;background:linear-gradient(90deg,#1B3140 0%,#25404F 50%,#1B3140 100%);background-size:200% 100%;animation:lp-shim 1.3s linear infinite}
@keyframes lp-shim{to{background-position:-200% 0}}
.lp-skr{display:grid;grid-template-columns:44px 32px minmax(160px,2fr) 1fr 1fr 1fr 1fr;gap:18px;align-items:center;height:54px;padding:0 22px;border-bottom:1px solid rgba(30,50,64,.65)}
.lp-skr .lp-sk.lp-av{width:32px;height:32px;border-radius:9px}
.lp-emp{display:flex;flex-direction:column;align-items:center;gap:10px;padding:56px 20px 60px;text-align:center}
.lp-emp-t{font-size:22px;font-weight:300;letter-spacing:-.02em;color:var(--text)}
.lp-emp-s{font-size:13px;color:var(--muted);max-width:420px}
.lp-emp-ic{width:52px;height:52px;display:flex;align-items:center;justify-content:center;border-radius:16px;background:var(--tint);border:1px solid var(--tint2);color:var(--accent2);animation:lp-bob 3.2s ease-in-out infinite}
@keyframes lp-bob{50%{transform:translateY(-4px)}}
.lp-tf{display:flex;align-items:center;gap:14px;flex-wrap:wrap;padding:12px 18px 14px 22px;font-size:12.5px;color:var(--muted)}
.lp-tf b{color:var(--text2);font-weight:700}
.lp-seg{display:inline-flex;padding:3px;border-radius:9px;background:rgba(255,255,255,.035);border:1px solid rgba(255,255,255,.06)}
.lp-seg button{height:26px;min-width:34px;padding:0 9px;border:0;border-radius:7px;background:transparent;color:var(--muted);font:600 11.5px/1 'JetBrains Mono',monospace;transition:background-color .2s,color .2s}
.lp-seg button.lp-on{background:rgba(255,255,255,.09);color:var(--text)}
.lp-pgr{display:flex;align-items:center;gap:4px}
.lp-pgb2{min-width:30px;height:30px;display:flex;align-items:center;justify-content:center;border:0;border-radius:8px;background:transparent;color:var(--muted);font:600 12px/1 'JetBrains Mono',monospace;padding:0 6px}
.lp-pgb2:hover{background:var(--card2);color:var(--text)}
.lp-pgb2.lp-on{background:var(--tint);color:var(--accent2)}
.lp-pgb2.lp-dis{opacity:.35;pointer-events:none}
.lp-pscrim{position:fixed;inset:0;z-index:55;border:0;padding:0;background:transparent;cursor:default}
.lp-pop{position:absolute;top:calc(100% + 8px);right:0;z-index:60;width:300px;padding:8px;border-radius:14px;background:var(--pop);border:1px solid var(--pop-b);box-shadow:0 34px 70px -24px rgba(0,0,0,.9);transform-origin:top right;animation:lp-pop .26s var(--spring);text-align:left}
.lp-pop.lp-l{left:0;right:auto;transform-origin:top left}
@keyframes lp-pop{from{opacity:0;transform:translateY(-6px) scale(.96)}to{opacity:1;transform:none}}
.lp-pop-h{display:flex;align-items:center;gap:8px;padding:8px 10px 10px;font-size:12px;font-weight:800;color:var(--text)}
.lp-pop-h .lp-mono{margin-left:auto;font-size:11px;font-weight:500;color:var(--muted)}
.lp-pop-g{padding:10px 10px 6px;font-size:10.5px;font-weight:700;letter-spacing:.09em;text-transform:uppercase;color:var(--faint)}
.lp-cko{display:flex;align-items:center;gap:10px;width:100%;height:36px;padding:0 10px;border:0;border-radius:9px;background:transparent;color:var(--text2);font-size:13px;font-weight:600;text-align:left;animation:lp-rise .35s var(--ease) backwards;animation-delay:calc(var(--i) * 22ms)}
.lp-cko:hover{background:rgba(255,255,255,.05);color:var(--text)}
.lp-cko.lp-dis{opacity:.55;pointer-events:none}
.lp-ckb{width:18px;height:18px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:5px;border:1.5px solid #3E5C6C;color:transparent;transition:background-color .2s,border-color .2s,color .2s}
.lp-cko.lp-on .lp-ckb{background:var(--accent);border-color:var(--accent);color:var(--on)}
.lp-xi{display:flex;align-items:center;gap:12px;width:100%;padding:10px;border:0;border-radius:10px;background:transparent;color:var(--text);text-align:left;transition:background-color .15s}
.lp-xi:hover{background:rgba(255,255,255,.05)}
.lp-xi b{display:block;font-size:13px;font-weight:700}
.lp-xi small{display:block;font-size:11.5px;color:var(--muted)}
.lp-ft{width:38px;height:38px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:10px;font:700 10px/1 'JetBrains Mono',monospace;letter-spacing:.02em}
.lp-ft.lp-csv{background:rgba(45,212,191,.12);color:var(--accent2)}
.lp-ft.lp-xls{background:rgba(147,197,253,.12);color:var(--blue)}
.lp-ft.lp-json{background:rgba(201,182,255,.12);color:var(--pend)}
.lp-pnote{padding:8px 10px 4px;font-size:11.5px;color:var(--faint)}
.lp-pin2{display:flex;gap:8px;padding:2px 2px 8px}
.lp-pin2 input{flex:1;min-width:0;height:36px;padding:0 12px;border-radius:9px;border:1px solid var(--btn);background:var(--surface);color:var(--text);font:inherit;font-size:13px;outline:none;transition:border-color .2s,box-shadow .2s}
.lp-pin2 input:focus{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint)}
.lp-vw{display:flex;align-items:center;gap:8px;padding:0 6px 0 10px;height:40px;border-radius:10px;transition:background-color .15s;animation:lp-rise .35s var(--ease) backwards;animation-delay:calc(var(--i) * 30ms)}
.lp-vw:hover{background:rgba(255,255,255,.04)}
.lp-vw.lp-on{background:var(--tint)}
.lp-vw-b{flex:1;min-width:0;display:flex;align-items:center;gap:10px;height:100%;border:0;background:transparent;padding:0;color:var(--text);font-size:13px;font-weight:700;text-align:left}
.lp-vw-b small{font-size:11.5px;font-weight:600;color:var(--muted)}
.lp-vw.lp-on .lp-vw-b{color:var(--accent2)}
.lp-vwi{width:28px;height:28px;display:flex;align-items:center;justify-content:center;border:0;border-radius:7px;background:transparent;color:var(--faint);padding:0}
.lp-vwi:hover{background:rgba(255,255,255,.06);color:var(--text)}
.lp-vwi.lp-on{color:var(--draft)}
.lp-vwi.lp-on svg{fill:rgba(246,193,119,.35)}
.lp-vwi.lp-del:hover{color:var(--err)}
.lp-opt{display:flex;align-items:center;gap:10px;width:100%;min-height:44px;padding:6px 10px;border:0;border-radius:10px;background:transparent;color:var(--text);text-align:left;animation:lp-rise .35s var(--ease) backwards;animation-delay:calc(var(--i) * 25ms)}
.lp-opt:hover{background:rgba(255,255,255,.05)}
.lp-opt.lp-on{background:var(--tint)}
.lp-opt b{display:block;font-size:13px;font-weight:700}
.lp-opt small{display:block;font:500 11px/1.3 'JetBrains Mono',monospace;color:var(--muted)}
.lp-opt .lp-ok{margin-left:auto;color:var(--accent)}
.lp-dw{position:fixed;inset:0;z-index:80;display:flex;align-items:center;justify-content:center;padding:24px}
.lp-dscrim{position:absolute;inset:0;border:0;padding:0;background:rgba(5,10,14,.6);-webkit-backdrop-filter:blur(4px);backdrop-filter:blur(4px);animation:lp-fade .3s;cursor:default}
@keyframes lp-fade{from{opacity:0}}
.lp-dlg{position:relative;width:min(560px,100%);border-radius:20px;background:linear-gradient(180deg,#172C39,#122230);border:1px solid var(--pop-b);box-shadow:0 60px 120px -40px rgba(0,0,0,.95);animation:lp-dlgin .45s var(--spring)}
@keyframes lp-dlgin{from{opacity:0;transform:translateY(16px) scale(.97)}to{opacity:1;transform:none}}
.lp-dlg-h{display:flex;align-items:center;gap:14px;padding:22px 22px 6px}
.lp-dlg-h h2{font-size:19px;font-weight:700;letter-spacing:-.01em}
.lp-dlg-h p{margin-top:2px;font-size:12.5px;color:var(--muted)}
.lp-dlg-ic{width:42px;height:42px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:12px;background:var(--tint);border:1px solid var(--tint2);color:var(--accent2)}
.lp-dlg-b{padding:16px 22px 6px}
.lp-dlg-f{display:flex;align-items:center;justify-content:flex-end;gap:8px;padding:16px 22px 20px}
.lp-xclose{width:34px;height:34px;display:flex;align-items:center;justify-content:center;border:0;border-radius:9px;background:transparent;color:var(--text2);padding:0;margin-left:auto;align-self:flex-start}
.lp-xclose:hover{background:var(--card2);color:var(--text)}
.lp-steps{display:flex;flex-direction:column;gap:0}
.lp-stp{position:relative;display:flex;align-items:flex-start;gap:12px;padding-bottom:14px}
.lp-stp:not(:last-child)::before{content:"";position:absolute;left:11px;top:26px;bottom:2px;width:1px;background:var(--hair)}
.lp-stp-n{width:23px;height:23px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;border:1px solid var(--btn);font:600 11px/1 'JetBrains Mono',monospace;color:var(--text2);background:var(--surface)}
.lp-stp-n.lp-ok{background:var(--accent);border-color:var(--accent);color:var(--on)}
.lp-stp b{display:block;font-size:13px;font-weight:700}
.lp-stp small{display:block;margin-top:2px;font-size:12px;color:var(--muted)}
.lp-drop{display:flex;flex-direction:column;align-items:center;gap:8px;width:100%;margin-top:4px;padding:26px 16px 24px;border-radius:16px;border:1.5px dashed #34576A;background:rgba(15,26,34,.5);color:var(--text);transition:border-color .25s,background-color .25s,transform .3s var(--spring)}
.lp-drop:hover{border-color:var(--accent);background:rgba(45,212,191,.05);transform:scale(1.01)}
.lp-drop b{font-size:14px;font-weight:700}
.lp-drop small{font-size:12px;color:var(--muted)}
.lp-drop .lp-u{color:var(--accent2);font-weight:700}
.lp-drop-ic{width:46px;height:46px;display:flex;align-items:center;justify-content:center;border-radius:50%;background:var(--tint);color:var(--accent2);animation:lp-bob 2.6s ease-in-out infinite}
.lp-fil{display:flex;align-items:center;gap:12px;margin-top:4px;padding:12px 12px 12px 12px;border-radius:14px;border:1px solid var(--border);background:var(--card);animation:lp-rise .4s var(--ease)}
.lp-fil b{display:block;font-size:13px;font-weight:700}
.lp-fil small{display:block;font-size:11.5px;color:var(--muted)}
.lp-prog{height:4px;margin-top:8px;border-radius:2px;background:rgba(255,255,255,.07);overflow:hidden}
.lp-prog i{display:block;height:100%;width:var(--p);border-radius:2px;background:linear-gradient(90deg,#1E8F82,#5EEAD4);transition:width .25s linear}
.lp-toast{position:fixed;left:calc(50% + 42px);bottom:28px;z-index:120;display:flex;align-items:center;gap:9px;height:42px;padding:0 18px 0 13px;border-radius:999px;background:#E6EEF2;color:#0F1A22;font-size:13px;font-weight:700;white-space:nowrap;transform:translate(-50%,18px);opacity:0;pointer-events:none;transition:transform .4s var(--spring),opacity .2s;box-shadow:0 20px 40px -14px rgba(0,0,0,.7)}
.lp-toast.lp-show{transform:translate(-50%,0);opacity:1}
.lp-toast svg{color:#0E9F8D}
.lp-sel2{display:flex;align-items:center;gap:12px;min-width:220px;height:56px;padding:0 14px 0 12px;border-radius:12px;border:1px solid transparent;background:rgba(15,26,34,.55);color:var(--text);text-align:left;transition:border-color .2s,background-color .2s}
.lp-sel2:hover,.lp-sel2.lp-on{border-color:var(--accent);background:rgba(15,26,34,.85)}
.lp-sel2.lp-dis{opacity:.5;pointer-events:none}
.lp-sel2-n{width:24px;height:24px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;background:var(--card2);color:var(--text2);font:600 11px/1 'JetBrains Mono',monospace;transition:background-color .3s,color .3s}
.lp-sel2-n.lp-ok{background:var(--accent);color:var(--on)}
.lp-sel2 small{display:block;font-size:10.5px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}
.lp-sel2 b{display:block;margin-top:1px;font-size:14px;font-weight:700}
.lp-sel2 b.lp-ph{color:var(--faint);font-weight:600}
.lp-sel2 .lp-chev{margin-left:auto;color:var(--muted);transition:transform .3s var(--spring)}
.lp-sel2.lp-on .lp-chev{transform:rotate(180deg)}
.lp-qs{flex:1 1 280px;display:flex;align-items:center;gap:10px;height:56px;padding:0 6px 0 14px;border-radius:12px;background:rgba(15,26,34,.55);border:1px solid transparent;color:var(--muted);transition:border-color .2s,box-shadow .25s}
.lp-qs:focus-within{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint)}
.lp-qs input{flex:1;min-width:0;height:100%;border:0;background:transparent;color:var(--text);font:inherit;font-size:13.5px;outline:none}
.lp-qs input::placeholder{color:var(--faint)}
.lp-hero0{display:flex;flex-direction:column;align-items:center;text-align:center;gap:14px;padding:70px 20px 80px}
.lp-hero0-t{font-size:30px;font-weight:300;letter-spacing:-.025em;max-width:640px}
.lp-hero0-s{font-size:13.5px;color:var(--muted);max-width:520px}
.lp-rb{margin-top:24px;display:grid;grid-template-columns:minmax(320px,390px) minmax(0,1fr);gap:18px;align-items:start}
.lp-rp{position:sticky;top:0;display:flex;flex-direction:column;gap:22px;padding:22px;border-radius:18px;border:1px solid var(--border);background:linear-gradient(180deg,rgba(21,39,51,.78),rgba(17,30,39,.55))}
.lp-tgl{display:flex;flex-wrap:wrap;gap:6px}
.lp-tg{display:inline-flex;align-items:center;gap:7px;height:32px;padding:0 12px 0 9px;border-radius:9px;border:1px solid var(--btn);background:transparent;color:var(--muted);font-size:12.5px;font-weight:700;transition:border-color .2s,background-color .2s,color .2s,transform .3s var(--spring)}
.lp-tg:active{transform:scale(.95)}
.lp-tg .lp-ckb{width:15px;height:15px;border-radius:4px}
.lp-tg.lp-on{border-color:rgba(45,212,191,.45);background:var(--tint);color:var(--accent2)}
.lp-tg.lp-on .lp-ckb{background:var(--accent);border-color:var(--accent);color:var(--on)}
.lp-rp-f{display:flex;flex-direction:column;gap:8px}
.lp-rp-f .lp-btn{justify-content:center}
.lp-rv{min-width:0}
.lp-rv-h{display:flex;align-items:center;gap:14px;flex-wrap:wrap;padding:18px 18px 14px 22px}
.lp-rv-h h3{font-size:15px;font-weight:700}
.lp-rv-h p{font-size:12.5px;color:var(--muted)}
.lp-more{display:flex;justify-content:center;gap:10px;padding:14px}
.lp-prm{margin-top:24px;border-radius:18px;border:1px solid var(--border);background:linear-gradient(180deg,rgba(21,39,51,.78),rgba(17,30,39,.55))}
.lp-prm-r{display:flex;align-items:flex-end;gap:12px;flex-wrap:wrap;padding:18px}
.lp-fld{display:flex;flex-direction:column;gap:7px;min-width:200px}
.lp-fld>small{display:flex;align-items:center;gap:7px;font-size:10.5px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}
.lp-fld .lp-sel2{height:46px;min-width:220px}
.lp-bseg{display:grid;grid-template-columns:1fr 1fr;position:relative;width:280px;padding:3px;border-radius:12px;background:rgba(15,26,34,.55);border:1px solid rgba(255,255,255,.06)}
.lp-bseg .lp-ind{position:absolute;top:3px;bottom:3px;left:3px;width:calc((100% - 6px) / 2);border-radius:9px;background:var(--tint);border:1px solid rgba(45,212,191,.35);transition:transform .5s var(--spring)}
.lp-bseg button{position:relative;z-index:1;height:38px;border:0;background:transparent;color:var(--muted);font-size:13px;font-weight:700;display:flex;align-items:center;justify-content:center;gap:7px;transition:color .2s}
.lp-bseg button.lp-on{color:var(--accent2)}
.lp-prm-f{display:flex;align-items:center;gap:10px;flex-wrap:wrap;padding:14px 18px;border-top:1px solid var(--hair)}
.lp-prm-f .lp-hintx{font-size:12.5px;color:var(--muted)}
.lp-mini{display:inline-flex;align-items:center;gap:6px;height:32px;padding:0 10px;border-radius:9px;border:1px solid var(--btn);background:var(--surface);color:var(--text2);font:600 11.5px/1 'JetBrains Mono',monospace}
.lp-ids{margin:0 18px 16px;display:flex;flex-direction:column;gap:6px;animation:lp-rise .4s var(--ease)}
.lp-ids textarea{min-height:62px;padding:10px 12px;border-radius:10px;border:1px solid var(--btn);background:rgba(15,26,34,.6);color:var(--text);font:500 12.5px/1.5 'JetBrains Mono',monospace;resize:vertical;outline:none}
.lp-ids textarea:focus{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint)}
.lp-ids small{font-size:11.5px;color:var(--faint)}
.lp-fet{margin-top:18px;padding:44px 20px 50px;border-radius:18px;border:1px solid var(--border);background:rgba(17,30,39,.5);display:flex;flex-direction:column;align-items:center;gap:16px;text-align:center}
.lp-fet-t{font-size:22px;font-weight:300;letter-spacing:-.02em}
.lp-fet-s{font-size:12.5px;color:var(--muted)}
.lp-fsteps{display:flex;align-items:center;gap:10px;flex-wrap:wrap;justify-content:center}
.lp-fstep{display:flex;align-items:center;gap:8px;height:32px;padding:0 12px;border-radius:999px;border:1px solid var(--border);color:var(--faint);font-size:12.5px;font-weight:700;transition:color .3s,border-color .3s,background-color .3s}
.lp-fstep.lp-now{color:var(--draft);border-color:rgba(246,193,119,.4);background:rgba(246,193,119,.08)}
.lp-fstep.lp-done{color:var(--accent2);border-color:rgba(45,212,191,.35);background:var(--tint)}
.lp-fstep .lp-dot{width:7px;height:7px;border-radius:50%;background:currentColor}
.lp-fstep.lp-now .lp-dot{animation:lp-blink 1s ease-in-out infinite}
@keyframes lp-blink{50%{opacity:.25}}
.lp-ibar{width:min(420px,90%);height:3px;border-radius:2px;background:rgba(255,255,255,.07);overflow:hidden}
.lp-ibar i{display:block;height:100%;width:var(--p);background:linear-gradient(90deg,#1E8F82,#5EEAD4);border-radius:2px;transition:width .3s linear}
.lp-rs{margin-top:18px}
.lp-vseg{display:grid;grid-template-columns:1fr 1fr;position:relative;padding:4px;border-radius:13px;background:rgba(255,255,255,.035);border:1px solid rgba(255,255,255,.07)}
.lp-vseg .lp-ind{position:absolute;top:4px;bottom:4px;left:4px;width:calc((100% - 8px) / 2);border-radius:9px;background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.12);transition:transform .5s var(--spring)}
.lp-vseg button{position:relative;z-index:1;height:34px;padding:0 18px;border:0;background:transparent;color:var(--muted);font-size:13px;font-weight:700;display:flex;align-items:center;gap:8px;transition:color .2s}
.lp-vseg button.lp-on{color:var(--text)}
.lp-dd{display:inline-flex;align-items:center;gap:8px;height:34px;padding:0 10px 0 12px;border-radius:9px;border:1px solid var(--btn);background:rgba(15,26,34,.55);color:var(--text2);font-size:12.5px;font-weight:600;transition:border-color .2s}
.lp-dd:hover{border-color:#3A5A6C;color:var(--text)}
.lp-dd small{color:var(--faint);font-weight:700;font-size:11px}
.lp-cgs{display:flex;align-items:center;gap:6px;flex-wrap:wrap;padding:0 18px 14px 22px}
.lp-cgs-l{font-size:11px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--faint);margin-right:4px}
.lp-gw{position:relative;overflow:auto;max-height:560px;border-top:1px solid var(--hair)}
.lp-gx{width:max-content;min-width:100%;font-size:12.5px}
.lp-gxr{display:grid;grid-template-columns:var(--gc);align-items:stretch}
.lp-gxc{display:flex;align-items:center;min-width:0;padding:0 14px;border-bottom:1px solid rgba(30,50,64,.65);color:var(--text2);white-space:nowrap}
.lp-gxh1{position:sticky;top:0;z-index:3}
.lp-gxh1 .lp-gxc{height:34px;background:#13232E;font-size:10.5px;font-weight:800;letter-spacing:.09em;text-transform:uppercase;color:var(--text2);border-bottom:1px solid var(--hair);gap:8px}
.lp-gxh1 .lp-gxc i{width:8px;height:8px;border-radius:2px;background:var(--c)}
.lp-gxh1 .lp-gxc.lp-gs{box-shadow:inset 0 2px 0 var(--c)}
.lp-gxh2{position:sticky;top:34px;z-index:3}
.lp-gxh2 .lp-gxc{height:38px;background:#13232E;font-size:10.5px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;color:var(--muted)}
.lp-gxh2 .lp-gxc.lp-c{justify-content:center;text-align:center;white-space:normal;line-height:1.2}
.lp-gxb .lp-gxr{animation:lp-row .5s var(--ease) backwards;animation-delay:calc(var(--i) * 40ms + 80ms)}
.lp-gxb .lp-gxc{min-height:52px;padding-top:6px;padding-bottom:6px;transition:background-color .15s}
.lp-gxb .lp-gxr:hover .lp-gxc{background:var(--row-h)}
.lp-gxc.lp-gs{border-left:1px solid var(--hair)}
.lp-gxc.lp-c{justify-content:center}
.lp-gxc.lp-stk{position:sticky;left:0;z-index:2;background:#12212B;box-shadow:inset -1px 0 0 var(--hair)}
.lp-gxh1 .lp-gxc.lp-stk,.lp-gxh2 .lp-gxc.lp-stk{z-index:4;background:#13232E}
.lp-gxb .lp-gxr:hover .lp-gxc.lp-stk{background:#1A3140}
.lp-who{display:flex;align-items:center;gap:10px;min-width:0}
.lp-who b{display:block;color:var(--text);font-weight:700;font-size:13px}
.lp-who small{display:block;font:500 11px/1.2 'JetBrains Mono',monospace;color:var(--muted)}
.lp-who .lp-mgx{width:30px;height:30px;font-size:10px;border-radius:8px}
.lp-ent{display:flex;flex-direction:column;gap:4px;padding:8px 0}
.lp-yn{display:inline-flex;align-items:center;height:22px;padding:0 8px;border-radius:6px;background:rgba(255,255,255,.05);color:var(--muted);font-size:11.5px;font-weight:700}
.lp-yn.lp-y{background:rgba(246,193,119,.12);color:var(--draft)}
@media (max-width:900px){.lp-pg{padding:22px 16px 60px}.lp-pt{font-size:28px}.lp-bign{display:none}.lp-sr,.lp-sr:focus-within{width:100%}.lp-cd-h .lp-tl{margin-left:0;width:100%}.lp-rb{grid-template-columns:minmax(0,1fr)}.lp-rp{position:static}.lp-toast{left:50%}.lp-tsep{display:none}}
.lp-sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0 0 0 0);white-space:nowrap;border:0}
.lp-anc.lp-blk{display:flex}
.lp-anc.lp-blk .lp-sel2{width:100%;min-width:0}
.lp-anc.lp-blk .lp-pop{width:100%}
.lp-scx{display:flex;align-items:flex-end;gap:14px 18px;flex-wrap:wrap;padding:18px 22px;border-bottom:1px solid var(--hair);border-radius:18px 18px 0 0;background:linear-gradient(180deg,rgba(45,212,191,.05),rgba(45,212,191,0))}
.lp-scx-f{display:flex;flex-direction:column;gap:7px;width:380px;max-width:100%}
.lp-scx-l{font-size:10.5px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}
.lp-scf{display:flex;align-items:center;gap:12px;width:100%;height:50px;padding:0 14px 0 8px;border-radius:12px;border:1px solid var(--btn);background:rgba(15,26,34,.7);color:var(--text);text-align:left;transition:border-color .2s,box-shadow .25s,background-color .2s}
.lp-scf:hover{border-color:#3A5A6C;background:rgba(15,26,34,.9)}
.lp-scf.lp-on{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint)}
.lp-scf b{flex:1;min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;font-size:14.5px;font-weight:700}
.lp-scf .lp-chev{color:var(--muted);transition:transform .35s var(--spring),color .2s}
.lp-scf.lp-on .lp-chev{transform:rotate(180deg);color:var(--accent2)}
.lp-scx-m{display:flex;align-items:center;gap:8px;flex-wrap:wrap;min-height:50px}
.lp-kv3{display:inline-flex;align-items:center;gap:8px;height:34px;padding:0 12px;border-radius:9px;background:rgba(255,255,255,.035);border:1px solid rgba(255,255,255,.06)}
.lp-kv3 small{font-size:11px;font-weight:700;color:var(--muted)}
.lp-kv3 b{font:600 12px/1 'JetBrains Mono',monospace;color:var(--text)}
.lp-scx-n{margin-left:auto;align-self:center;font-size:12.5px;color:var(--muted)}
.lp-pop.lp-scp{width:100%;padding:8px}
.lp-psr{display:flex;align-items:center;gap:8px;height:38px;margin-bottom:2px;padding:0 10px;border-radius:9px;border:1px solid var(--btn);background:var(--surface);color:var(--muted);transition:border-color .2s,box-shadow .2s}
.lp-psr:focus-within{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint)}
.lp-psr input{flex:1;min-width:0;height:100%;border:0;background:transparent;color:var(--text);font:inherit;font-size:13px;outline:none}
.lp-psr input::placeholder{color:var(--faint)}
.lp-psr input::-webkit-search-cancel-button{display:none}
.lp-scp .lp-opt .lp-lc{margin-left:auto}
.lp-scp .lp-opt .lp-ok{margin-left:4px}
@media (max-width:900px){.lp-scx-f{width:100%}.lp-scx-n{margin-left:0}}
.lp-nwm{width:340px}
.lp-nwi{display:flex;align-items:center;gap:12px;width:100%;padding:10px;border:0;border-radius:11px;background:transparent;color:var(--text);text-align:left;transition:background-color .15s;animation:lp-rise .35s var(--ease) backwards;animation-delay:calc(var(--i) * 45ms)}
.lp-nwi:hover{background:rgba(255,255,255,.05)}
.lp-nwi.lp-on{background:var(--tint)}
.lp-nwi b{display:block;font-size:13.5px;font-weight:700}
.lp-nwi small{display:block;margin-top:1px;font-size:11.5px;color:var(--muted)}
.lp-nwi-ic{width:36px;height:36px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:10px;background:var(--mg-bg);border:1px solid var(--mg-b);color:var(--accent2);transition:transform .4s var(--spring)}
.lp-nwi:hover .lp-nwi-ic{transform:scale(1.08)}
.lp-nw-sep{height:1px;margin:6px 8px;background:var(--hair)}
.lp-btn .lp-chev2{margin-left:-2px;transition:transform .35s var(--spring)}
.lp-btn.lp-on .lp-chev2{transform:rotate(180deg)}
.lp-btn-p.lp-on{background:var(--accent2)}
.lp-scp .lp-opt .lp-lc{height:auto;padding:0;background:none;border:0}
.lp-btn:hover{transform:translateY(-1px)}
.lp-btn:active{transform:scale(.96);transition-duration:.08s}
.lp-btn-p{overflow:hidden}
.lp-btn-p::after{content:"";position:absolute;inset:0;background:linear-gradient(105deg,transparent 30%,rgba(255,255,255,.55) 50%,transparent 70%);transform:translateX(-130%);pointer-events:none}
.lp-btn-p:hover::after{transform:translateX(130%);transition:transform .8s var(--ease)}
.lp-btn svg{transition:transform .4s var(--spring)}
.lp-btn:hover .lp-pl{transform:rotate(90deg)}
.lp-btn-p.lp-on .lp-pl{transform:rotate(45deg)}
.lp-btn:hover .lp-dl{transform:translateY(2px)}
.lp-btn.lp-ok{border-color:rgba(45,212,191,.5);color:var(--accent2);background:var(--tint)}
.lp-okc path{stroke-dasharray:24;stroke-dashoffset:24;animation:lp-draw .45s .05s var(--ease) forwards}
@keyframes lp-draw{to{stroke-dashoffset:0}}
.lp-ibx svg{transition:transform .45s var(--spring)}
.lp-ibx:hover svg{transform:scale(1.1)}
.lp-ibx[aria-label="Refresh"]:hover svg{transform:rotate(140deg)}
.lp-ibx:active{transform:scale(.92)}
.lp-gt button:active,.lp-seg button:active,.lp-bseg button:active,.lp-vseg button:active{transform:scale(.96)}
.lp-fb-n{display:inline-block;transform-origin:left bottom}
.lp-fb.lp-on .lp-fb-n{animation:lp-bump .5s var(--spring)}
.lp-fb:active .lp-fb-n{transform:scale(.92)}
@keyframes lp-bump{0%{transform:scale(.88)}55%{transform:scale(1.14)}100%{transform:none}}
.lp-sr>svg{transition:transform .3s var(--spring)}
.lp-sr:focus-within>svg{animation:lp-nudge .55s var(--spring)}
@keyframes lp-nudge{30%{transform:rotate(-16deg) scale(1.15)}65%{transform:rotate(8deg)}100%{transform:none}}
.lp-hits{flex-shrink:0;height:22px;padding:0 8px;border-radius:999px;background:var(--tint);color:var(--accent2);font:600 10.5px/22px 'JetBrains Mono',monospace;white-space:nowrap;animation:lp-pop .3s var(--spring)}
.lp-tw.lp-slr1{animation:lp-slr1 .55s var(--ease)}
.lp-tw.lp-slr2{animation:lp-slr2 .55s var(--ease)}
.lp-tw.lp-sll1{animation:lp-sll1 .55s var(--ease)}
.lp-tw.lp-sll2{animation:lp-sll2 .55s var(--ease)}
@keyframes lp-slr1{from{opacity:0;transform:translateX(30px)}to{opacity:1;transform:none}}
@keyframes lp-slr2{from{opacity:0;transform:translateX(30px)}to{opacity:1;transform:none}}
@keyframes lp-sll1{from{opacity:0;transform:translateX(-30px)}to{opacity:1;transform:none}}
@keyframes lp-sll2{from{opacity:0;transform:translateX(-30px)}to{opacity:1;transform:none}}
.lp-pin.lp-on svg{animation:lp-pindrop .6s var(--spring)}
@keyframes lp-pindrop{0%{transform:translateY(-10px) scale(.5);opacity:0}60%{transform:translateY(1px) scale(1.18);opacity:1}100%{transform:none}}
.lp-tbd .lp-tr.lp-sel{animation:lp-selin .5s var(--ease)}
@keyframes lp-selin{from{box-shadow:inset 0 0 0 var(--accent),0 0 0 rgba(45,212,191,0)}to{box-shadow:inset 3px 0 0 var(--accent)}}
.lp-nmb:active .lp-mgx{transform:scale(.88)}
.lp-rab{transition:background-color .2s,color .2s,transform .3s var(--spring)}
.lp-rab:hover{transform:translateY(-1px)}
.lp-rab:active,.lp-ola:active{transform:scale(.88)}
.lp-ckb svg path{stroke-dasharray:22;stroke-dashoffset:22;transition:stroke-dashoffset .35s var(--ease)}
.lp-cko.lp-on .lp-ckb svg path,.lp-tg.lp-on .lp-ckb svg path{stroke-dashoffset:0}
.lp-cko.lp-on .lp-ckb,.lp-tg.lp-on .lp-ckb{animation:lp-ckpop .4s var(--spring)}
@keyframes lp-ckpop{0%{transform:scale(.7)}60%{transform:scale(1.18)}100%{transform:none}}
.lp-pop{background:rgba(23,42,54,.94);-webkit-backdrop-filter:blur(12px);backdrop-filter:blur(12px)}
.lp-toast{overflow:hidden}
.lp-tck{width:22px;height:22px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;background:#0E9F8D}
.lp-toast .lp-tck svg{color:#fff}
.lp-toast.lp-show .lp-tck{animation:lp-tckpop .5s .08s var(--spring) backwards}
.lp-tck path{stroke-dasharray:24;stroke-dashoffset:24}
.lp-toast.lp-show .lp-tck path{animation:lp-draw .4s .25s var(--ease) forwards}
.lp-tbar{position:absolute;left:0;right:0;bottom:0;height:2px;background:#0E9F8D;opacity:.45;transform-origin:left}
.lp-toast.lp-show .lp-tbar{animation:lp-tcount 2.6s linear forwards}
@keyframes lp-tcount{from{transform:scaleX(1)}to{transform:scaleX(0)}}
@keyframes lp-tckpop{0%{transform:scale(0)}100%{transform:none}}
.lp-drop:active{transform:scale(.98)}
.lp-drop-ic svg{transition:transform .35s var(--spring)}
.lp-drop:hover .lp-drop-ic svg{transform:translateY(-4px)}
.lp-opt,.lp-nwi,.lp-xi,.lp-vw-b,.lp-cko{transition:background-color .15s,transform .25s var(--spring)}
.lp-opt:active,.lp-nwi:active,.lp-xi:active,.lp-cko:active{transform:scale(.98)}
.lp-sel2:active,.lp-scf:active{transform:scale(.985)}
.lp-sel2-n.lp-ok{animation:lp-ckpop .45s var(--spring)}
.lp-colbar{display:flex;align-items:center;flex-wrap:wrap;gap:6px;padding:0 18px 14px 22px}
.lp-colbar-l{display:inline-flex;align-items:center;gap:7px;margin-right:6px;font-size:11px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--faint)}
.lp-colbar-l b{font:600 10.5px/1 'JetBrains Mono',monospace;letter-spacing:0;color:var(--muted)}
.lp-cch{display:inline-flex;align-items:center;gap:7px;height:30px;padding:0 12px 0 7px;border-radius:999px;border:1px dashed #34576A;background:transparent;color:var(--muted);font-size:12.5px;font-weight:600;white-space:nowrap;transition:background-color .25s,border-color .25s,color .25s,transform .3s var(--spring);animation:lp-rise .4s var(--ease) backwards;animation-delay:calc(var(--i) * 25ms + 200ms)}
.lp-cch:hover{border-color:#4B7186;color:var(--text);transform:translateY(-1px)}
.lp-cch:active{transform:scale(.95)}
.lp-cch.lp-on{border:1px solid rgba(45,212,191,.32);background:rgba(45,212,191,.08);color:var(--text)}
.lp-cch.lp-on:hover{border-color:rgba(45,212,191,.55)}
.lp-cc{width:18px;height:18px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;background:rgba(255,255,255,.06);color:var(--muted);transition:background-color .25s,color .25s}
.lp-cch.lp-on .lp-cc{background:var(--accent);color:var(--on);animation:lp-ckpop .4s var(--spring)}
.lp-cch.lp-on .lp-cc path{stroke-dasharray:22;stroke-dashoffset:22;animation:lp-draw .35s .05s var(--ease) forwards}
.lp-colrs{margin-left:6px;animation:lp-pop .3s var(--spring)}
.lp-tb.lp-cf1{animation:lp-cfade1 .45s var(--ease)}
.lp-tb.lp-cf2{animation:lp-cfade2 .45s var(--ease)}
@keyframes lp-cfade1{from{opacity:.35}to{opacity:1}}
@keyframes lp-cfade2{from{opacity:.35}to{opacity:1}}
.lp-ghost{position:relative;width:min(440px,90%);display:flex;flex-direction:column;gap:8px;margin-bottom:14px;-webkit-mask-image:linear-gradient(180deg,#000 30%,transparent);mask-image:linear-gradient(180deg,#000 30%,transparent)}
.lp-gh-r{display:flex;align-items:center;gap:12px;height:48px;padding:0 14px;border-radius:12px;border:1px solid rgba(255,255,255,.06);background:linear-gradient(90deg,rgba(27,49,64,.7),rgba(21,39,51,.35));animation:lp-rise .7s var(--ease) backwards,lp-ghfloat 6s ease-in-out infinite;animation-delay:calc(var(--i) * 120ms + 150ms),calc(var(--i) * -1.2s)}
.lp-gh-r:nth-child(2){margin-left:22px;margin-right:-22px}
.lp-gh-r:nth-child(3){margin-left:8px;margin-right:-8px}
@keyframes lp-ghfloat{50%{transform:translateY(-3px)}}
.lp-gh-av{width:26px;height:26px;flex-shrink:0;border-radius:8px;background:rgba(45,212,191,.16);border:1px solid rgba(45,212,191,.25)}
.lp-gh-l{height:8px;border-radius:4px;background:rgba(230,238,242,.14)}
.lp-gh-l.lp-s{background:rgba(230,238,242,.07)}
.lp-gh-p{margin-left:auto;width:62px;height:18px;flex-shrink:0;border-radius:999px;background:rgba(45,212,191,.12);border:1px solid rgba(45,212,191,.22)}
.lp-hero0{padding:48px 20px 70px}
.lp-fnd2{margin-top:22px;display:grid;grid-template-columns:minmax(0,1.4fr) auto minmax(0,1fr);align-items:stretch;border-radius:18px;border:1px solid var(--border);background:linear-gradient(180deg,rgba(21,39,51,.75),rgba(17,30,39,.55))}
.lp-fz{display:flex;flex-direction:column;gap:14px;padding:18px 20px 20px;min-width:0}
.lp-fz-h{display:flex;align-items:center;gap:12px}
.lp-fz-h b{display:block;font-size:14px;font-weight:700}
.lp-fz-h small{display:block;margin-top:1px;font-size:12px;color:var(--muted)}
.lp-fz-ic{width:34px;height:34px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:10px;background:var(--tint);border:1px solid var(--tint2);color:var(--accent2)}
.lp-fz-row{display:flex;align-items:center;gap:8px}
.lp-fz-row .lp-anc{flex:1 1 0;min-width:0}
.lp-fz-row .lp-sel2{min-width:0;width:100%}
.lp-fz-ln{flex-shrink:0;width:28px;height:28px;display:flex;align-items:center;justify-content:center;border-radius:50%;background:rgba(255,255,255,.04);color:var(--muted)}
.lp-fz-or{position:relative;width:1px;margin:18px 0;background:var(--hair)}
.lp-fz-or span{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);padding:5px 9px;border-radius:999px;border:1px solid var(--border);background:#132530;font-size:10.5px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--faint)}
.lp-fnd2 .lp-qs{flex:none;height:56px}
.lp-ctx{display:inline-flex;align-items:center;gap:10px;padding:5px 14px 5px 6px;border-radius:12px;border:1px solid var(--border);background:rgba(21,39,51,.6);animation:lp-rise .5s var(--ease) backwards;animation-delay:.15s}
.lp-ctx .lp-mgx{width:30px;height:30px;font-size:10.5px}
.lp-ctx b{display:block;font-size:13px;font-weight:700}
.lp-ctx small{display:block;font:500 11px/1.2 'JetBrains Mono',monospace;color:var(--muted)}
@media (max-width:900px){.lp-fnd2{grid-template-columns:minmax(0,1fr)}.lp-fz-or{width:auto;height:1px;margin:0 18px}}
.lp-rp{gap:0;padding:22px 22px 20px}
.lp-rs2{position:relative;display:flex;gap:14px;padding-bottom:24px}
.lp-rs2:not(.lp-end)::before{content:"";position:absolute;left:12px;top:30px;bottom:6px;width:1px;background:linear-gradient(180deg,rgba(45,212,191,.45),var(--hair))}
.lp-rs2-n{width:25px;height:25px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;border:1px solid var(--btn);background:var(--surface);color:var(--text2);font:600 11px/1 'JetBrains Mono',monospace;transition:background-color .3s,color .3s,border-color .3s}
.lp-rs2-n.lp-ok{background:var(--accent);border-color:var(--accent);color:var(--on);box-shadow:0 0 0 4px rgba(45,212,191,.12)}
.lp-rs2-b{flex:1;min-width:0;display:flex;flex-direction:column;gap:10px}
.lp-rs2-t{display:flex;align-items:center;gap:8px;min-height:25px;font-size:13.5px;font-weight:700}
.lp-rs2-t .lp-mono{font-size:11px;font-weight:500;color:var(--faint)}
.lp-rs2-t .lp-lk{margin-left:auto}
.lp-rs2-b .lp-sel2{height:50px}
.lp-fmts{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:8px}
.lp-fmt{display:flex;flex-direction:column;align-items:flex-start;gap:6px;padding:12px 10px 11px;border-radius:12px;border:1px solid var(--btn);background:rgba(15,26,34,.5);color:var(--text);text-align:left;transition:border-color .25s,background-color .25s,transform .3s var(--spring)}
.lp-fmt:hover{border-color:#3A5A6C;transform:translateY(-1px)}
.lp-fmt:active{transform:scale(.97)}
.lp-fmt.lp-on{border-color:rgba(45,212,191,.55);background:var(--tint);box-shadow:0 10px 24px -16px rgba(45,212,191,.6)}
.lp-fmt .lp-ft{width:34px;height:34px;font-size:9px;border-radius:9px;transition:transform .35s var(--spring)}
.lp-fmt.lp-on .lp-ft{transform:scale(1.06)}
.lp-fmt b{font-size:13px;font-weight:700}
.lp-fmt small{font-size:11px;color:var(--muted);line-height:1.3}
.lp-rp-f{flex-direction:row;padding-top:4px;border-top:1px solid var(--hair);padding-top:16px}
.lp-rp-f .lp-btn{flex:1}
.lp-doc-h{display:flex;align-items:center;gap:8px;flex-wrap:wrap;padding:16px 18px 14px 18px}
.lp-doc-tab{display:inline-flex;align-items:center;gap:10px;height:40px;padding:0 14px 0 6px;margin-right:6px;border-radius:11px;border:1px solid var(--border);background:rgba(15,26,34,.6)}
.lp-doc-tab .lp-ft{width:30px;height:30px;font-size:8.5px;border-radius:8px}
.lp-doc-tab b{font:600 13px/1 'JetBrains Mono',monospace;color:var(--text)}
.lp-pgrid{margin-top:24px;display:grid;grid-template-columns:minmax(0,1fr) 300px;gap:16px;align-items:stretch}
.lp-pgrid .lp-prm{margin-top:0}
.lp-prm-h{padding:18px 18px 0;font-size:11px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}
.lp-prm-r{align-items:flex-end}
.lp-prm-ln{margin-bottom:9px}
.lp-mxk{display:flex;flex-direction:column;align-items:flex-start;gap:6px;padding:18px;border-radius:18px;border:1px solid var(--border);background:linear-gradient(160deg,rgba(147,197,253,.08),rgba(17,30,39,.55) 55%)}
.lp-mxk .lp-ft{width:40px;height:40px;margin-bottom:6px}
.lp-mxk-t{font-size:15px;font-weight:700}
.lp-mxk-s{font-size:12.5px;color:var(--muted)}
.lp-mxk-f{display:flex;align-items:center;gap:8px;margin-top:auto;padding-top:12px}
.lp-rhd{display:flex;align-items:center;gap:10px;flex-wrap:wrap;padding:14px 16px;border-radius:16px;border:1px solid var(--border);background:linear-gradient(180deg,rgba(21,39,51,.7),rgba(17,30,39,.45));margin-bottom:14px}
.lp-rhd-n{display:flex;align-items:baseline;gap:10px;margin-right:10px;padding-right:18px;border-right:1px solid var(--hair)}
.lp-rhd-n .lp-rhd-b{font-size:34px;font-weight:300;letter-spacing:-.03em;line-height:1;color:var(--accent2)}
.lp-rhd-n .lp-rhd-l{font-size:12.5px;font-weight:700;color:var(--muted)}
.lp-rs .lp-cd{margin-top:0}
.lp-xg{display:inline-flex;align-items:center;height:36px;border-radius:10px;border:1px solid var(--btn);background:rgba(17,30,39,.6);overflow:hidden}
.lp-xg-l{display:inline-flex;align-items:center;gap:6px;height:100%;padding:0 10px 0 12px;font-size:12.5px;font-weight:700;color:var(--muted);border-right:1px solid var(--btn)}
.lp-xg button{height:100%;padding:0 12px;border:0;border-right:1px solid rgba(43,69,86,.6);background:transparent;color:var(--text);font-size:12.5px;font-weight:700;transition:background-color .2s,color .2s}
.lp-xg button:last-child{border-right:0}
.lp-xg button:hover{background:var(--tint);color:var(--accent2)}
.lp-xg button:active{background:var(--tint2)}
@media (max-width:900px){.lp-pgrid{grid-template-columns:minmax(0,1fr)}.lp-prm-ln{display:none}}
.lp-rs2 .lp-tgl{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:6px}
.lp-rs2 .lp-tg{width:100%;min-width:0;padding:0 10px 0 8px;font-size:12px}
@media (max-width:900px){.lp-rs2 .lp-tgl{grid-template-columns:minmax(0,1fr)}}
/* ===== Table adapter: put these classes on YOUR existing table component (native <table> or a grid library). It never sets display, so resize, drag and sort keep working. ===== */
.lp-xt-wrap{position:relative;overflow:auto;border-top:1px solid var(--hair)}
.lp-xt{width:100%;border-collapse:separate;border-spacing:0;font-size:13px;color:var(--text2);background:transparent}
.lp-xt-th{position:relative;height:42px;padding:0 12px;text-align:left;vertical-align:middle;border-bottom:1px solid var(--hair);background:transparent;color:var(--muted);font-size:11px;font-weight:700;letter-spacing:.065em;text-transform:uppercase;white-space:nowrap;user-select:none;transition:color .2s,background-color .2s}
.lp-xt-th:hover{color:var(--text2);background:rgba(255,255,255,.02)}
.lp-xt-th.lp-on{color:var(--accent2)}
.lp-xt-th .lp-sa{display:inline-block;vertical-align:-2px;margin-left:6px;opacity:0;transition:transform .35s var(--spring),opacity .2s}
.lp-xt-th:hover .lp-sa{opacity:.6}
.lp-xt-th.lp-on .lp-sa{opacity:1}
.lp-xt-th.lp-desc .lp-sa{transform:rotate(180deg)}
.lp-xt-sticky .lp-xt-th{position:sticky;top:0;z-index:3;background:#13232E}
.lp-xt-rz{position:absolute;top:0;right:-4px;width:9px;height:100%;z-index:2;cursor:col-resize;touch-action:none}
.lp-xt-rz::after{content:"";position:absolute;left:4px;top:12px;bottom:12px;width:1px;border-radius:1px;background:var(--hair);transition:background-color .15s,top .2s var(--ease),bottom .2s var(--ease),box-shadow .2s}
.lp-xt-th:hover .lp-xt-rz::after{background:var(--btn)}
.lp-xt-rz:hover::after,.lp-xt-rz.lp-on::after{top:0;bottom:0;left:3.5px;width:2px;background:var(--accent);box-shadow:0 0 8px rgba(45,212,191,.6)}
.lp-xt.lp-resizing,.lp-xt.lp-resizing *{cursor:col-resize!important;user-select:none}
.lp-xt-th[draggable=true]{cursor:grab}
.lp-xt-th[draggable=true]:active{cursor:grabbing}
.lp-xt-th.lp-drag{opacity:.45}
.lp-xt-th.lp-drop-l{box-shadow:inset 2px 0 0 var(--accent)}
.lp-xt-th.lp-drop-r{box-shadow:inset -2px 0 0 var(--accent)}
.lp-xt-r{transition:background-color .16s}
.lp-xt-td{height:54px;padding:0 12px;vertical-align:middle;border-bottom:1px solid rgba(30,50,64,.65);color:var(--text2);font-size:13px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;transition:background-color .16s}
.lp-xt-click .lp-xt-r{cursor:pointer}
.lp-xt-r:hover>.lp-xt-td{background:var(--row-hs)}
.lp-xt-r.lp-sel>.lp-xt-td{background:var(--row-on)}
.lp-xt-r.lp-sel>.lp-xt-td:first-child{box-shadow:inset 3px 0 0 var(--accent)}
.lp-xt-r.lp-pinned>.lp-xt-td{background:rgba(45,212,191,.045)}
.lp-xt-r.lp-pinned:hover>.lp-xt-td{background:var(--row-h)}
.lp-xt-r.lp-flash>.lp-xt-td{animation:lp-flash 1.2s var(--ease)}
.lp-xt-body .lp-xt-r{animation:lp-row .55s var(--ease) backwards;animation-delay:calc(var(--i,0) * 38ms + 60ms)}
.lp-xt-body.lp-lb .lp-xt-r{animation-name:lp-row2}
.lp-xt-r:hover .lp-mgx{transform:scale(1.06);border-color:#3D6476}
.lp-xt-r:hover .lp-pin{opacity:1}
.lp-xt-r:hover .lp-nmt{color:#fff}
.lp-xt-num{width:56px;color:var(--faint);font:500 12px/1 'JetBrains Mono',monospace}
.lp-xt-th.lp-xt-num{font:700 11px/1 'Manrope',system-ui,sans-serif;letter-spacing:.065em}
.lp-xt-act{width:56px;text-align:right}
.lp-xt-stk{position:sticky;left:0;z-index:2;background:#12212B;box-shadow:inset -1px 0 0 var(--hair)}
.lp-xt-th.lp-xt-stk{z-index:4;background:#13232E}
.lp-xt-r:hover>.lp-xt-stk{background:#1A3140}
.lp-xt-gh{height:34px;padding:0 14px;text-align:left;vertical-align:middle;background:#13232E;border-bottom:1px solid var(--hair);border-left:1px solid var(--hair);box-shadow:inset 0 2px 0 var(--c,transparent);color:var(--text2);font-size:10.5px;font-weight:800;letter-spacing:.09em;text-transform:uppercase;white-space:nowrap}
.lp-xt-gh i{display:inline-block;width:8px;height:8px;margin-right:8px;border-radius:2px;background:var(--c)}
.lp-xt-gs{border-left:1px solid var(--hair)}
.lp-xt-dense .lp-xt-th{height:38px;font-size:10.5px;letter-spacing:.06em}
.lp-xt-dense .lp-xt-td{height:auto;padding-top:6px;padding-bottom:6px;font-size:12.5px;line-height:1.35}
.lp-xt-dense .lp-xt-r>.lp-xt-td{height:52px}
.lp-xt-wrap.lp-xt-tall{max-height:560px}
.lp-xt-wrap.lp-slr1{animation:lp-slr1 .55s var(--ease)}
.lp-xt-wrap.lp-slr2{animation:lp-slr2 .55s var(--ease)}
.lp-xt-wrap.lp-sll1{animation:lp-sll1 .55s var(--ease)}
.lp-xt-wrap.lp-sll2{animation:lp-sll2 .55s var(--ease)}
.lp-xt.lp-cf1{animation:lp-cfade1 .45s var(--ease)}
.lp-xt.lp-cf2{animation:lp-cfade2 .45s var(--ease)}
.lp-xt-r.lp-sel>.lp-xt-td:first-child{animation:lp-selin .5s var(--ease)}
.lp-xt-grp{padding:0!important;height:auto!important;border-bottom:0}
.lp-xt-sticky .lp-xt-gh{position:sticky;top:0;z-index:3}
.lp-xt-sticky .lp-xt-gh.lp-xt-stk{left:0;z-index:5}
.lp-xt-grouped.lp-xt-sticky .lp-xt-th{top:34px}
.lp-more>span{align-self:center;margin-right:6px;font-size:12.5px;color:var(--muted)}
.lp-btn:disabled{opacity:.45;pointer-events:none}
select.lp-mini{cursor:pointer}
.lp-err{margin-top:8px;font-size:12.5px;font-weight:600;color:var(--err)}
/* Column chips: drag to reorder (only if your column menu can reorder today) */
.lp-cch[draggable=true]{cursor:grab}
.lp-cch.lp-drag{opacity:.45}
.lp-cch.lp-drop-l{box-shadow:-3px 0 0 var(--accent)}
.lp-cch.lp-drop-r{box-shadow:3px 0 0 var(--accent)}
.lp-cch.lp-req{cursor:default;opacity:.75}
/* Info tooltip trigger next to a filter label (FMID Reports) */
.lp-inf{display:inline-flex;align-items:center;justify-content:center;width:18px;height:18px;margin-left:2px;border:0;border-radius:50%;padding:0;background:transparent;color:var(--faint);transition:color .2s,background-color .2s}
.lp-inf:hover,.lp-inf:focus-visible{color:var(--accent2);background:var(--tint)}
.lp-req-star{color:var(--err);margin-left:3px;font-weight:700}
@media (prefers-reduced-motion:reduce){.lp-root *,.lp-root *::before,.lp-root *::after{animation-duration:.01ms!important;animation-iteration-count:1!important;transition-duration:.01ms!important}}
```

---

# Appendix B — Markup

Read section 1 first. Value names such as `{uItamT}` or `pFetch` come from my mockup; map each to your own state and handlers as the notes say. Wording inside the markup is mockup wording: wherever the app has its own text, the app's text wins (section 0.3). Where a comment says "YOUR EXISTING …", keep your component and only add the classes.

## B.1 Page wrapper

Wraps the content area of each of the seven pages (section 3.1). `{lp-pga|lp-pgb}`: see 3.1. `anyPopoverOpen` / `closePopover`: your popover state.

```html
<div class="lp-root">
  <div class="lp-pg">
    <div class="lp-aur" aria-hidden="true">
      <i class="lp-a1"></i><i class="lp-a2"></i>
    </div>
    <div class="{lp-pga|lp-pgb}">
      <!-- if anyPopoverOpen -->
        <button type="button" class="lp-pscrim" aria-label="Close" onClick={closePopover}>
        </button>
      <!-- /if -->
      <!-- B.2 page header, then the page's own blocks: B.3 / B.4 / B.8 / B.9 / B.10 -->
    </div>
  </div>
  <!-- B.7 upload dialog (pages that have Upload) -->
  <!-- B.11 toast -->
</div>
```

## B.2 Page header, Presets popover, New menu

Section 3. `{bign}` and `{pn}` = module number (`01`–`07`). `{pTitle}` / `{pSub}` = the app's title and subtitle. `{synced}` = the app's "Updated …" text; `syncGo` = existing refresh. `hasView` = page has Presets; `{viewT}` = applied preset name; `{viewN}` = number of saved presets; `views` = your presets (`v.n` name, `v.sub` meta, `v.pick` apply, `v.star` set default, `v.del` delete, `v.canDel`). `vSaved`/`vNot` = the 1.5s "Saved" flash. `isUpload` = Data Policy Management. `hasNew` = a page with a single create button (`{newL}` = its existing label). `isAppNew` = Application Management's New dropdown: `newItems` = your existing items (`n.t` title, `n.d` description, `n.isApps/isFeats/isActs` picks the icon), `itamOK` = your existing role check.

```html
<header class="lp-ph lp-rz" style="--d: 0">
  <span class="lp-bign" aria-hidden="true">{bign}</span>
  <div class="lp-ph-l">
    <p class="lp-eb">
      <b>{pn}</b><i></i>FMCES
    </p>
    <h1 class="lp-pt">{pTitle}</h1>
    <p class="lp-ps">
      <span>{pSub}</span>
      <!-- if hasSync -->
        <button type="button" class="{lp-sync [lp-busy]}" onClick={syncGo} title="Refresh now">
          <span class="lp-pulse"></span>{synced}
        </button>
      <!-- /if -->
    </p>
  </div>
  <div class="lp-pa">
    <!-- if hasView -->
      <div class="lp-anc">
        <button type="button" class="{lp-btn lp-btn-g [lp-on]}" onClick={viewGo} aria-haspopup="dialog" aria-expanded={viewOpen}>
          <Icon name="layers" size="16"/>Presets<span class="lp-sub">{viewT}</span><span class="lp-bdg">{viewN}</span><Icon name="chevron-down" size="14" bold/>
        </button>
        <!-- if viewOpen -->
          <div class="lp-pop" role="dialog" aria-label="Presets">
            <p class="lp-pop-h">
              <Icon name="layers" size="15"/>Presets
            </p>
            <div class="lp-pin2">
              <input type="text" ref={vnRef} value={viewName} onChange={onVn} onKeyDown={vnKey} placeholder="Name this preset…" aria-label="Preset name">
              <button type="button" class="lp-btn lp-btn-p lp-btn-sm" onClick={saveView}>
                <!-- if vNot -->
                  Save
                <!-- /if -->
                <!-- if vSaved -->
                  <Icon name="check" size="14" bold class="lp-okc"/>Saved
                <!-- /if -->
              </button>
            </div>
            <!-- for each v in views -->
              <div class="{lp-vw [lp-on]}" style="--i: {v.i}">
                <button type="button" class="lp-vw-b" onClick={v.pick}>
                  <span class="lp-grow"><span style="display: block">{v.n}</span><small>{v.sub}</small></span>
                </button>
                <button type="button" class="{lp-vwi [lp-on]}" onClick={v.star} aria-label={v.starL} title={v.starL}>
                  <Icon name="star" size="15"/>
                </button>
                <!-- if v.canDel -->
                  <button type="button" class="lp-vwi lp-del" onClick={v.del} aria-label={v.delL}>
                    <Icon name="trash" size="15"/>
                  </button>
                <!-- /if -->
              </div>
            <!-- /for -->
            <p class="lp-pnote">The starred preset opens when you come to this page.</p>
          </div>
        <!-- /if -->
      </div>
    <!-- /if -->
    <!-- if isUpload -->
      <button type="button" class="lp-btn lp-btn-g" onClick={upGo}>
        <Icon name="upload" size="16"/>Upload
      </button>
    <!-- /if -->
    <!-- if hasNew -->
      <button type="button" class="lp-btn lp-btn-p" onClick={newGo}>
        <Icon name="plus" size="16" bold class="lp-pl"/>{newL}
      </button>
    <!-- /if -->
    <!-- if isAppNew -->
      <div class="lp-anc">
        <button type="button" class="{lp-btn lp-btn-p [lp-on]}" onClick={newMenuGo} aria-haspopup="menu" aria-expanded={newOpen}>
          <Icon name="plus" size="16" bold class="lp-pl"/>New<Icon name="chevron-down" size="14" bold class="lp-chev2"/>
        </button>
        <!-- if newOpen -->
          <div class="lp-pop lp-nwm" role="menu" aria-label="Create">
            <p class="lp-pop-h">Create</p>
            <!-- for each n in newItems -->
              <button type="button" class="{lp-nwi [lp-on]}" role="menuitem" onClick={n.go} style="--i: {n.i}">
                <span class="lp-nwi-ic">
                <!-- if n.isApps -->
                  <Icon name="grid" size="17"/>
                <!-- /if -->
                <!-- if n.isFeats -->
                  <Icon name="box" size="17"/>
                <!-- /if -->
                <!-- if n.isActs -->
                  <Icon name="zap" size="17"/>
                <!-- /if -->
                </span><span class="lp-grow"><b>{n.t}</b><small>{n.d}</small></span>
              </button>
            <!-- /for -->
            <!-- if itamOK -->
              <div class="lp-nw-sep">
              </div>
              <button type="button" class="lp-nwi" role="menuitem" onClick={newItam} style="--i: 3">
                <span class="lp-nwi-ic"><Icon name="database" size="17"/></span><span class="lp-grow"><b>New ITAM record</b><small>Add an ITAM record</small></span>
              </button>
            <!-- /if -->
          </div>
        <!-- /if -->
      </div>
    <!-- /if -->
  </div>
</header>
```

## B.3 Application Management tabs

Section 4. `amTabs` = the visible tabs in today's order (`t.t` name, `t.n` count [IF DATA EXISTS], `t.sel`, `t.go`). `{amN}` = number of visible tabs. `{amX}` = `translateX(index × 100%)`.

```html
<!-- if isApp -->
  <div class="lp-gtw lp-rz" style="--d: 1">
    <nav class="lp-gt" style="--n: {amN}" role="tablist" aria-label="App Management sections">
      <span class="lp-ind" style="transform: {amX}"></span>
      <!-- for each t in amTabs -->
        <button type="button" role="tab" class="{[lp-on]}" aria-selected={t.sel} onClick={t.go}>{t.t}<em>{t.n}</em></button>
      <!-- /for -->
    </nav>
  </div>
<!-- /if -->
```

## B.4 List card: application band, facts, toolbar, export menu, column chips, table, footer

Sections 5 and 6. `isScoped` = Features / Actions tabs (application band, section 5). `facts` / `dist` = section 6.1 (`f.n` count, `f.t` label, `f.c` colour, `f.hasDot`). `{q}`, `{ph}`, `onQ` = your search; `{hitsT}` = "N matches". `fOn` / `{fT}` = active fact chip. **The funnel filter button is not drawn here: add your existing one before `.lp-tsep`, styled `.lp-ibx` (6.2).** `refGo` = existing refresh. Export: `{exN}` = "N rows · N cols". `colChips` = all columns (`c.t`, `c.on`, `c.isOn`/`c.isOff`, `c.tog`); `{colN}` = visible count; `colChanged`/`colReset` only if a reset exists. The table block is YOUR table with adapter classes (6.4). Footer: `{foot}` = the app's footer text; `pers` = 15 / 25 / 50; the page buttons stand for your pagination.

```html
<!-- if showList -->
  <section class="lp-cd lp-rz" style="--d: 2" aria-label={nouns}>
    <!-- if isScoped -->
      <div class="lp-scx">
        <div class="lp-scx-f">
          <span class="lp-scx-l" id="scl">Application</span>
          <div class="lp-anc lp-blk">
            <button type="button" class="{lp-scf [lp-on]}" onClick={scGo} aria-haspopup="dialog" aria-expanded={scExp} aria-labelledby="scl">
              <span class="lp-mgx">{scIni}</span><b>{scName}</b><Icon name="chevron-down" size="16" bold class="lp-chev"/>
            </button>
            <!-- if scOpen -->
              <div class="lp-pop lp-l lp-scp" role="dialog" aria-label="Choose an application">
                <label class="lp-psr">
                  <Icon name="search" size="15"/>
                  <input type="search" value={scQ} onChange={onScQ} placeholder="Search applications or ITAM" aria-label="Search applications or ITAM">
                </label>
                <p class="lp-pop-g">Applications</p>
                <div role="listbox" aria-label="Applications">
                  <!-- for each o in scOpts -->
                    <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                      <span class="{lp-mgx [lp-dr]}">{o.ini}</span><span class="lp-grow"><b>{o.t}</b><small>{o.m}</small></span><span class="{lp-lc lp-lc-live|lp-lc-draft|lp-lc-pend|lp-lc-off}"><i></i></span>
                      <!-- if o.on -->
                        <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                      <!-- /if -->
                    </button>
                  <!-- /for -->
                </div>
                <!-- if scNone -->
                  <p class="lp-pnote">No application matches “{scQ}”.</p>
                <!-- /if -->
              </div>
            <!-- /if -->
          </div>
        </div>
        <div class="lp-scx-m">
          <span class="lp-kv3"><small>App UID</small><b>{scUid}</b></span><span class="lp-kv3"><small>ITAM</small><b>{scItam}</b></span><span class="lp-kv3"><small>Status</small><span class="{lp-lc lp-lc-live|lp-lc-draft|lp-lc-pend|lp-lc-off}"><i></i><span class="lp-lct">{scLcT}</span></span></span>
        </div>
        <p class="lp-scx-n">{scNote}</p>
      </div>
    <!-- /if -->
    <div class="lp-cd-h">
      <div class="lp-fx">
        <div class="lp-fx-r" role="group" aria-label="Filter by status">
          <!-- for each f in facts -->
            <button type="button" class="{lp-fb [lp-on] [lp-zero]}" onClick={f.pick}>
              <span class="lp-fb-n">{f.n}</span><span class="lp-fb-l">
              <!-- if f.hasDot -->
                <i class="lp-fb-d" style="--c: {f.c}"></i>
              <!-- /if -->
              {f.t}</span>
            </button>
          <!-- /for -->
        </div>
        <div class="lp-dist" aria-hidden="true">
          <!-- for each d in dist -->
            <i style="--c: {d.c}; --w: {d.w}; --k: {d.k}"></i>
          <!-- /for -->
        </div>
      </div>
      <div class="lp-tl">
        <label class="lp-sr">
          <Icon name="search" size="16"/>
          <input type="search" ref={srRef} value={q} onChange={onQ} placeholder={ph} aria-label={ph}>
          <!-- if hasQ -->
            <span class="lp-hits">{hitsT}</span>
          <!-- /if -->
          <!-- if hasQ -->
            <button type="button" class="lp-x" onClick={clearQ} aria-label="Clear search">
              <Icon name="close" size="12" bold/>
            </button>
          <!-- /if -->
          <!-- if noQ -->
            <kbd>/</kbd>
          <!-- /if -->
        </label>
        <!-- if fOn -->
          <span class="lp-fchip">{fT}
          <button type="button" onClick={fClear} aria-label="Clear the filter">
            <Icon name="close" size="12" bold/>
          </button>
          </span>
        <!-- /if -->
        <!-- YOUR EXISTING filter (funnel) button goes here: <button type="button" class="{lp-ibx [lp-on]}" aria-label={the app's label}>…</button> with its existing panel (section 6.2) -->
        <span class="lp-tsep"></span>
        <button type="button" class="{lp-ibx [lp-spin]}" onClick={refGo} aria-label="Refresh" title="Refresh">
          <Icon name="refresh" size="17"/>
        </button>
        <div class="lp-anc">
          <button type="button" class="{lp-btn lp-btn-g [lp-on] [lp-ok]}" onClick={exGo} aria-haspopup="menu" aria-expanded={exOpen}>
            <!-- if exNot -->
              <Icon name="download" size="16" class="lp-dl"/>Export<Icon name="chevron-down" size="14" bold class="lp-chev2"/>
            <!-- /if -->
            <!-- if exDone -->
              <Icon name="check" size="16" bold class="lp-okc"/>Exported
            <!-- /if -->
          </button>
          <!-- if exOpen -->
            <div class="lp-pop" role="menu" aria-label="Export">
              <p class="lp-pop-h">Export<span class="lp-mono">{exN}</span></p>
              <button type="button" class="lp-xi" role="menuitem" onClick={exCsv}>
                <span class="lp-ft lp-csv">CSV</span><span><b>CSV</b><small>Comma-separated values</small></span>
              </button>
              <button type="button" class="lp-xi" role="menuitem" onClick={exXls}>
                <span class="lp-ft lp-xls">XLSX</span><span><b>Excel</b><small>Microsoft Excel (.xlsx)</small></span>
              </button>
              <p class="lp-pnote">Only the rows and columns you can see are exported.</p>
            </div>
          <!-- /if -->
        </div>
      </div>
    </div>
    <div class="lp-colbar" role="group" aria-label="Columns">
      <span class="lp-colbar-l"><Icon name="columns" size="14"/>Columns<b>{colN}</b></span>
      <!-- for each c in colChips -->
        <button type="button" class="{lp-cch [lp-on]}" role="checkbox" aria-checked={c.on} onClick={c.tog} style="--i: {c.i}">
          <span class="lp-cc">
          <!-- if c.isOn -->
            <Icon name="check" size="11" bold/>
          <!-- /if -->
          <!-- if c.isOff -->
            <Icon name="plus" size="11" bold/>
          <!-- /if -->
          </span>{c.t}
        </button>
      <!-- /for -->
      <!-- if colChanged -->
        <button type="button" class="lp-lk lp-colrs" onClick={colReset}>
          <Icon name="refresh" size="13"/>Reset
        </button>
      <!-- /if -->
    </div>
    <!-- YOUR EXISTING TABLE COMPONENT, with the adapter classes (section 6.4). Native elements shown; use your library's equivalents. -->
    <div class="{lp-xt-wrap [lp-slr1|lp-slr2|lp-sll1|lp-sll2]}">
      <table class="{lp-xt lp-xt-click [lp-cf1|lp-cf2] [lp-resizing]}" aria-label={nouns}>
        <thead>
          <tr>
            <th class="lp-xt-th" title="Pinned rows stay on top"><Icon name="pin" size="14"/><span class="lp-sr-only">Pin</span></th>
            <th class="lp-xt-th lp-xt-num">#</th>
            <!-- for each h in your visible columns, in your order -->
              <th class="{lp-xt-th [lp-on] [lp-desc] [lp-drag] [lp-drop-l|lp-drop-r]}" aria-sort={h.ariaSort} draggable={your existing drag setting} style="width: {your existing width}">
                <!-- your existing sort click target -->{h.title}<Icon name="arrow-up" size="13" bold class="lp-sa"/>
                <span class="{lp-xt-rz [lp-on]}" aria-hidden="true"><!-- your existing resize handle + handlers --></span>
              </th>
            <!-- /for -->
            <th class="lp-xt-th lp-xt-act"><span class="lp-sr-only">Edit</span></th>
          </tr>
        </thead>
        <!-- if loading -->
          <tbody>
            <tr>
              <td class="lp-xt-grp" colspan={all columns}>
                <!-- SKELETON (B.6) -->
              </td>
            </tr>
          </tbody>
        <!-- /if -->
        <!-- if showRows -->
          <tbody class="{lp-xt-body [lp-lb]}">
            <!-- if pinned rows are grouped at the top today -->
              <tr>
                <td class="lp-xt-grp" colspan={all columns}>
                  <div class="lp-tgr"><Icon name="pin" size="13"/>Pinned <b>{pinnedCount}</b></div>
                </td>
              </tr>
              <!-- TABLE ROW (B.5), once for each pinned row -->
              <tr>
                <td class="lp-xt-grp" colspan={all columns}>
                  <div class="lp-tgr">{label for the other rows, e.g. "All {nouns}"}</div>
                </td>
              </tr>
            <!-- /if -->
            <!-- TABLE ROW (B.5), once for each row on this page -->
          </tbody>
        <!-- /if -->
        <!-- if empty -->
          <tbody>
            <tr>
              <td class="lp-xt-grp" colspan={all columns}>
                <div class="lp-emp">
                  <span class="lp-emp-ic"><Icon name="search" size="22"/></span>
                  <p class="lp-emp-t">{the app's empty title}</p>
                  <p class="lp-emp-s">{the app's empty sentence}</p>
                  <!-- if the emptiness comes from a search or a fact filter -->
                    <button type="button" class="lp-btn lp-btn-g lp-btn-sm" onClick={clearSearchOrFilter}>{Clear search | Show all}</button>
                  <!-- /if -->
                </div>
              </td>
            </tr>
          </tbody>
        <!-- /if -->
      </table>
    </div>
    <div class="lp-tf">
      <span>{foot}</span><span class="lp-grow"></span><span>Rows per page</span>
      <div class="lp-seg" role="group" aria-label="Rows per page">
        <!-- for each p in pers -->
          <button type="button" class="{[lp-on]}" onClick={p.go}>{p.t}</button>
        <!-- /for -->
      </div>
      <div class="lp-pgr">
        <button type="button" class="lp-pgb2 lp-dis" aria-label="Previous page">
          <Icon name="chevron-left" size="15"/>
        </button>
        <button type="button" class="lp-pgb2 lp-on" aria-current="page">1</button>
        <button type="button" class="lp-pgb2 lp-dis" aria-label="Next page">
          <Icon name="chevron-right" size="15"/>
        </button>
      </div>
    </div>
  </section>
<!-- /if -->
```

## B.5 Table row and cell contents (adapter)

Sections 6.4 and 6.5. One body row of your table. Each column gets exactly one of the cell contents shown, chosen by its type.

```html
<!-- One body row of YOUR table. Keep your row click (opens the drawer) and your existing cell renderers' values. -->
<tr class="{lp-xt-r [lp-sel] [lp-pinned] [lp-flash]}" style="--i: {index on this page}" data-k={record id} onClick={your existing open-drawer handler}>
  <td class="lp-xt-td">
    <button type="button" class="{lp-pin [lp-on]}" onClick={your existing pin handler (stops propagation)} aria-pressed={isPinned} aria-label={Pin | Unpin} title={Pin | Unpin}>
      <Icon name="pin" size="15"/>
    </button>
  </td>
  <td class="lp-xt-td lp-xt-num">{row number}</td>
  <!-- for each c in your visible columns: ONE of the cell contents below, chosen by column type (section 6.5) -->
    <td class="lp-xt-td" title={full text value}>
      <!-- PRIMARY NAME column -->
        <span class="lp-nmb">
          <span class="{lp-mgx [lp-dr|lp-pd] [lp-sv]}" aria-hidden="true">{initials} <!-- or <Icon name="user" size="15"/> / <Icon name="server" size="15"/> for accounts --></span>
          <span class="{lp-nmt [lp-mono]}">{name} <!-- client-side search: {before}<mark>{match}</mark>{after} --></span>
        </span>
      <!-- IDS, CODES, OWNERS, CII NUMBER, DLs -->
        <span class="lp-tt lp-mono">{value}</span>
      <!-- PLAIN TEXT / DESCRIPTION -->
        <span class="lp-tt">{value}</span>
      <!-- EMPTY VALUE (keep the app's marker, e.g. "—") -->
        <span class="lp-tt lp-dim">{empty marker}</span>
      <!-- STATUS -->
        <span class="{lp-lc lp-lc-live|lp-lc-draft|lp-lc-pend|lp-lc-off}"><i></i><span class="lp-lct">{the app's status word}</span>
          <!-- if the app shows a newer draft next to an approved version --><span class="lp-lc-x">{the app's text, e.g. v3 draft}</span><!-- /if -->
        </span>
      <!-- ACTIVE / ENABLED -->
        <span class="{lp-st [lp-off]}"><i></i>{the app's word}</span>
      <!-- VERSION -->
        <span class="lp-vr">{version}</span>
      <!-- COUNT -->
        <span class="lp-cn"><b class="{[lp-z]}">{count}</b></span>
      <!-- DATE (Onboard Date, Last modified, Created…) -->
        <span class="lp-dt"><b>{date, formatted exactly as today}</b><small>{relative, e.g. 3 days ago}</small></span>
      <!-- OLA -->
        <button type="button" class="lp-ola" onClick={your existing OLA handler (stops propagation)} aria-label={the app's OLA label} title={the app's OLA label}>
          <Icon name="doc" size="16"/>
        </button>
      <!-- CHIP(S): application names, roles -->
        <span class="lp-chps"><span class="lp-chp">{value}</span><span class="lp-chp">{value}</span></span>
      <!-- TYPE (User / Service) -->
        <span class="lp-ty"><Icon name="user" size="15"/>{the app's word}</span>
      <!-- YES / NO FLAG -->
        <span class="{lp-yn [lp-y]}">{the app's word}</span>
    </td>
  <!-- /for -->
  <td class="lp-xt-td lp-xt-act">
    <button type="button" class="lp-rab" onClick={your existing edit handler (stops propagation)} aria-label={Edit {name}} title="Edit">
      <Icon name="pencil" size="15"/>
    </button>
  </td>
</tr>
```

## B.6 Loading skeleton

Section 6.6. Six rows; `k.w` = 55%, 57%, 59%, 61%, 63%, 65% (row 0 to 5).

```html
<div aria-hidden="true">
  <!-- for each k in sk -->
    <div class="lp-skr">
      <span class="lp-sk" style="width: 16px"></span><span class="lp-sk lp-av"></span><span class="lp-sk" style="width: {k.w}"></span><span class="lp-sk"></span><span class="lp-sk" style="width: 70%"></span><span class="lp-sk" style="width: 60%"></span><span class="lp-sk" style="width: 50%"></span>
    </div>
  <!-- /for -->
</div>
```

## B.7 Upload dialog (Data Policy Management)

Section 7. `{upPT}` = the size / status text. `{upBtnT}` = the app's upload button label. `upBusy` / `{upP}` = upload progress [IF DATA EXISTS].

```html
<!-- if upOpen -->
  <div class="lp-dw">
    <button type="button" class="lp-dscrim" aria-label="Close" onClick={upClose}>
    </button>
    <section class="lp-dlg" role="dialog" aria-modal="true" aria-label="Upload data policies">
      <div class="lp-dlg-h">
        <span class="lp-dlg-ic"><Icon name="upload" size="20"/></span>
        <div>
          <h2>Upload data policies</h2>
          <p>Add or update many policies at once from an Excel file.</p>
        </div>
        <button type="button" class="lp-xclose" onClick={upClose} aria-label="Close">
          <Icon name="close" size="18"/>
        </button>
      </div>
      <div class="lp-dlg-b">
        <div class="lp-steps">
          <div class="lp-stp">
            <span class="lp-stp-n lp-ok">1</span>
            <div class="lp-grow">
              <b>Download the template</b><small>Keep its column headings as they are.</small>
            </div>
            <button type="button" class="lp-lk" onClick={tplGo}>
              <Icon name="download" size="14"/>{template file name}
            </button>
          </div>
          <div class="lp-stp">
            <span class="lp-stp-n lp-ok">2</span>
            <div class="lp-grow">
              <b>Fill in one policy per row</b><small>Each row becomes one policy. Save the file as .xlsx.</small>
            </div>
          </div>
          <div class="lp-stp">
            <span class="{lp-stp-n [lp-ok]}">3</span>
            <div class="lp-grow">
              <b>Upload it here</b>
              <!-- if upNoFile -->
                <button type="button" class="lp-drop" onClick={upPick}>
                  <span class="lp-drop-ic"><Icon name="upload" size="22"/></span><b>Drop your Excel file here</b><small>or <span class="lp-u">browse your computer</span> · .xlsx only</small>
                </button>
              <!-- /if -->
              <!-- if upHasFile -->
                <div class="lp-fil">
                  <span class="lp-ft lp-xls">XLSX</span>
                  <div class="lp-grow">
                    <b>{file name}</b><small>{upPT}</small>
                    <!-- if upBusy -->
                      <div class="lp-prog">
                        <i style="--p: {upP}"></i>
                      </div>
                    <!-- /if -->
                  </div>
                  <button type="button" class="lp-xclose" style="align-self: center" onClick={upRm} aria-label="Remove file">
                    <Icon name="close" size="16"/>
                  </button>
                </div>
              <!-- /if -->
            </div>
          </div>
        </div>
      </div>
      <div class="lp-dlg-f">
        <button type="button" class="lp-btn lp-btn-g" onClick={upClose}>Cancel</button>
        <button type="button" class="{lp-btn lp-btn-p [lp-dis]}" onClick={upGo}>
          <Icon name="upload" size="16" bold/>{upBtnT}
        </button>
      </div>
    </section>
  </div>
<!-- /if -->
```

## B.8 Entitlement Users: finder, start state, tabs, context card

Section 8. `uItams` / `uApps` = your ITAM and application options (`o.t` name, `o.m` meta, `o.on` selected, `o.go`). `{uItamT}` / `{uAppT}` = the chosen value or the app's placeholder. `uQ` / `uSearch` = your quick search. `uStart` = nothing chosen yet; `uLoading`; `uLoaded`. `uTabs` = Accounts / Entitlements with counts; `{uX}` = indicator transform. `ghost` = 4 decorative rows (`g.i` 0–3; `g.w1` 62%, 48%, 70%, 54%; `g.w2` 34%, 40%, 28%, 36%). The same ghost rows are used on Reports and FMID Reports. After this block, render the list card (B.4) for the active tab.

```html
<!-- if isUsers -->
  <section class="lp-fnd2 lp-rz" style="--d: 1" role="search" aria-label="Find user accounts">
    <div class="lp-fz">
      <p class="lp-fz-h">
        <span class="lp-fz-ic"><Icon name="grid" size="17"/></span><span><b>Browse by application</b><small>Pick the ITAM, then one of its applications.</small></span>
      </p>
      <div class="lp-fz-row">
        <div class="lp-anc lp-blk">
          <button type="button" class="{lp-sel2 [lp-on]}" onClick={uItamGo} aria-haspopup="listbox" aria-expanded={uItamOpen}>
            <span class="{lp-sel2-n [lp-ok]}">1</span><span><small>ITAM</small><b class="{[lp-ph]}">{uItamT}</b></span><Icon name="chevron-down" size="16" bold class="lp-chev"/>
          </button>
          <!-- if uItamOpen -->
            <div class="lp-pop lp-l" role="listbox" aria-label="ITAM">
              <!-- for each o in uItams -->
                <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                  <span class="lp-grow"><b>{o.t}</b>
                  <!-- if o.m -->
                    <small>{o.m}</small>
                  <!-- /if -->
                  </span>
                  <!-- if o.on -->
                    <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                  <!-- /if -->
                </button>
              <!-- /for -->
            </div>
          <!-- /if -->
        </div>
        <span class="lp-fz-ln" aria-hidden="true"><Icon name="chevron-right" size="16"/></span>
        <div class="lp-anc lp-blk">
          <button type="button" class="{lp-sel2 [lp-on] [lp-dis]}" onClick={uAppGo} aria-haspopup="listbox" aria-expanded={uAppOpen}>
            <span class="{lp-sel2-n [lp-ok]}">2</span><span><small>Application</small><b class="{[lp-ph]}">{uAppT}</b></span><Icon name="chevron-down" size="16" bold class="lp-chev"/>
          </button>
          <!-- if uAppOpen -->
            <div class="lp-pop lp-l" role="listbox" aria-label="Application">
              <!-- for each o in uApps -->
                <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                  <span class="lp-grow"><b>{o.t}</b>
                  <!-- if o.m -->
                    <small>{o.m}</small>
                  <!-- /if -->
                  </span>
                  <!-- if o.on -->
                    <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                  <!-- /if -->
                </button>
              <!-- /for -->
            </div>
          <!-- /if -->
        </div>
      </div>
    </div>
    <div class="lp-fz-or" aria-hidden="true">
      <span>or</span>
    </div>
    <div class="lp-fz">
      <p class="lp-fz-h">
        <span class="lp-fz-ic"><Icon name="search" size="17"/></span><span><b>Find one person</b><small>Search by owner or account name across every ITAM.</small></span>
      </p>
      <label class="lp-qs">
        <Icon name="search" size="17"/>
        <input type="search" value={uQ} onChange={onUQ} onKeyDown={uQKey} placeholder="Owner or account name" aria-label="Search accounts across all ITAMs">
        <button type="button" class="lp-btn lp-btn-p lp-btn-sm" onClick={uSearch}>Search</button>
      </label>
    </div>
  </section>
  <!-- if uLoaded -->
    <div class="lp-gtw lp-rz" style="--d: 2">
      <nav class="lp-gt" style="--n: 2" role="tablist" aria-label="Accounts or entitlements">
        <span class="lp-ind" style="transform: {uX}"></span>
        <!-- for each t in uTabs -->
          <button type="button" role="tab" class="{[lp-on]}" onClick={t.go}>{t.t}<em>{t.n}</em></button>
        <!-- /for -->
      </nav>
      <!-- if uCtxApp -->
        <span class="lp-ctx"><span class="lp-mgx">{uCtxIni}</span><span><b>{uCtxName}</b><small>{uCtxMeta}</small></span></span>
      <!-- /if -->
      <!-- if uCtxCross -->
        <span class="lp-ctx"><span class="lp-mgx"><Icon name="search" size="15"/></span><span><b>Results for “{uCtxQ}”</b><small>across all ITAMs</small></span></span>
      <!-- /if -->
    </div>
  <!-- /if -->
  <!-- if uStart -->
    <div class="lp-hero0 lp-rz" style="--d: 2">
      <div class="lp-ghost" aria-hidden="true">
        <!-- for each g in ghost -->
          <div class="lp-gh-r" style="--i: {g.i}">
            <span class="lp-gh-av"></span><span class="lp-gh-l" style="width: {g.w1}"></span><span class="lp-gh-l lp-s" style="width: {g.w2}"></span><span class="lp-gh-p"></span>
          </div>
        <!-- /for -->
      </div>
      <p class="lp-hero0-t">Pick an ITAM and an application to see who has access.</p>
      <p class="lp-hero0-s">Accounts and the entitlements they hold load as soon as you choose the application.</p>
    </div>
  <!-- /if -->
  <!-- if uLoading -->
    <section class="lp-cd" aria-label="Loading accounts">
      <div class="lp-cd-h">
        <span class="lp-sk" style="width: 260px; height: 30px"></span>
      </div>
      <div aria-hidden="true">
        <!-- for each k in sk -->
          <div class="lp-skr">
            <span class="lp-sk" style="width: 16px"></span><span class="lp-sk lp-av"></span><span class="lp-sk" style="width: {k.w}"></span><span class="lp-sk"></span><span class="lp-sk" style="width: 70%"></span><span class="lp-sk" style="width: 60%"></span><span class="lp-sk" style="width: 50%"></span>
          </div>
        <!-- /for -->
      </div>
    </section>
  <!-- /if -->
<!-- /if -->
```

## B.9 Reports

Section 9. `rApps` = your applications; `{rAppT}` = chosen app or the app's placeholder ("Loading applications…" while loading). `rFields` / `{rFN}` = field toggles and selected count [only if they exist]. `rFmtCards` = format cards [only if they exist] (`f.fs` badge text CSV / XLSX / JSON, `f.t` name, `f.d` description). `rPrevGo` = Generate Preview, `rDlGo` / `{rDlT}` = Download Report. `rEmpty` / `rBusy` / `rGen` = before / generating / generated. `{rFt}` / `{rFile}` = format badge and file name [IF DATA EXISTS]; `{rMetaN}` / `{rMetaF}` = "N rows" / "N fields". `rQ` = your results search.

```html
<!-- if isReports -->
  <div class="lp-rb lp-rz" style="--d: 1">
    <aside class="lp-rp" aria-label="Report settings">
      <div class="lp-rs2">
        <span class="{lp-rs2-n [lp-ok]}">1</span>
        <div class="lp-rs2-b">
          <p class="lp-rs2-t">Application</p>
          <div class="lp-anc lp-blk">
            <button type="button" class="{lp-sel2 [lp-on]}" onClick={rAppGo} aria-haspopup="listbox" aria-expanded={rAppOpen}>
              <span><small>Accounts of</small><b class="{[lp-ph]}">{rAppT}</b></span><Icon name="chevron-down" size="16" bold class="lp-chev"/>
            </button>
            <!-- if rAppOpen -->
              <div class="lp-pop lp-l" role="listbox" aria-label="Accounts of" style="width: 100%">
                <!-- for each o in rApps -->
                  <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                    <span class="lp-grow"><b>{o.t}</b>
                    <!-- if o.m -->
                      <small>{o.m}</small>
                    <!-- /if -->
                    </span>
                    <!-- if o.on -->
                      <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                    <!-- /if -->
                  </button>
                <!-- /for -->
              </div>
            <!-- /if -->
          </div>
        </div>
      </div>
      <div class="lp-rs2">
        <span class="lp-rs2-n lp-ok">2</span>
        <div class="lp-rs2-b">
          <p class="lp-rs2-t">
            Fields<span class="lp-mono">{rFN}</span>
            <!-- if rAllOff -->
              <button type="button" class="lp-lk" onClick={rAllF}>Select all</button>
            <!-- /if -->
          </p>
          <div class="lp-tgl">
            <!-- for each f in rFields -->
              <button type="button" class="{lp-tg [lp-on]}" aria-pressed={f.on} onClick={f.go}>
                <span class="lp-ckb"><Icon name="check" size="11" bold/></span>{f.t}
              </button>
            <!-- /for -->
          </div>
        </div>
      </div>
      <div class="lp-rs2 lp-end">
        <span class="lp-rs2-n lp-ok">3</span>
        <div class="lp-rs2-b">
          <p class="lp-rs2-t">Format</p>
          <div class="lp-fmts" role="radiogroup" aria-label="File format">
            <!-- for each f in rFmtCards -->
              <button type="button" class="{lp-fmt [lp-on]}" role="radio" aria-checked={f.on} onClick={f.go}>
                <span class="{lp-ft lp-csv|lp-xls|lp-json}">{f.fs}</span><b>{f.t}</b><small>{f.d}</small>
              </button>
            <!-- /for -->
          </div>
        </div>
      </div>
      <div class="lp-rp-f">
        <button type="button" class="lp-btn lp-btn-g" onClick={rPrevGo}>
          <Icon name="eye" size="16"/>Preview report
        </button>
        <button type="button" class="{lp-btn lp-btn-p [lp-dis]}" onClick={rDlGo}>
          <Icon name="download" size="16" bold class="lp-dl"/>{rDlT}
        </button>
      </div>
    </aside>
    <section class="lp-cd lp-rv" style="margin-top: 0" aria-label="Report preview">
      <!-- if rEmpty -->
        <div class="lp-hero0" style="padding: 70px 20px 80px">
          <div class="lp-ghost" aria-hidden="true">
            <!-- for each g in ghost -->
              <div class="lp-gh-r" style="--i: {g.i}">
                <span class="lp-gh-av"></span><span class="lp-gh-l" style="width: {g.w1}"></span><span class="lp-gh-l lp-s" style="width: {g.w2}"></span><span class="lp-gh-p"></span>
              </div>
            <!-- /for -->
          </div>
          <p class="lp-hero0-t" style="font-size: 24px">Your report preview appears here</p>
          <p class="lp-hero0-s">Pick an application and the fields you need, then preview it before you download.</p>
        </div>
      <!-- /if -->
      <!-- if rBusy -->
        <div class="lp-rv-h">
          <span class="lp-sk" style="width: 220px; height: 16px"></span>
        </div>
        <div aria-hidden="true">
          <!-- for each k in sk -->
            <div class="lp-skr">
              <span class="lp-sk" style="width: 16px"></span><span class="lp-sk lp-av"></span><span class="lp-sk" style="width: {k.w}"></span><span class="lp-sk"></span><span class="lp-sk" style="width: 70%"></span><span class="lp-sk" style="width: 60%"></span><span class="lp-sk" style="width: 50%"></span>
            </div>
          <!-- /for -->
        </div>
      <!-- /if -->
      <!-- if rGen -->
        <div class="lp-doc-h">
          <span class="lp-doc-tab"><span class="{lp-ft lp-csv|lp-xls|lp-json}">{rFt}</span><b>{rFile}</b></span><span class="lp-kv3"><b>{rMetaN}</b></span><span class="lp-kv3"><b>{rMetaF}</b></span><span class="lp-grow"></span>
          <label class="lp-sr" style="width: 240px">
            <Icon name="search" size="16"/>
            <input type="search" value={rQ} onChange={onRQ} placeholder="Search the report" aria-label="Search the report">
          </label>
        </div>
        <!-- YOUR EXISTING results table, with the adapter classes (section 6.4) -->
        <div class="lp-xt-wrap">
          <table class="lp-xt" aria-label="Report preview">
            <thead>
              <tr>
                <th class="lp-xt-th lp-xt-num">#</th>
                <!-- for each h in your existing columns -->
                  <th class="lp-xt-th">{h.title}</th>
                <!-- /for -->
              </tr>
            </thead>
            <tbody class="{lp-xt-body [lp-lb]}">
              <!-- for each r in the rows you show today -->
                <tr class="lp-xt-r" style="--i: {index}">
                  <td class="lp-xt-td lp-xt-num">{row number}</td>
                  <!-- for each c in r's cells -->
                    <td class="lp-xt-td" title={value}><span class="{lp-tt [lp-mono]}">{value}</span></td>
                  <!-- /for -->
                </tr>
              <!-- /for -->
            </tbody>
          </table>
          <!-- if no rows -->
            <div class="lp-emp">
              <p class="lp-emp-t">{the app's no-results text}</p>
            </div>
          <!-- /if -->
        </div>
        <!-- if rMore -->
          <div class="lp-more">
            <span>{the app's "Showing 10 of 20"}</span><button type="button" class="lp-btn lp-btn-g lp-btn-sm" onClick={your Show 10 more handler}>Show 10 more</button><button type="button" class="lp-btn lp-btn-g lp-btn-sm" onClick={your Show all handler}>Show all</button>
          </div>
        <!-- /if -->
      <!-- /if -->
    </section>
  </div>
<!-- /if -->
```

## B.10 FMID Reports

Section 10. `pItams` / `pApps` = your ITAM ID and Application options. Add the required asterisk `<span class="lp-req-star">*</span>` after each required label. `pModes` = your Bank ID modes, `{pMX}` = indicator transform, `pSpecific` = the IDs input is shown. `pFetch` / `{pFetchT}` = Fetch Report; `pReset` = Reset (add Clear all next to it, same style, if the app has both); `{pHint}` = the app's hint. `{pAppName}`, `{pXFmt}` (render your existing format select here, class `lp-mini`), `mxGo` = Download Security Matrix. `pIdle` / `pFetching` / `pDone` = states. `pSteps` / `{pP}` only if real progress exists (10.4). Results: `{pUsers}` / `{pUsersL}` = number + label; the `.lp-kv3` pills stand for your result chips; `pViews` = Detail / Matrix, `{pVX}` = indicator transform; `pCsv` / `pXls` / `pJson` = your exports. **The four `.lp-dd` buttons are placeholders for YOUR filters (names, order, menus) — add each one's info tooltip trigger `<button type="button" class="lp-inf" aria-label="About {filter}"><Icon name="info" size="14"/></button>` after its `<small>` label.** `{pCount}` = the app's counts line. `pGroups` = your COLUMNS groups. The table block is your Detail table — and, the same way, your Matrix table with ALL its columns.

```html
<!-- if isReport -->
  <div class="lp-pgrid lp-rz" style="--d: 1">
    <section class="lp-prm" aria-label="Search parameters">
      <p class="lp-prm-h">Search parameters</p>
      <div class="lp-prm-r">
        <div class="lp-fld">
          <small><span class="lp-sel2-n lp-ok">1</span>ITAM ID</small>
          <div class="lp-anc">
            <button type="button" class="{lp-sel2 [lp-on]}" onClick={pItamGo} aria-haspopup="listbox" aria-expanded={pItamOpen}>
              <span><small class="lp-sr-only">ITAM</small><b class="{pItamPh}">{pItamT}</b></span><Icon name="chevron-down" size="16" bold class="lp-chev"/>
            </button>
            <!-- if pItamOpen -->
              <div class="lp-pop lp-l" role="listbox" aria-label="ITAM" style="width: 280px">
                <!-- for each o in pItams -->
                  <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                    <span class="lp-grow"><b>{o.t}</b>
                    <!-- if o.m -->
                      <small>{o.m}</small>
                    <!-- /if -->
                    </span>
                    <!-- if o.on -->
                      <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                    <!-- /if -->
                  </button>
                <!-- /for -->
              </div>
            <!-- /if -->
          </div>
        </div>
        <span class="lp-fz-ln lp-prm-ln" aria-hidden="true"><Icon name="chevron-right" size="16"/></span>
        <div class="lp-fld">
          <small><span class="lp-sel2-n lp-ok">2</span>Application</small>
          <div class="lp-anc">
            <button type="button" class="{lp-sel2 [lp-on]}" onClick={pAppGo} aria-haspopup="listbox" aria-expanded={pAppOpen}>
              <span><small class="lp-sr-only">Application</small><b class="{pAppPh}">{pAppT}</b></span><Icon name="chevron-down" size="16" bold class="lp-chev"/>
            </button>
            <!-- if pAppOpen -->
              <div class="lp-pop lp-l" role="listbox" aria-label="Application" style="width: 280px">
                <!-- for each o in pApps -->
                  <button type="button" class="{lp-opt [lp-on]}" role="option" aria-selected={o.on} onClick={o.go} style="--i: {o.i}">
                    <span class="lp-grow"><b>{o.t}</b>
                    <!-- if o.m -->
                      <small>{o.m}</small>
                    <!-- /if -->
                    </span>
                    <!-- if o.on -->
                      <span class="lp-ok"><Icon name="check" size="16" bold/></span>
                    <!-- /if -->
                  </button>
                <!-- /for -->
              </div>
            <!-- /if -->
          </div>
        </div>
        <span class="lp-fz-ln lp-prm-ln" aria-hidden="true"><Icon name="chevron-right" size="16"/></span>
        <div class="lp-fld">
          <small><span class="lp-sel2-n lp-ok">3</span>Bank IDs</small>
          <div class="lp-bseg" role="group" aria-label="Bank IDs">
            <span class="lp-ind" style="transform: {pMX}"></span>
            <!-- for each m in pModes -->
              <button type="button" class="{[lp-on]}" onClick={m.go}>{m.t}</button>
            <!-- /for -->
          </div>
        </div>
      </div>
      <!-- if pSpecific -->
        <div class="lp-ids">
          <textarea value={pIds} onChange={onPIds} placeholder={the app's placeholder} aria-label="Bank IDs">
          </textarea>
          <small>Paste Bank IDs separated by commas, spaces or new lines.</small>
        </div>
      <!-- /if -->
      <div class="lp-prm-f">
        <button type="button" class="{lp-btn lp-btn-p [lp-dis]}" onClick={pFetch}>
          <Icon name="arrow-right" size="16" bold/>{pFetchT}
        </button>
        <button type="button" class="lp-btn lp-btn-g" onClick={pReset}>Reset</button>
        <span class="lp-hintx">{pHint}</span>
      </div>
    </section>
    <aside class="lp-mxk" aria-label="Security matrix">
      <span class="lp-ft lp-xls">XLSX</span>
      <p class="lp-mxk-t">Security matrix</p>
      <p class="lp-mxk-s">The functional entitlement security matrix for {pAppName}.</p>
      <div class="lp-mxk-f">
        <span class="lp-mini">{pXFmt}</span>
        <button type="button" class="lp-btn lp-btn-g lp-btn-sm" onClick={mxGo}>
          <Icon name="download" size="15" class="lp-dl"/>Download
        </button>
      </div>
    </aside>
  </div>
  <!-- if pIdle -->
    <div class="lp-hero0 lp-rz" style="--d: 2; padding: 56px 20px 70px">
      <div class="lp-ghost" aria-hidden="true">
        <!-- for each g in ghost -->
          <div class="lp-gh-r" style="--i: {g.i}">
            <span class="lp-gh-av"></span><span class="lp-gh-l" style="width: {g.w1}"></span><span class="lp-gh-l lp-s" style="width: {g.w2}"></span><span class="lp-gh-p"></span>
          </div>
        <!-- /for -->
      </div>
      <p class="lp-hero0-t" style="font-size: 24px">Nothing fetched yet</p>
      <p class="lp-hero0-s">Choose the ITAM, the application and whose Bank IDs to include, then fetch. Large applications can take a moment.</p>
    </div>
  <!-- /if -->
  <!-- if pFetching -->
    <div class="lp-fet" role="status" aria-live="polite">
      <p class="lp-fet-t">Fetching entitlement data…</p>
      <div class="lp-fsteps">
        <!-- for each s in pSteps -->
          <span class="{lp-fstep [lp-now|lp-done]}"><span class="lp-dot"></span>{s.t}</span>
        <!-- /for -->
      </div>
      <div class="lp-ibar">
        <i style="--p: {pP}"></i>
      </div>
      <p class="lp-fet-s">This may take a moment for large applications.</p>
    </div>
  <!-- /if -->
  <!-- if pDone -->
    <section class="lp-rs lp-rz" style="--d: 0" aria-label="Results">
      <div class="lp-rhd">
        <div class="lp-rhd-n">
          <b class="lp-rhd-b">{pUsers}</b><span class="lp-rhd-l">{pUsersL}</span>
        </div>
        <span class="lp-kv3"><small>ITAM</small><b>{pItamV}</b></span><span class="lp-kv3"><small>Application</small><b>{pAppV}</b></span><span class="lp-grow"></span>
        <div class="lp-vseg" role="tablist" aria-label="View">
          <span class="lp-ind" style="transform: {pVX}"></span>
          <!-- for each v in pViews -->
            <button type="button" role="tab" class="{[lp-on]}" onClick={v.go}>{v.t}</button>
          <!-- /for -->
        </div>
        <div class="lp-xg" role="group" aria-label="Export">
          <span class="lp-xg-l"><Icon name="download" size="14" class="lp-dl"/>Export</span>
          <button type="button" onClick={pCsv}>CSV</button>
          <button type="button" onClick={pXls}>Excel</button>
          <button type="button" onClick={pJson}>JSON</button>
        </div>
      </div>
      <div class="lp-cd">
        <div class="lp-cd-h">
          <label class="lp-sr">
            <Icon name="search" size="16"/>
            <input type="search" value={pQ} onChange={onPQ} placeholder="Search users, FMIDs, entitlements" aria-label="Search users, FMIDs, entitlements">
          </label>
          <button type="button" class="lp-dd">
            <small>Domicile</small>All countries<Icon name="chevron-down" size="14" bold/>
          </button>
          <button type="button" class="lp-dd">
            <small>Entity</small>All entity countries<Icon name="chevron-down" size="14" bold/>
          </button>
          <button type="button" class="lp-dd">
            <small>Bank ID</small>All<Icon name="chevron-down" size="14" bold/>
          </button>
          <button type="button" class="lp-dd">
            <small>FMID</small>All<Icon name="chevron-down" size="14" bold/>
          </button>
          <span class="lp-grow"></span><span class="lp-mono" style="font-size: 11.5px; color: var(--muted)">{pCount}</span>
        </div>
        <!-- if pDetail -->
          <div class="lp-cgs">
            <span class="lp-cgs-l">Columns</span>
            <!-- for each g in pGroups -->
              <button type="button" class="{lp-tg [lp-on]}" aria-pressed={g.on} onClick={g.go}>
                <span class="lp-ckb"><Icon name="check" size="11" bold/></span>{g.t}
              </button>
            <!-- /for -->
          </div>
        <!-- /if -->
        <!-- YOUR EXISTING Detail table (and, in the same way, your existing Matrix table with ALL its columns), with the dense adapter (sections 10.5, 10.6) -->
        <div class="lp-xt-wrap lp-xt-tall">
          <table class="{lp-xt lp-xt-dense lp-xt-sticky [lp-xt-grouped]}" aria-label={the app's label}>
            <thead>
              <!-- if the table has a group header row -->
                <tr>
                  <th class="lp-xt-gh lp-xt-stk" style="--c: transparent"></th>
                  <!-- for each g in your column groups -->
                    <th class="lp-xt-gh" colspan={g.columnCount} style="--c: {group colour}"><i></i>{g.title}</th>
                  <!-- /for -->
                </tr>
              <!-- /if -->
              <tr>
                <th class="lp-xt-th lp-xt-stk">{first column title, as today}</th>
                <!-- for each h in your columns (ALL of them) -->
                  <th class="{lp-xt-th [lp-xt-gs]}">{h.title}<!-- keep your resize / sort handles if the table has them --></th>
                <!-- /for -->
              </tr>
            </thead>
            <tbody class="{lp-xt-body [lp-lb]}">
              <!-- for each r in your rows -->
                <tr class="lp-xt-r" style="--i: {index}">
                  <td class="lp-xt-td lp-xt-stk">
                    <span class="lp-who"><span class="lp-mgx">{initials}</span><span><b>{user name}</b><small>{Bank ID}</small></span></span>
                  </td>
                  <!-- for each c in r's cells (ALL columns) -->
                    <td class="{lp-xt-td [lp-xt-gs]}">
                      <!-- TEXT --><span class="{lp-tt [lp-mono] [lp-dim]}">{value}</span>
                      <!-- LIST OF ENTITLEMENTS --><span class="lp-ent"><span class="lp-chp">{entitlement}</span><span class="lp-chp">{entitlement}</span></span>
                      <!-- YES / NO or TICK --><span class="{lp-yn [lp-y]}">{the app's value}</span>
                    </td>
                  <!-- /for -->
                </tr>
              <!-- /for -->
            </tbody>
          </table>
          <!-- if no rows match -->
            <div class="lp-emp">
              <p class="lp-emp-t">{the app's no-results text}</p>
            </div>
          <!-- /if -->
        </div>
      </div>
    </section>
  <!-- /if -->
<!-- /if -->
```

## B.11 Toast

Section 11. `{toast}` = your message. `lp-show` while visible.

```html
<div class="{lp-toast [lp-show]}" role="status" aria-live="polite">
  <span class="lp-tck"><Icon name="check" size="14" bold/></span>{toast}<i class="lp-tbar"></i>
</div>
```

---

# Appendix C — Icons

24×24 viewBox, stroke only (section 1). Each icon is one `<path d="…">`.

| Name | Path `d` |
|---|---|
| `arrow-right` | `M5 12h14M13 6l6 6-6 6` |
| `arrow-up` | `M12 19V5M6 11l6-6 6 6` |
| `box` | `M12 3 4 7v10l8 4 8-4V7zM4 7l8 4 8-4M12 11v10` |
| `check` | `M5 12.5l4.5 4.5L19 7.5` |
| `chevron-down` | `M6 9l6 6 6-6` |
| `chevron-left` | `M15 6l-6 6 6 6` |
| `chevron-right` | `M9 6l6 6-6 6` |
| `close` | `M6 6l12 12M18 6 6 18` |
| `columns` | `M4 5h16v14H4zM9.5 5v14M14.5 5v14` |
| `database` | `M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3zM4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3` |
| `doc` | `M7 3h7l5 5v13H7zM14 3v5h5M10 13h6M10 17h6` |
| `download` | `M12 4v11M7 10l5 5 5-5M5 20h14` |
| `eye` | `M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12zM12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6z` |
| `grid` | `M4 4h6v6H4zM14 4h6v6h-6zM4 14h6v6H4zM14 14h6v6h-6z` |
| `info` | `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM12 11v6M12 7.5v.01` |
| `layers` | `M12 3 3 7.5l9 4.5 9-4.5zM3 12l9 4.5 9-4.5M3 16.5 12 21l9-4.5` |
| `pencil` | `M4 20h4L19 9l-4-4L4 16zM13.5 6.5l4 4` |
| `pin` | `M9 4h6l-1 6 4 3v2H6v-2l4-3zM12 15v6` |
| `plus` | `M12 5v14M5 12h14` |
| `refresh` | `M20 11a8 8 0 1 0-2.3 5.7M20 4v7h-7` |
| `search` | `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4` |
| `server` | `M4 4h16v6H4zM4 14h16v6H4zM8 7h.01M8 17h.01` |
| `star` | `M12 3.5l2.6 5.3 5.9.9-4.3 4.1 1 5.8-5.2-2.7-5.2 2.7 1-5.8L3.5 9.7l5.9-.9z` |
| `trash` | `M4 7h16M10 11v6M14 11v6M6 7l1 13h10l1-13M9 7V4h6v3` |
| `upload` | `M12 16V5M7 10l5-5 5 5M5 20h14` |
| `user` | `M12 12a4 4 0 1 0 0-8 4 4 0 0 0 0 8zM4 21a8 8 0 0 1 16 0` |
| `zap` | `M13 3 5 14h6l-1 7 8-11h-6z` |
