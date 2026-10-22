# FMCES: restyle the 7 list & report pages — v2 (styling only, self-verified on localhost)

You are working in the FMCES codebase ("Markets Operations One", dark navy + teal). The app runs locally at **http://localhost:8001** and you can open it in the browser. Your job is to make seven pages look **exactly** like the reference design in the folder `fmces-ui-kit/`, **without changing any behaviour**, and to **prove it** with the included self-check before you finish.

**Do not stop until every page passes.** Do not ask me to look at it for you: open the pages yourself, run the self-check, compare with the reference, fix, and repeat. Only stop early for the three reasons in §3.4.

---

## 0. Why there is a v2 (read this, so you don't repeat it)

A first attempt from my earlier prompt came out very different from the design:

| What went wrong | What it must be |
|---|---|
| Several pages kept the app's **ocean-blue** colours: Application Management, Entitlement Users, Reports, FMID Reports. That covered the Add New, Create Account, Search, Download Report and Fetch Report buttons, the CSV/EXCEL/JSON toggle, the tab underline, the Refresh link and the blue column chips. | **No blue anywhere** on these pages. The primary button is solid teal `#2DD4BF` with dark text `#062A26`. Secondary buttons are dark with a `#2B4556` border. Selected items get a teal tint. |
| There were **no gradients and no depth**. Cards were flat grey boxes, and the page had no background glow. | Cards use the gradient `linear-gradient(180deg, rgba(21,39,51,.72), rgba(17,30,39,.5))`, an 18px radius, a `#23394A` border and a soft shadow. The page has the slow teal/blue glow `.lp-aur` behind the header. |
| Application Management had plain **underline tabs**. | **Glass segmented tabs** (`.lp-gt`) with a sliding indicator. |
| The **facts** (All / Draft / Approved …) sat under the page title, outside the card, as a plain row. | Facts sit **inside the list card**, top-left, with the coloured **mix bar** under them. The toolbar (search, filter, refresh, export) is on the same row, on the right. |
| Status and Active were **coloured pills**. | A **7px dot + the word** in colour, with no pill background (`.lp-lc`, `.lp-st`). |
| The **edit pencil was always visible**. | The pencil is **hidden until the row is hovered** (or focused). It stays in its own last column. |
| Table header and rows used the app's own table styling. | The header is 42px, 11px uppercase, muted `#86A0AE`, with a transparent background. Rows are 54px with 13px `#A7B8C4` text, a hairline divider and hover `#162A37`. |

Most of this happened because **the app's UI library theme won over the new stylesheet**. Some library components (Button, Tabs, Chip, ToggleButton, Select…) were kept, and only some classes were added to them. §2.4 fixes that.

**This time you have:**
- `fmces-pages.css`: the exact stylesheet. Every rule is scoped under `.lp-root`, with higher specificity, so a library theme can't override it.
- `reference/*.html`: **the target pages themselves**, rendered with that stylesheet and the exact class names, one per page and state. Open them in the browser next to your page. `reference/index.html` lists them all.
- `fmces-verify.js`: a **self-check** you run on each page. It prints PASS / FAIL with the expected and actual values (colours, sizes, structure, hover pencil, no blue…).
- `markup.md`: the templates for every piece, with conditionals and loops, plus the icon paths.

---

## 1. The kit

Put the folder `fmces-ui-kit/` in the project root. If it isn't there, that is the one thing to ask me about before starting. It contains:

| File | What it is | What you do with it |
|---|---|---|
| `PROMPT.md` | This document. | Follow it. |
| `fmces-pages.css` | The stylesheet, about 70 KB. | Copy it **byte for byte** into the app's styles. Load it globally, **after** all other CSS, including the UI library's. Never edit, reformat, translate (Tailwind, SCSS, CSS-in-JS) or split it. |
| `markup.md` | The HTML templates for every block, plus the icons. | Translate to your framework's templates and bind your existing state and handlers. |
| `reference/` | 17 static reference pages (`.html`), PNG snapshots, `index.html`, and a row-hover close-up. | Your visual target. Open them in the browser. They need `../fmces-pages.css`, so open them from inside the kit folder: serve the folder (for example `npx serve fmces-ui-kit`) or open the file directly in the browser. |
| `fmces-verify.js` | The self-check. | Run it on every page and state (§10). |

About the reference pages:
- They show **the look only**. Their data, column sets, top bar and dock are stand-ins. **Never copy their data, columns or texts into the app.** The app's own data, columns and labels always win.
- The PNGs were rendered without web fonts. Trust the HTML pages for typography.

Also create these two files and keep them updated as you work:
- `fmces-ui-kit/out/INVENTORY.md`: for every page, every control, column (in order, with its header text), button, menu item, filter, tooltip, label, message and state that exists **today**. Write it **before** changing anything. It is your checklist at the end. Include **every column of the FMID Reports Matrix view**.
- `fmces-ui-kit/out/DECISIONS.md`: every place where you used a default from §3.5 or made a judgement call, one line each.

---

## 2. Rules

### 2.1 Styling only. Keep everything.

You may:
- add classes and wrappers;
- swap an element's **visual** form while calling the same handler and reading the same state (for example, a library `<Button>` becomes a native `<button class="lp-btn lp-btn-p">` with the same `onClick`);
- add purely visual pieces from the design;
- add view-only state for animations.

You may **not**:
- change routes, API calls, payloads, auth or role checks;
- change the logic of existing handlers;
- remove or rename any column, field, button, menu item, tab, filter, tooltip, label or message;
- change default sort, page size, visible columns or column order;
- change ids or `data-testid`s.

Everything below must exist and work exactly as before, on every page where it exists today:
- **Tables:**
  - column **resize** (drag the header edge);
  - column **drag / reorder**;
  - column **show / hide**;
  - the **pin** column and pinned rows;
  - the **`#`** column;
  - **sorting**;
  - **row click opens the existing drawer** (the drawers are already restyled; don't touch them);
  - the **edit pencil** (now hover-revealed, same handler);
  - **OLA**, **Active**, **Status**, **CII Number**, **CR Number** and **Onboard Date** columns;
  - **pagination** with **Rows 15 / 25 / 50**;
  - the footer text.
- **Toolbars:**
  - **search**;
  - the **filter (funnel)** button and its panel;
  - **Refresh**;
  - **Re-evaluate** (Entitlement Management);
  - **Export** (the "N rows · N cols" menu with CSV / Excel).
- **Header:**
  - **Presets**, with "Page defaults", save, star default, delete and their toasts;
  - **Upload** (Excel modal), wherever it exists today;
  - the create buttons: **Add New**, **New Profile**, **New Policy**, **Create Account**;
  - "**Updated just now**".
- **Application Management:**
  - the tabs **ITAM, Applications, Features, Actions**;
  - **Add New** dropdown. Its ITAM item and the ITAM tab stay **only for the roles that see them today**: reuse the existing check. **Never show "restricted" / "role" text**; users without the role simply don't see those items.
  - **Select Application** on Features / Actions.
- **Entitlement Users:**
  - **Find User Accounts**;
  - **ITAM NAME → APPLICATION** ("select ITAM first");
  - **QUICK SEARCH** cross-ITAM lookup, with its **Search** button;
  - **Accounts / Entitlements** tabs with counts;
  - the empty message;
  - **Create Account**.
- **Reports:**
  - Application select, including "Loading applications…";
  - **Fields** toggles (Account Name, Account Owner, Account Type, Account Status, Is Privileged, Application);
  - **Format** CSV / EXCEL / JSON;
  - **Generate Preview**;
  - **Download Report**;
  - the "No Report Generated Yet" empty state;
  - preview table and its search;
  - "Showing X of Y";
  - **Show 10 more** / **Show all**.
- **FMID Reports** (page title "**Report**"):
  - **Search Parameters**;
  - **ITAM ID \***, **Application \*** ("Select ITAM ID first"), **Bank ID(s) \*** with **All Users / Specific IDs**;
  - the hint "All users for the selected Application will be fetched in one request.";
  - **Fetch Report (All Users)**, **Reset** (and **Clear all** if it exists);
  - **Functional Entitlement Security Matrix**, with the **Excel (.xlsx)** select and **Download Security Matrix**;
  - the fetching state;
  - result chips;
  - **Export CSV / Excel / JSON**;
  - **Detail / Matrix**;
  - filters **with their info tooltips**;
  - search;
  - the counts line;
  - **COLUMNS** groups;
  - the grouped Detail table;
  - the **Matrix view with ALL of its columns**: same order, same headers, nothing dropped, merged or renamed.

### 2.2 Names stay

- **Dock (sidebar):** no name, icon, order or link changes. The only change is the gliding highlight (§9).
- **Page titles and subtitles:** keep the app's text. Today they are:
  - "Application Management";
  - "Data Profile Management";
  - "Data Policy Management";
  - "Entitlement Management";
  - "Entitlement Users";
  - "Reports";
  - "Report", which is **FMID Reports** in the dock. **Never rename anything to "Entitlement Report".**
- Column headers, statuses, filter names, tooltips, empty messages and toasts keep the app's text. Only CSS changes their letter case: headers are uppercased by `text-transform`.

### 2.3 Don't touch

The top app bar, the workspace tabs, the dock items, the record drawers, the New/Edit forms and the Home dashboard.

### 2.4 No library look on these pages (this caused the blue)

On these seven pages, render these elements as **native HTML elements with the `lp-` classes from `markup.md`**, calling the same handlers and reading the same state:
- buttons;
- tabs;
- segmented toggles (Detail/Matrix, All Users/Specific IDs, rows per page);
- chips and toggles;
- menus and popovers (Presets, Add New, Export, Select Application);
- the select triggers;
- the search inputs;
- the page header;
- cards;
- the toast.

Keep a library component **only** where it carries logic you would otherwise have to rewrite: the **table/grid** (resize, drag, sort, virtualisation), date pickers, an autocomplete, a modal's focus trap. For those:
- put the adapter classes on its parts (§6.2, §6.4);
- neutralise its own colours, borders, backgrounds, heights and fonts **only inside `.lp-root`**, in a separate file `fmces-pages-overrides.css` (selectors like `.lp-root .lp-xt .MuiTableCell-root { … }`), loaded after `fmces-pages.css`;
- **never change the global theme**, because other pages use it.

Anything portaled to `<body>` (menus, modals, tooltips, toasts) is outside `.lp-root` and would lose the styles. Wrap the portaled content in its own `<div class="lp-root">`, or portal it into the page's `.lp-root`.

### 2.5 Fonts

The design needs **Manrope 200–800** and **JetBrains Mono 400–600**. Make sure this is loaded once, globally. If an existing link lacks weights, replace it:

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@200;300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">
```

If the corporate network blocks Google Fonts, self-host the same families and weights, and note it in DECISIONS.md.

---

## 3. How you work: a loop, page by page, until everything passes

### 3.1 Setup (once)

1. Find the kit. Read this file, `markup.md` and `reference/index.html`.
2. Map the code: for each page, the component, the route on localhost:8001, the table component, and where resize, drag, show/hide, sort, pin, pagination, presets and the row-click drawer live.
3. Write `out/INVENTORY.md` (§1) **before changing anything**. Open each page on localhost and click through it to make sure nothing is missed. Take a "before" screenshot of each page into `out/before/`.
4. Copy `fmces-pages.css` into the app and load it **last**. Add the fonts. Load `fmces-verify.js` in development only (§10.1).
5. Open any page, add the wrapper (§5), and run `await lpVerify()`. The **Stylesheet** checks must pass before you go on.

### 3.2 For each page, in this order

Application Management → Data Profile Management → Data Policy Management → Entitlement Management → Entitlement Users → Reports → FMID Reports.

1. **Build** the page from §6 and `markup.md`.
2. **Self-check:** open every state of the page listed in §10.2 on localhost:8001, at **1440 × 900**, with the mouse **outside** the table, and run `await lpVerify()`. Fix every **FAIL**. Re-run until it reports **0 FAIL**.
3. **Compare** with the reference: open the matching `reference/*.html` at 1440 × 900 and screenshot both. Write down every visible difference (colour, spacing, size, position, font, radius, border, shadow, icon, missing element), fix it, and screenshot again. Ignore differences in data, columns and labels; those come from the app. Repeat until you can't find a difference in look.
4. **Hover check:** hover a row. The pin and the edit pencil must fade in at the end of the row, and the row tint must change. Screenshot it, and compare with `reference/02-data-profiles-row-hover.png`.
5. **Behaviour check:** run the page's part of §10.4 (resize a column, drag a column, sort, pin, toggle a column chip, change rows per page, row click opens the drawer, pencil opens edit, export, presets…). Everything must still work.
6. Save screenshots to `out/after/<page>-<state>.png`. Paste the self-check summary line into `out/REPORT.md`. Then move to the next page.

### 3.3 Final pass

When all seven pages are done:
- run §10 again on **every** state;
- re-check `out/INVENTORY.md` line by line;
- run the app's tests and lint, if it has them;
- finish `out/REPORT.md` (§11).

Only then are you done.

### 3.4 The only reasons to stop and ask me

1. The kit folder is missing.
2. Something in this prompt would **remove or change a feature, a column or data**, and there is no way to keep it. Example: the table library cannot keep resize with the new header.
3. You need a backend or API change.

For anything else, use the defaults in §3.5, write the choice in `out/DECISIONS.md` and keep going.

### 3.5 Defaults (instead of questions)

| Topic | Default |
|---|---|
| FMID Matrix view columns | Keep **every** column exactly as today. List them in INVENTORY.md. The self-check prints the columns it sees; they must match. |
| Edit pencil | Stays in its own last column (`lp-xt-act`), **hidden until row hover / focus** (CSS does it). Same handler. Always visible on touch screens (CSS does it). |
| "Updated just now" | Show it in the header subtitle line as the `.lp-sync` pill. Clicking it runs the existing refresh. Keep any other place it appears today. |
| Card title ("Applications 22 total") | Keep it, as the `.lp-cdt` row at the top of the list card. |
| Facts counts | Count the **whole list**. If the list is paged on the server and the API has no totals per status, hide the facts row and mix bar for that list, put `data-lp-nofacts` on its `.lp-root`, and note it. Never count only the visible page. |
| Facts order | All, then the statuses the list has, in the app's order: Draft, Approved, Pending, Revision Required, Rejected (plus others it has). |
| Module numbers in the eyebrow | Dock order: 01 Application Management, 02 Data Profile Management, 03 Data Policy Management, 04 Entitlement Management, 05 Entitlement Users, 06 Reports, 07 FMID Reports. |
| Column menu | Replaced by the on-screen chips (§6.2.3), driven by the same state and handlers. Locked columns can't be hidden. Reordering by dragging chips is added only if the old menu could reorder. The Reset link appears only if a reset existed. Keep header drag-reorder if it exists. |
| FMID fetching progress | Show the stage pills and progress bar only if the app has real progress data. Otherwise show just the `.lp-fet` title and sentence. Never fake progress. |
| Reports layout | Left panel with numbered steps: 1 Application, 2 Fields, 3 Format. The footer has Generate Preview + Download Report. The preview goes on the right. |
| Library components you keep | Restyle them through `fmces-pages-overrides.css` under `.lp-root`, and list them in DECISIONS.md. |

---

## 4. How to read the markup

`markup.md` is framework-neutral HTML:

| Notation | Meaning |
|---|---|
| `{value}` | A value from your state. |
| `<!-- if x -->…<!-- /if -->` | Render only when `x` is true. |
| `<!-- for each a in list -->…<!-- /for -->` | Repeat for each item. |
| `class="lp-a {lp-b [lp-c]\|lp-d}"` | Static classes, plus a computed part: `[x]` is an optional modifier, `a\|b` is one of the alternatives. |
| `<Icon name="…" size="…" bold class="…"/>` | An inline SVG: `<svg class="lp-i [lp-b] [extra]" width height viewBox="0 0 24 24" aria-hidden="true"><path d="…"/></svg>`. The paths are at the end of `markup.md`. |

- Bind **your existing handlers**; the names in the templates are placeholders.
- Wording in the templates is a fallback; the app's text wins.
- When you are unsure how a block should look, open the matching `reference/*.html` and **read its source**. It is the rendered result, with the final class names.

---

## 5. The page wrapper (every page)

```html
<div class="lp-root" data-lp-page="{page id}">
  <div class="lp-pg">
    <div class="lp-aur" aria-hidden="true"><i class="lp-a1"></i><i class="lp-a2"></i></div>
    <div class="lp-pga">
      … header (§6.1), then the page's blocks …
    </div>
  </div>
  … upload dialog, toast …
</div>
```

| Page | `data-lp-page` | Reference files |
|---|---|---|
| Application Management | `app-management` | `01-app-management*.html` (5 states) |
| Data Profile Management | `data-profiles` | `02-data-profiles*.html` |
| Data Policy Management | `data-policies` | `03-data-policies*.html` |
| Entitlement Management | `entitlement-management` | `04-entitlement-management*.html` |
| Entitlement Users | `entitlement-users` | `05-entitlement-users-*.html` |
| Reports | `reports` | `06-reports-*.html` |
| FMID Reports | `fmid-reports` | `07-fmid-reports-*.html` |

- The wrapper covers everything to the right of the dock and below the workspace tabs.
- `.lp-pg` provides the page padding (`30px 44px 72px 14px`). Remove the old container's padding so it isn't doubled.
- The page background behind it stays the app's `#0F1A22`. If it isn't that colour, set it on the page container.

---

## 6. Page by page

Sizes and colours are all in the CSS. This section says **what goes where** and **which existing thing each piece replaces**. The reference pages show the result.

### 6.1 Header (all pages), markup B.2

Left side:
- **Eyebrow `.lp-eb`:** the module number in mono teal, a short line, then "FMCES".
- **Watermark `.lp-bign`:** the same number, 150px / 200, at 3.5% opacity behind the title.
- **Title `.lp-pt`:** 36px / 300.
- **Subtitle line `.lp-ps`:** the app's subtitle, then the `.lp-sync` "Updated just now" pill (list pages only).

Right side, **`.lp-pa`**, in this order:
1. **Presets** (`lp-btn lp-btn-g`): the layers icon, "Presets", the current preset name in `.lp-sub` (e.g. "Page defaults"), the count in `.lp-bdg`, and a chevron.
   - It opens the `.lp-pop` Presets popover: the name input + **Save**, then one `.lp-vw` row per preset, with star (default) and delete (only where allowed today), and a footnote.
2. **Upload** (`lp-btn lp-btn-g`, upload icon), where it exists today.
3. The create button (`lp-btn lp-btn-p`, plus icon), with the app's label: **Add New**, **New Profile**, **New Policy**, **Create Account**.
   - On Application Management it is the **Add New dropdown**: a chevron, and the plus turns 45° while open. It opens `.lp-pop.lp-nwm` with one `.lp-nwi` row per existing item (icon tile, bold title, small description). The ITAM row sits under a separator and only for the allowed roles.

### 6.2 The list card (Application Management tabs, Data Profile, Data Policy, Entitlement Management, Entitlement Users tabs), markup B.4 + B.5 + B.6

One card, `section.lp-cd`, top to bottom:

**6.2.1 Card title row `.lp-cdt`:** the app's card title and the "N total" pill.

**6.2.2 Card header `.lp-cd-h`:** two parts on one row.

- **Left: facts + mix bar.**
  - One `button.lp-fb` per fact: the big number `.lp-fb-n` (30px / 300) and under it the label `.lp-fb-l` with a 7px coloured dot `.lp-fb-d` (`--c`). "All" has no dot. A zero fact gets `lp-zero`, the active one `lp-on`.
  - Clicking a fact filters the list. Drive the existing status filter if there is one, so the two stay in sync. Click again (or All) to clear.
  - Under the facts, the **mix bar** `.lp-dist`: one segment `<i style="--c; --w: count; --k: index">` per fact with count > 0 (All excluded).
  - Fact colours: Draft `#F6C177`, Approved `#2DD4BF`, Pending `#C9B6FF`, Revision Required `#FDBA74`, Rejected `#F4A3A3`, Not active `#46636F`, People `#5EEAD4`, Service accounts `#93C5FD`, Enabled `#2DD4BF` (Enabled is not in the mix bar).
- **Right: toolbar `.lp-tl`**, in this order:
  1. search `.lp-sr`: icon, input, a `.lp-hits` "N matches" pill while there's a query, `.lp-x` clear, and a `<kbd>/</kbd>` hint when empty;
  2. the active-fact chip `.lp-fchip`;
  3. **your funnel filter button** as `.lp-ibx` (icon only, 38×38). It gets `lp-on` while its panel is open or a filter is applied;
  4. separator `.lp-tsep`;
  5. **Refresh** as `.lp-ibx`, with the refresh icon (`lp-spin` while loading). The app's text button "Refresh" becomes this icon button, with `aria-label="Refresh"` and `title="Refresh"`;
  6. extra actions (for example **Re-evaluate**) as `lp-btn lp-btn-g`, with their icon and label;
  7. **Export** `lp-btn lp-btn-g`, with the download icon, "Export" and a chevron. It opens a `.lp-pop` menu: the heading "Export" + mono "N rows · N cols", then `.lp-xi` rows for CSV and Excel with `.lp-ft` badges. After exporting, it shows "Exported" with a drawn check for 1.8s.

**6.2.3 Column chips `.lp-colbar`:**
- The label: the columns icon, "COLUMNS", then `<b>` with the number of visible columns.
- Then **one chip `.lp-cch` per column, in current order, hidden ones included**:
  - visible: `lp-on`, teal tint, check;
  - hidden: dashed border, plus;
  - locked: `lp-on lp-req`.
- Clicking a chip calls your existing show/hide handler. This replaces the old column dropdown.

**6.2.4 Table:** your existing table component with the **adapter** (§6.4) and the cell contents (§6.5), inside `.lp-xt-wrap`.

**6.2.5 Footer `.lp-tf`:**
- the app's footer text ("Showing 1–15 of 48 · Updated just now");
- "Rows per page" with `.lp-seg` buttons **15 / 25 / 50**;
- `.lp-pgr` page buttons (`.lp-pgb2`: `lp-on` for the current page, `lp-dis` for disabled).

**States:**
- **Loading:** 6 `.lp-skr` shimmer rows in a full-width row.
- **Empty:** `.lp-emp`, with a floating icon tile, the app's message and an optional "Clear search" / "Show all" button.

### 6.3 Application Management, markup B.3 + B.4

- **Tabs:** `.lp-gtw > nav.lp-gt[role=tablist]` with `style="--n: {visible tab count}"`.
  - Inside it, the sliding `.lp-ind` with `style="transform: translateX({active index × 100}%)"`.
  - One `button[role=tab]` per visible tab: name + `<em>` count. The active tab gets `lp-on` and `aria-selected="true"`.
  - **Remove the old underline tabs.** Not just hidden: the same tab state now drives the glass tabs.
  - Without the ITAM role there are 3 tabs and `--n: 3`. Compare with `reference/01-app-management-no-itam-role.html`.
- **Features / Actions:** the card starts with the **application band** `.lp-scx`. It replaces the old "Select Application" control and keeps its logic.
  - The label "Application", then the big trigger `.lp-scf` (monogram tile + name + chevron).
  - It opens `.lp-pop.lp-l.lp-scp`: a search box `.lp-psr`, the group label, and `.lp-opt` rows (monogram, name, meta "ITAM 51515 · 7 features · 4 actions" if you have the data, status dot, and a check on the selected row).
  - On the right of the band, `.lp-kv3` pills: App UID, ITAM, Status (only data you have).
  - Compare with `01-app-management-features.html` and `01-app-management-select-application.html`.
- **Add New** dropdown as in §6.1.

### 6.4 Table adapter: put these classes on YOUR table's elements

Do **not** replace your table component, and do **not** use the mockup grid classes `lp-tb lp-tr lp-td lp-thc lp-tbd lp-tw lp-gx* lp-ra` (the self-check fails on them). Add:

| Your element | Classes |
|---|---|
| Scroll container | `lp-xt-wrap` (+ `lp-xt-tall` for the FMID tables) |
| Table / grid root | `lp-xt lp-xt-click` (`lp-xt-click` = rows open the drawer). FMID tables: `lp-xt lp-xt-dense lp-xt-sticky` (+ `lp-xt-grouped` with a group header row). Add `lp-resizing` while a column is being resized. |
| Header cell | `lp-xt-th`. Sorted: + `lp-on`; descending: + `lp-desc`. Put the sort arrow as `<Icon name="arrow-up" size="13" bold class="lp-sa"/>` after the text. Dragged header: `lp-drag`; drop target: `lp-drop-l` / `lp-drop-r`. |
| **Resize handle** | `lp-xt-rz` on your existing handle element, inside the header cell (+ `lp-on` while dragging). It is a 9px hit area at the right edge, drawn as a hairline that becomes a glowing 2px teal line on hover and drag. Keep your resize logic and min/max widths. |
| Body (tbody / rows container) | `lp-xt-body`. Toggle `lp-lb` each time the whole row set is replaced (tab, filter, preset, refresh, application, results, Detail ↔ Matrix), to replay the row entrance. |
| Body row | `lp-xt-r`, with `style="--i: {index on page}"` and `data-k="{record id}"`. Selected (drawer open): `lp-sel`; pinned: `lp-pinned`; just changed: `lp-flash` for 1.2s. |
| Body cell | `lp-xt-td` |
| Pin column | the cell contains `button.lp-pin` (+ `lp-on` when pinned) |
| `#` column | `lp-xt-th lp-xt-num` / `lp-xt-td lp-xt-num` |
| **Edit pencil column** (last) | `lp-xt-th lp-xt-act` (empty header with `<span class="lp-sr-only">Edit</span>`), and `lp-xt-td lp-xt-act` holding `button.lp-rab` with the pencil icon. **Hidden until row hover / focus: the CSS does it.** |
| Full-width label rows ("PINNED 2") | `<tr><td class="lp-xt-grp" colspan="{all}"><div class="lp-tgr">…</div></td></tr>`. Only if pinned rows are grouped today. |
| FMID group header cells | `lp-xt-gh`, with `style="--c: {group colour}"` and a leading `<i></i>`. First column cells: `lp-xt-stk` (sticky). First column of each group: `lp-xt-gs`. |

If the library forces its own row height or header styles, set its row height prop to **54** (header **42**; FMID dense rows 52, header 38), and neutralise its paddings, backgrounds and borders in `fmces-pages-overrides.css`. Don't fight it inside `fmces-pages.css`.

### 6.5 Cell contents, markup B.5 (by column type; keep each value and its formatting)

| Column | Content |
|---|---|
| Primary name (App Name, Profile Name, Policy, Entitlement Name, Account…) | `span.lp-nmb` > `span.lp-mgx` (two-letter monogram; draft `lp-dr`, pending `lp-pd`, revision `lp-rv`, rejected `lp-rj`, service account `lp-sv` with the server icon, person with the user icon) + `span.lp-nmt` (bold name, + `lp-mono` for code-like names). |
| IDs, codes, owners, CII Number, CR Number, DLs | `span.lp-tt.lp-mono` |
| Text, descriptions | `span.lp-tt` (ellipsis + `title` with the full value) |
| Empty value | `span.lp-tt.lp-dim` with the app's marker ("—") |
| **Status** | `span.lp-lc.lp-lc-live` (Approved/Active/Published), `lp-lc-draft` (Draft), `lp-lc-pend` (Pending), `lp-lc-rev` (Revision Required), `lp-lc-err` (Rejected), `lp-lc-off` (Inactive/other): `<i></i><span class="lp-lct">{app word}</span>`. No pill. |
| **Active** / Enabled | `span.lp-st` (teal) or `span.lp-st.lp-off` (grey): `<i></i>{app word}`. No pill. |
| Version | `span.lp-vr` (small mono pill, e.g. "V1"). Keep the app's value. |
| Counts (Total Features, Total Actions…) | `span.lp-cn > b` (+ `lp-z` when 0). **No progress bars.** |
| Dates (Last Modified, Onboard Date, Created Date…) | `span.lp-dt > b{date as the app formats it} + small{relative, e.g. "6 days ago"}` |
| OLA | `button.lp-ola` (blue document icon tile), existing handler |
| Application / role chips | `span.lp-chp` (several in `span.lp-chps`) |
| Type (Person / Service) | `span.lp-ty` with the user / server icon |
| Yes / No | `span.lp-yn` (+ `lp-y` for Yes) |

### 6.6 Data Profile, Data Policy, Entitlement Management

These use the standard header (§6.1) and list card (§6.2), with their own columns, facts, buttons and the **Upload** dialog where it exists.

**Upload dialog** (markup B.7, `03-data-policies-upload.html`):
- `.lp-dw` full-screen layer > `.lp-dscrim` blurred backdrop > `section.lp-dlg`.
- Header: icon tile, title and sub-line, close.
- `.lp-steps`: template download, instructions, then the dashed `.lp-drop` zone or the `.lp-fil` file card.
- Footer: Cancel + teal Upload.
- Keep the modal's logic, validation and accepted types.

### 6.7 Entitlement Users, markup B.8 (`05-entitlement-users-*.html`)

- **Finder `.lp-fnd2`** (one card, three columns):
  - **Left `.lp-fz`:** heading "Find User Accounts" + "Select an application to load its accounts and entitlements." Then two big selects `.lp-sel2` joined by a chevron:
    1. **ITAM Name**: a numbered badge (teal when chosen), a small label, the value or placeholder `lp-ph`, and a chevron.
    2. **Application**: the same, with `lp-dis` until an ITAM is chosen ("Select ITAM first").

    Their options open in `.lp-pop.lp-l` lists of `.lp-opt`.
  - **Middle:** the "or" divider `.lp-fz-or`.
  - **Right `.lp-fz`:** heading "Quick search" + "Cross-ITAM lookup by owner or account name." Then `label.lp-qs` (56px) with the input "Owner / account name…" and a teal **Search** `lp-btn lp-btn-p lp-btn-sm`. Enter also searches.
- **Before choosing:** `.lp-hero0` with floating ghost rows, the app's empty message as the 30px / 300 title, and a muted line.
- **Loaded:**
  - glass tabs `.lp-gt` (`--n: 2`): **Accounts N** / **Entitlements N**;
  - next to them, the context card `.lp-ctx` (monogram, application name, "ITAM 55508 · FMCES"; or "Results for "…" across all ITAMs");
  - then the list card (§6.2) for the active tab. On Accounts, Status Enabled/Disabled uses `.lp-st`.
- **Create Account** is the teal button in the header actions.

### 6.8 Reports, markup B.9 (`06-reports-*.html`)

Two columns `.lp-rb`: the sticky settings panel `.lp-rp` (320–390px) on the left, and the preview card `.lp-cd.lp-rv` on the right.

**Panel:** numbered steps `.lp-rs2`, joined by a vertical line. The last step gets `lp-end`.
1. **Application:** a `.lp-sel2` select. While loading it shows "Loading applications…" in placeholder style.
2. **Fields:** the step title shows the count in mono ("6 of 6"). The `.lp-tg` toggles sit in a two-column grid and draw their check. Add "Select all" `.lp-lk` only if it exists today.
3. **Format:** three `.lp-fmt` radio cards, CSV / Excel / JSON, each with a coloured `.lp-ft` badge, name and description.

Footer `.lp-rp-f`: **Generate Preview** (`lp-btn lp-btn-g`, eye icon) and **Download Report** (`lp-btn lp-btn-p`, download icon, `lp-dis` until possible).

**Preview area:**
- **Before:** `.lp-hero0` with ghost rows, "No Report Generated Yet" and the app's sentence.
- **Generating:** skeleton rows.
- **After:**
  - `.lp-doc-h`: format badge + file name tab (only if you have a name), `.lp-kv3` pills "N accounts" / "N fields", and the existing search as `.lp-sr`;
  - the preview table with the adapter (no pin, no pencil, `#` column mono);
  - `.lp-more` with "Showing X of Y" and **Show 10 more** / **Show all** (`lp-btn lp-btn-g lp-btn-sm`).

### 6.9 FMID Reports, markup B.10 (`07-fmid-reports-*.html`)

**Top grid `.lp-pgrid`:**
- **Parameters card `.lp-prm`:**
  - the heading "Search Parameters";
  - one row with three fields joined by chevrons, each with a numbered teal badge, an uppercase label and a red `<span class="lp-req-star">*</span>`:
    1. **ITAM ID**: `.lp-sel2`;
    2. **Application**: `.lp-sel2`;
    3. **Bank ID(s)**: `.lp-bseg` segmented **All Users / Specific IDs** with a sliding `.lp-ind`.
  - With Specific IDs, the existing input appears as `.lp-ids` (textarea + helper).
  - Footer `.lp-prm-f`: **Fetch Report (All Users)** (`lp-btn lp-btn-p`, arrow icon), **Reset** (and **Clear all**) as `lp-btn lp-btn-g`, then the hint as `.lp-hintx`.
- **Matrix card `.lp-mxk`** (300px, faint blue tint): the XLSX badge, the title "Functional Entitlement Security Matrix", a line naming the application, then the existing **Excel (.xlsx)** select as `select.lp-mini` and **Download Security Matrix** (`lp-btn lp-btn-g lp-btn-sm`).

**Before fetching:** `.lp-hero0` with ghost rows and "Nothing fetched yet" (or the app's text).

**Fetching:** the `.lp-fet` panel (§3.5 default).

**Results:**
- **`.lp-rhd` bar:**
  - the big teal user count `.lp-rhd-b` (34px / 300) + "users fetched";
  - the result chips as `.lp-kv3`;
  - the **Detail / Matrix** `.lp-vseg` with a sliding `.lp-ind`;
  - the joined **Export | CSV | Excel | JSON** group `.lp-xg`.
- **Card under it:**
  - toolbar: search `.lp-sr`, then **each existing filter** as `span.lp-ddg > button.lp-dd` (small label + value + chevron) **+ `button.lp-inf`** (the info icon) wired to the **existing tooltip**; on the right, the app's counts line in mono;
  - Detail only: the COLUMNS group toggles `.lp-cgs .lp-tg`;
  - the table with the dense adapter: sticky header, group header row `lp-xt-gh` with coloured top lines, sticky first column `.lp-who` (monogram + name + Bank ID), entitlements as `.lp-ent .lp-chp`.
- Group colours, in order: `#5EEAD4`, `#93C5FD`, `#C9B6FF`, `#F6C177`, `#2DD4BF`, `#F4A3A3`, and repeat.
- **Matrix view:** the same dense styling on your matrix table, **with every column it has today**.

### 6.10 Toast, markup B.11

Restyle the existing toasts: a light pill at the bottom centre, a teal check that pops and draws, the message, and a thin countdown bar of 2.6s. Add `lp-show` to show it. Keep the messages and triggers.

---

## 7. Colours (tokens on `.lp-root`, already in the CSS)

| Token | Value | Use |
|---|---|---|
| `--bg` | `#0F1A22` | Page |
| `--card` / `--card2` | `#152733` / `#1B3140` | Cards / raised fills |
| `--border` / `--hair` / `--btn` | `#23394A` / `#1E3240` / `#2B4556` | Card borders / dividers / button borders |
| `--text` / `--text2` / `--muted` / `--faint` | `#E6EEF2` / `#A7B8C4` / `#86A0AE` / `#6F8896` | Text levels |
| `--accent` / `--accent2` / `--on` | `#2DD4BF` / `#5EEAD4` / `#062A26` | Primary teal / light teal / text on teal |
| `--draft` / `--pend` / `--err` / `--blue` | `#F6C177` / `#C9B6FF` / `#F4A3A3` / `#93C5FD` | Amber / lavender / red / info |

The only blue allowed is the light info blue `#93C5FD` (OLA button, service accounts, matrix card tint) and the faint background glow. **No `#1E88E5`, `#2196F3` or `#1976D2`-type blue anywhere**; the self-check scans every element for it.

---

## 8. Motion

All of this is CSS in `fmces-pages.css`, except the JS hooks marked **JS**.

**Already in the CSS:**
- Blocks rise in one after another (`lp-rz` with `--d`). Rows rise with a 38ms stagger (`--i`).
- Glass tab and segmented indicators slide with a spring (`transform` on `.lp-ind`).
- Buttons lift on hover and press on click; the primary has a soft sheen; the plus turns; the download icon nudges.
- Popovers pop from their corner, with their rows staggering. Checks draw themselves. The active fact number bumps. The search icon nudges on focus.
- Pin drops in. Skeletons shimmer. Ghost rows and the empty icon float. The toast check pops.

**JS hooks:**
- Toggle `lp-lb` on the table body when the row set is replaced (§6.4).
- **Directional tab slide:** on a tab change, add `lp-slr1`/`lp-slr2` to `.lp-xt-wrap` (new tab to the right) or `lp-sll1`/`lp-sll2` (to the left), alternating 1/2 each time.
- **Column fade:** toggle `lp-cf1` ↔ `lp-cf2` on `.lp-xt` when a column is shown or hidden.
- **Count-up:** facts and the FMID user count go from 0 to their value over 750ms with `requestAnimationFrame`, ending on the exact value.
- **FLIP** on sort / pin / filter: record each row's top by `data-k` before the change, then animate rows from their old position (600ms, `cubic-bezier(.22,1,.36,1)`). New rows fade up 8px. Skip it if the table virtualises rows.
- All of it respects `prefers-reduced-motion`. The CSS handles the CSS part; skip count-up and FLIP in JS.

**Do not add:**
- a top loading line;
- a pulsing or spinning sync icon;
- moving upload stripes;
- progress bars in table cells;
- rings or lifecycle bars;
- a cursor glow;
- a hero panel replacing the header;
- any "restricted/role" text.

---

## 9. Dock: gliding highlight (the only dock change)

- Inside the dock's item container (`position: relative`), add `<span class="lp-dk-sl" aria-hidden="true"></span>` behind the items.
- Set `style="transform: translateY({active item offsetTop}px)"` and recompute it on route change and resize.
- Match the current active highlight's position and size with `--dk-x`, `--dk-w`, `--dk-h` and `--dk-r` (defaults 7px, 44px, 44px, 12px).
- Remove the old static highlight's background and left bar, so it isn't drawn twice. Items get `position: relative; z-index: 1`.
- Hide the slider when no dock item is active.

The CSS is at the end of `fmces-pages.css`.

---

## 10. Verification protocol

### 10.1 Loading the self-check

Use whichever works in your setup:
- **Browser tool:** read `fmces-ui-kit/fmces-verify.js`, evaluate its contents in the page, then evaluate `await lpVerify()` and read the returned object (`summary`, `problems`).
- **Dev-only include:** in development builds only, load `fmces-verify.js` (for example copy it to the dev `public/` folder and add a `<script>` that is excluded from production builds). Then open any page with `?lpverify` in the URL: it runs after 2.5s and shows a results panel in the bottom-right, which you can screenshot. Or call `await lpVerify()`. Make sure it never ships in production, and write in the report how to run it.

Run it with the page fully loaded, at 1440 × 900, and **the mouse pointer outside the table**.

The self-check also reads `window.__lpVerifyLast`. The panel shows only FAIL / WARN / INFO lines; PASS lines are counted.

### 10.2 States to check (each must reach 0 FAIL)

| # | Page / state | How to get there | Reference |
|---|---|---|---|
| 1 | Application Management, Applications | open the page | `01-app-management.html` |
| 2 | … Add New menu open | click Add New | `01-app-management-new-menu.html` |
| 3 | … as a user without the ITAM role | use such a user, or the existing role toggle / mock you have; if neither exists, verify by code reading and say so | `01-app-management-no-itam-role.html` |
| 4 | … ITAM tab | click ITAM | (same look as 1) |
| 5 | … Features, application chosen | click Features, choose an application | `01-app-management-features.html` |
| 6 | … Select Application open | click the application trigger | `01-app-management-select-application.html` |
| 7 | … Actions, application chosen | click Actions | (same look as 5) |
| 8 | Data Profile Management | open | `02-data-profiles.html` |
| 9 | … searching | type a few letters | `02-data-profiles-search.html` |
| 10 | Data Policy Management | open | `03-data-policies.html` |
| 11 | … Upload dialog | click Upload | `03-data-policies-upload.html` |
| 12 | Entitlement Management | open | `04-entitlement-management.html` |
| 13 | … Presets open | click Presets | `04-entitlement-management-presets.html` |
| 14 | Entitlement Users, before choosing | open | `05-entitlement-users-start.html` |
| 15 | … Accounts loaded | choose ITAM + Application | `05-entitlement-users-loaded.html` |
| 16 | … Entitlements tab | click Entitlements | (same look as 15) |
| 17 | … quick search results | search an owner | (same look as 15, context card "Results for …") |
| 18 | Reports, before preview | open | `06-reports-start.html` |
| 19 | … preview generated | choose application, Generate Preview | `06-reports-preview.html` |
| 20 | FMID Reports, before fetch | open | `07-fmid-reports-start.html` |
| 21 | … results, Detail | choose ITAM + Application, Fetch | `07-fmid-reports-results.html` |
| 22 | … results, **Matrix** | click Matrix | (same dense table look; the INFO line lists its columns: compare with INVENTORY.md) |
| 23 | Any list, row hovered | hover a row (don't run the self-check while hovering) | `02-data-profiles-row-hover.png` |
| 24 | Every page at 900px wide | resize the window | (layout stacks per the CSS) |

### 10.3 Visual comparison (after the self-check passes)

For each state that has a reference:
- Screenshot your page and the reference at 1440 × 900.
- List the differences in **look**: colours, gradients, borders, radius, shadows, spacing, sizes, font family/weight, icon, position, missing or extra elements.
- Fix them and screenshot again.
- Data, column sets and labels may differ (they come from the app). Everything else should match.

### 10.4 Behaviour regression (per page, before and after must behave the same)

- **Tables:**
  - resize a column by dragging the handle; the handle glows teal while dragging;
  - drag a column to a new position, if supported;
  - sort ascending/descending; the arrow rotates;
  - pin and unpin a row;
  - hide and show a column with its chip;
  - Rows per page 15 / 25 / 50;
  - next / previous page;
  - **click a row → the existing drawer opens**;
  - hover a row → click the pencil → the existing edit opens, and the drawer does **not** open;
  - the OLA button works.
- **Header and toolbar:**
  - Presets: save a preset, apply it, star it, delete it (toasts appear);
  - Export CSV and Excel;
  - Refresh (and Re-evaluate);
  - the funnel filter;
  - search;
  - click a fact (filters) and click it again (clears).
- **Application Management:**
  - switch every tab;
  - Add New: each item works;
  - the ITAM items are role-gated as before;
  - Select Application works on Features and Actions.
- **Upload:** open, pick a file, cancel.
- **Entitlement Users:**
  - ITAM Name → Application loads accounts;
  - the tabs switch;
  - quick search works;
  - Create Account works.
- **Reports:**
  - application loads;
  - fields toggle;
  - formats switch;
  - Generate Preview;
  - search the preview;
  - Show 10 more / Show all;
  - Download Report.
- **FMID Reports:**
  - required validation;
  - All Users / Specific IDs;
  - Fetch;
  - Detail / Matrix;
  - filters and their tooltips;
  - COLUMNS groups;
  - search;
  - Export CSV / Excel / JSON;
  - Download Security Matrix;
  - Reset / Clear all.
- The **dock highlight glides** between pages, and the dock names are unchanged.

### 10.5 Done means

- Every state in §10.2 reports **0 FAIL**, with the fonts loaded. Every WARN is either fixed or explained in REPORT.md.
- No visible look differences remain against the references.
- §10.4 all works.
- Every INVENTORY.md line is still present.

---

## 11. Final report: `fmces-ui-kit/out/REPORT.md`, then a short summary to me

1. **Self-check results:** one line per state of §10.2, the exact `summary` string, for example `FMCES verify v2 — page: data-profiles — 74 PASS, 0 FAIL, 0 WARN`.
2. **Screenshots:** before / after per page, plus the hover state, in `out/before/` and `out/after/`.
3. **Inventory:** INVENTORY.md with every line ticked. The FMID **Matrix columns** are listed and confirmed as unchanged.
4. **Decisions:** DECISIONS.md, including every library component you kept, how you styled it, and every default you used.
5. **Files changed**, and how to run the self-check again.
6. Anything you could not match, and why. This should be empty.

Then give me a short summary in chat, with the table of self-check lines.
