# FMCES: rebuild the Data Profile and Data Policy pages (create and edit)

Rebuild the look and layout of 4 pages in FMCES, an internal "Markets Operations" app (dark navy with a teal accent):
- Create data profile
- Edit data profile
- Create data policy
- Edit data policy

Match this spec exactly, and do not change behaviour. You get no images; everything is here. Use the given numbers — never adjust by eye.

## 0. Rules

**Keep exactly as they are today:**
- Routes.
- API calls, payload shapes and field names.
- The form-state approach.
- Validation rules: required fields, max lengths, formats and server errors. Restyle errors only.
- What Save and Submit do, the toasts, and where the user lands after submit.
- Permissions.
- Any ids, names or data-testids used by tests.

**Do not touch** the header, the sidebar dock, the list pages, the detail drawer, or any other page. No new endpoints and no backend changes.

**[IF DATA EXISTS]** means: build it only if the page already has that data, or an existing API returns it. Otherwise leave it out completely — no placeholders.

**Remove from these pages:**
- Glass and blur panels.
- Blue left bars on section titles.
- ALL-CAPS section titles.
- Floating labels, and placeholders used as labels.
- `**` required markers.
- Key/Operator/Value labels repeated on every condition row.
- Mixed field heights.
- Full-width buttons.

**Sizing:** no `zoom`, `transform: scale()`, vw-based font sizes, or root-rem tricks. Every size comes from the CSS variables, which are px values switched by media queries.

**CSS files:**
- Appendix A goes into `create-forms.css` **only if that file does not exist yet**. An earlier prompt may already have created a fuller version — keep it.
- Appendix B always goes into `create-forms-extra.css`, loaded **after** `create-forms.css`.
- Both are plain global stylesheets, copied verbatim, even if the app uses CSS modules or Tailwind. Do not translate them.
- Everything is scoped under `.cf`.

**Components:**
- Reuse the shared components from the other create pages if they exist: FormPage, PageHeader, FormSection, Field, SearchSelect, ProgressRail, ActionBar. Otherwise create them.
- Also build TokenField, ConditionBuilder (with ConditionRow and ValuesInput) and PickerSheet. Keep them generic — the Entitlement page will reuse them.

**Fonts:** add this once if it is missing:
`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

### Step 0 — map the current code before you change it

Write the mapping in your first message or in the PR description:

1. **Fields:** the component for each page, and every field with its state key, whether it is required, its max length or format, and its control type.
2. **Profile data source:**
   - where the Data Source Name options come from;
   - whether Data Source Attribute depends on the chosen source;
   - whether Attribute Name is typed or filled in automatically.
3. **Conditions (both pages):**
   - the condition's shape in state and in the payload;
   - whether the value is one string or a list;
   - where the Key and Operator options come from;
   - any per-operator rules (for example, one value only);
   - whether conditions combine with AND or OR;
   - whether at least one condition is required.
4. **Policy:**
   - whether it allows one application or several;
   - how "Add Data Profile" works today: the UI, where the options come from, whether they are filtered by application, and whether the payload holds IDs or objects;
   - what Global Policy changes, including whether it disables the application field.
5. **Buttons:** the Save, Submit and Reset handlers.
6. **Edit pages:**
   - the load call;
   - which fields are locked;
   - the update handler and payload;
   - whether there is a Save as well as Submit;
   - whether the record has status, version and last-modified data;
   - whether editing an approved record creates a new draft version or continues an open draft.

**Order of work:** shared parts → the 2 create pages → the 2 edit pages → the checks in §8.

## 1. Look

**Density tiers**, picked by viewport — Compact (default) / Comfortable (≥1440×820) / Large (≥1880×980):

| | Compact | Comfortable | Large |
|---|---|---|---|
| Inputs, selects, search selects, buttons | 30px | 34px | 36px |
| Small buttons | 26px | 28px | 28px |
| Chips | 22px | 24px | 24px |
| Action bar | 52px | 60px | 64px |
| Body text | 12.5px | 13px | 13.5px |
| Labels (weight 700) | 11.5px | 12px | 12px |
| Help and error text | 10.5px | 11px | 11px |
| Page title (weight 800) | 19px | 22px | 24px |
| Section title (weight 800) | 13.5px | 14.5px | 15px |
| Progress rail width | 256px | 288px | 320px |
| Max page width (left aligned) | 1320px | 1360px | 1480px |

**Colours:**
- Page `#0F1A22`, fields `#111E27`, cards `#152733`, borders `#23394A`.
- Text `#E6EEF2`, secondary text `#A7B8C4`, muted text `#86A0AE`.
- Accent `#2DD4BF`, with `#062A26` text on it.
- Error `#F4A3A3`. Draft `#F6C177`.

**Fonts:**
- Manrope for all text.
- JetBrains Mono (`.cf-input--mono` / `.cf-mono`) for IDs, codes, data source names and attributes, condition keys and values, owners and system names.

**Icons:**
- 24×24 stroke icons, stroke width 1.75 (2.25 where marked `bold`), colour `currentColor`.
- If the app uses lucide, use: ChevronLeft, ChevronDown, X, Search, Plus, Trash2, Check, CheckCircle2, AlertTriangle, RotateCcw, Save, Send, AppWindow, Lock, Layers, ListFilter. Otherwise use the paths in Appendix D.
- In the markup, `[icon:name 16]` means that icon at 16px.

## 2. Page shell (all 4 pages) — markup C.1, C.2

**Mounting**
- Put the page inside the existing content area (right of the dock, below the header).
- The root is `<form class="cf" novalidate>`. If the page already has a `<form>`, add the `cf` class to it instead.
- The content area must be the scroll container, with a fixed height. `.cf` has `min-height: 100%`; without a fixed-height parent, use `min-height: calc(100vh - <header height in px>)`.
- `.cf-bar` is `position: sticky; bottom: 0`. No element between it and the scroll container may have `overflow: hidden` or `overflow: auto`.

**Header**
- Back button (to the list page), then a mono breadcrumb (every part except the last is a link), the title (sentence case) and a one-line subtitle.
- Right side: the badge `New`, which becomes `Draft` after the first successful Save.

**Layout:** `.cf-ledger` — section cards on the left, a sticky progress rail on the right. Below 1100px the rail hides, and the bar shows `.cf-bar-progress` ("· X of Y required") instead.

**Section card**
- A number tile:
  - `is-done` — a teal check, when all the section's required fields are valid;
  - `is-error` — a red "!", while the section shows an error.
- A title, a one-line description, and a body on a 12-column `.cf-grid` with `.cf-span-4/6/12`. Every field goes full width when the card is narrower than 600px.

**Rail**
- **Card 1:** `PROGRESS` with `X of Y required` (Y = required fields from today's validation; X = how many are valid), a teal progress bar, and one `.cf-toc-item` per section.
  - Meta per item: `Done`; `1 issue` / `2 issues` (red, with `is-error`); `Optional` when the section has no required fields; or the counts named on each page.
  - Clicking an item smoothly scrolls to its section and focuses its first field.
  - The section containing focus gets `is-current`.
- **Card 2:** `AFTER YOU SUBMIT`, with these steps:
  - Draft — "You can keep editing" (now)
  - Pending approval — "A checker reviews it"
  - Approved — "Locked as V1"
  - Active — "Ready to use"

**Fields**
- The label sits above the control (linked with `for`/`id`).
- Required: one red `*` (`.cf-req`, `aria-hidden`) plus `required`. Optional: `<span class="cf-opt">Optional</span>`.
- Under the control there is **one** line: help text (`.cf-help`), or an error (`.cf-error` with the alert icon), which replaces the help. Link it with `aria-describedby`.
- An invalid control gets `.is-invalid` and `aria-invalid`.
- Counter `.cf-count` (`used / max`, plus `.is-over` when over the limit) — only where a max length already exists.
- Read-only fields use `readonly` (dashed). Disabled fields show at 50% opacity.

**Controls**
- **Text:** `.cf-input`.
- **Textarea:** `textarea.cf-input`.
- **Select:** a native select inside `.cf-select`, with `.is-empty` until a value is chosen.
- **Search select:** `.cf-combo > .cf-control` — icon, input, optional mono `.cf-control-sub`, clear button `.cf-control-btn`, chevron. Options appear in `.cf-menu` / `.cf-menu-item` (`is-active`, mono `.cf-menu-meta`).
  - Typing filters the existing options. ↑ ↓ move, Enter picks, Esc closes, and a click outside closes.
  - With no results, show "No matches" (`.cf-menu-empty`).
  - The menu is at most 280px tall and scrolls.
  - Use combobox / listbox ARIA.
  - If the app already has a combobox component, restyle it instead of building a new one.

**Action bar**
- **Left** (`.cf-bar-meta`):
  - after a successful Save: `✓ Draft saved · 2 min ago` (relative time, refreshed every 60s; show nothing before the first save);
  - `* Required`;
  - after a failed submit: `N fields need attention` (`.is-error`).
- **Right**, in this order:
  - `Reset` (ghost, rotate icon) — returns the form to its initial values.
  - `Save draft` (outline) — today's Save; only if a Save exists.
  - Primary `Submit for approval` — today's Submit. Label it just `Submit` if it does not start an approval.
- While a request runs, its button gets `.is-loading` and the others are disabled.

**Validation**
- Never show errors on untouched fields. Validate on blur.
- On Submit: validate everything, show the count in the bar, then scroll to and focus the first invalid field.
- Save draft runs only today's Save rules.
- Server errors for a field go under that field. Other server errors go in `.cf-alert cf-alert--error` at the top of the first section.

## 3. Create data profile — markup C.3

- Breadcrumb: `FMCES / Data Profiles / New`
- Title: `Create data profile`
- Subtitle: `Define profile details, application scope and access conditions.`

**1. Profile details** — "Name the profile and pick the application it applies to."
- Profile name — span 6.
- Application — span 6. A search select with the app-window icon, mono; **[IF DATA EXISTS]** a sub-label such as `10 features`. If several applications are allowed today, use the multi-select (C.7), labelled `Applications`.
- Description — span 12, textarea, 2 rows.

**2. Data source** — "Where the data comes from and which attribute the conditions filter on."
- Data source name — span 4, mono.
- Data source attribute — span 4, mono. If it depends on the source: disabled, with placeholder `Select data source first`, and cleared when the source changes.
- Data source attribute name — span 4. Read-only if it is filled in automatically.
- Keep today's control types.

**3. Profile conditions** — "Rows are visible only when every condition matches." (with OR: "Rows are visible when any condition matches.")
- The condition builder (§5).
- Rail meta when valid: `2 conditions`.

## 4. Create data policy

- Breadcrumb: `FMCES / Data Policies / New`
- Title: `Create data policy`
- Subtitle: `Define data policy details, applications, profiles and conditions.`

**1. Policy details** — "Name the policy and pick the application it applies to."
- Policy name — span 6.
- Application — span 6, same as on the profile page.
- Description — span 12.
- Global policy — span 12. A `.cf-switch` replaces today's checkbox, with the same state key, value and side effects. If it disables the application field today, it must do so here too.

**2. Data profiles** — "The profiles whose rows this policy uses."
- `Profiles` — span 12, required as today. A token field (C.4): one chip per profile (name, mono `#ID`, ×) and a dashed `+ Add profile` button (`aria-haspopup="dialog"`) that opens the picker (§6).
- **None selected:** show the empty box `No profiles selected` with a `+ Add data profile` button.
- **If profiles depend on the application and none is chosen:** disable the add buttons and show the help line `Pick an application first.`
- **Failed submit with no profiles:** `.is-invalid` and the error `Add at least one profile`.

**3. Policy conditions** — same as on the profile page.

**Rail meta:** `1 profile` / `2 profiles`; `2 conditions`.

## 5. Condition builder (shared) — markup C.5

**Layout**
- **Header row, once** (`.cf-cond-head`): KEY* · OPERATOR* · VALUES*. Never repeat labels on each row.
- **Each row** `.cf-cond` has `role="group"` and an `aria-label` like `Condition 2`. It holds:
  - Key — a select or search select, mono;
  - Operator — a select;
  - Values;
  - a trash icon button.

  Every control has an `aria-label`, for example `Key, condition 2`.
- **Between rows:** `.cf-cond-join` with `AND` or `OR`, matching the backend. If you cannot tell which, show the divider without a word and change the section description to "Rows are visible only when the conditions match."
- **Foot:** a small outline `+ Add condition` button. With AND, add the note `All conditions must match.`
- **No rows:** the empty box `No conditions added yet.` (list-filter icon) with `+ Add condition`. If at least one condition is required, a failed submit adds `.is-invalid` and the error `Add at least one condition`.

**Values**
- **If the value is a list today:** a `.cf-tokens` box with mono removable chips and an inline input.
  - Placeholder: `Type a value, press Enter` when empty, `Add value` once it has chips.
  - Enter or comma turns the text into a chip. Trim it, and ignore empty values and duplicates within the row.
  - Backspace in the empty input removes the last chip.
  - Pasting splits on commas, semicolons and line breaks.
  - × removes a chip.
  - If an operator takes one value: one chip only, then hide the input.
- **If the value is a single string today:** a plain `.cf-input cf-input--mono`.

**Behaviour**
- `Add condition` appends a row and focuses its Key.
- The trash button removes the row with no confirm, then focuses the next row's Key or the Add button.
- Clear a row's values when its Key changes **only** if today's code does.
- Keep the rows in order. The payload is today's array.

**Validation** — on touched rows, or on all rows after a submit attempt. The first missing item wins: `Pick a key` → `Pick an operator` → `Add at least one value`.
- Show one `.cf-error` line; the CSS places it under Values.
- The missing control gets `.is-invalid` and `aria-invalid`.
- Add no new rules.

**Section status:** `is-done` when every row is valid and the minimum is met (meta `N conditions`); `is-error` while any row shows an error.

## 6. Profile picker sheet — markup C.6

Restyle today's picker. Keep its data, search, filtering and selection logic.

**Placement**
- A `.cf-scrim` plus a `.cf-sheet` with `role="dialog"`, `aria-modal` and `aria-labelledby`.
- Right side, full height, width `clamp(380px, 32vw, 480px)`; full width below 900px.
- Render it inside the `.cf` form. If dialogs portal out of the page, wrap it in `<div class="cf cf-portal">`.

**Head:** `Select data profiles` and a close (×) button.

**Tools:**
- A search box: `Search by name, ID or owner`. If today's search covers fewer fields, adjust the placeholder.
- A segmented filter `.cf-seg` (`All N` / `Selected N`, using `aria-pressed`).
- **[IF DATA EXISTS]** a note on the right: `Profiles in <application>`.

**List:** one `.cf-option` label row per profile, so clicking anywhere toggles it. Each row has:
- a checkbox;
- the name (bold, ellipsis);
- a mono meta line `Owner <owner> · <application>` (only the parts that exist);
- a mono `#ID`.

Selected rows get `.is-selected`. While loading, show three `.cf-skel` placeholders, 44px tall. With no match, show muted `No profiles match "<query>"`.

**Foot:** `N selected` · `Clear` (`.cf-link`) · Cancel (ghost) · Confirm (primary, check icon).
- Confirm applies the selection the same way today's picker does.
- Cancel, ×, Esc and clicking the scrim close the sheet without changes.

**Focus:** goes to the search box on open, stays trapped inside the sheet, and returns to `Add profile` on close.

## 7. Edit pages — markup C.8, C.9

Use the same page component with `mode: 'create' | 'edit'`; both existing routes render it. Load with the existing fetch, and save with the existing update call and **exactly the payload the old edit page sends**.

**Header**
- Title `Edit data profile` / `Edit data policy`. Breadcrumb `… / <record name> / Edit`.
- Subtitle `Update the details below.` If edits need approval, add ` Changes are reviewed before they replace the live version.`
- **[IF DATA EXISTS]** a `.cf-head-meta` line under the subtitle, for example `ID 71 · Owner 2023504 · Last modified 29 Sep 2026 by g.fmces.001.dev`.
- Badges: the current status — `cf-badge--ok` (Approved), `--draft` or `--pending`, using the app's own text — plus **[IF DATA EXISTS]** the version, `cf-badge--version` (`V1`).

**Loading values**
- Prefill every field.
- Load the data source before its attribute, and the application before the profiles. Do not run the "clear dependent field" logic during the initial load.

**Read-only fields**
- Lock **exactly** the fields today's edit page locks — no more, no fewer.
- A search select becomes `.cf-control.is-readonly` with a trailing `[icon:lock 14]`; a text field gets `readonly`.
- No `*` on locked fields. Add one help line, for example `A profile can’t move to another application.`

**Changes** — compared with the loaded values, trimmed:
- A changed field's label gets `<span class="cf-edited">Edited</span>` (after the `*` / Optional, before the counter) and the control gets `.is-changed`. Both disappear when the value goes back.
- **Global policy:** the tag goes after the switch text.
- **Profiles:** compare the sets of IDs; the box gets `.cf-tokens.is-changed`.
- **Conditions:** new or changed rows get `.cf-cond.is-changed` (a teal dot). Removed rows also count as changes.
- **Change count:** each field = 1, the profiles set = 1, each added, changed or removed condition = 1.

**Bar**
- Left: `N unsaved changes` (`<span class="is-changed">`, or "1 unsaved change"), or `No changes` (muted); then `* Required`, then any errors. After Save draft: `✓ Draft saved · just now`.
- Buttons:
  - `Discard changes` — ghost, rotate icon; returns to the loaded values; disabled when there are no changes.
  - `Save draft` — only if the edit page has Save.
  - Primary — `Submit for approval` if it starts an approval, otherwise `Save changes`. Disabled while there are no changes.
- Keep any existing unsaved-changes guard. Do not add a new one.

**Rail**
- Per-section priority: error > changes (teal `1 change`, `.cf-toc-meta.is-changed`) > the create-page rules.
- **[IF DATA EXISTS: versioning]** Lifecycle for an approved record, using the real version numbers:
  - Draft V2 — "You can keep editing" (now)
  - Pending approval
  - Approved — "Replaces V1"
  - Active — "V1 stays live until then"
- **If an open draft exists** (the drawer shows "Continue editing"): edit that draft. The badges show `Draft` and its version, and the lifecycle's "now" step is that draft.

**Loading state**
- The same layout with `.cf-skel` placeholders, and the bar buttons disabled.
- On failure: `.cf-alert cf-alert--error` with the existing message and a small outline `Retry` button that calls the load again.
- Nothing may jump when the data arrives.

## 8. Checks

Run these in Chrome at 1366×768, 1440×900, 1920×1080 and 1024×768.

1. **Sizes:** controls measure 30 / 34 / 36px and body text 12.5 / 13 / 13.5px at the first three sizes. Measure them.
2. **Action bar:** stays pinned while you scroll Create data profile at 1366×768, and its buttons line up with the content's right edge. On narrow windows the left text gets an ellipsis and the buttons never overlap.
3. **Layout:**
   - No horizontal page scroll.
   - At 1024×768 the rail is hidden and the bar shows `· X of Y required`.
4. **Fields:**
   - Labels sit above fields; required fields have one red `*`; optional fields have the `Optional` tag.
   - No `**`, floating labels, caps titles, left bars or glass panels.
   - Condition labels appear only in the header row.
5. **Conditions:**
   - Add and remove rows; focus moves correctly.
   - Enter, comma and Backspace work, and pasting "AR, AU" makes 2 chips.
   - An empty row shows `Add at least one value` under Values.
   - The empty state shows when there are no rows.
   - The payload is identical to the old page's (check the Network tab).
6. **Dependent attribute:** disabled until a source is chosen, and cleared when the source changes.
7. **Picker:**
   - Opens on the right, full height, with a scrim.
   - Search and the Selected filter work.
   - Confirm fills the chips; Cancel, ×, Esc and the scrim change nothing.
   - Focus returns to `Add profile`.
   - A chip's × removes it.
8. **Global policy:** sends the same value as before, with the same side effects.
9. **Edit pages:**
   - Stored values show, including chips and conditions.
   - Placeholders show while loading, and nothing jumps.
   - The Edited tag, the condition dot and the change count update; reverting a value clears them.
   - Discard restores everything. The primary button is disabled when there are no changes.
   - The locked fields are the same as on the old page.
   - The payload matches the old edit page's.
10. **Final:**
    - Save, Submit and Reset behave as before, with the same toasts and navigation.
    - No console errors. Tests pass; update snapshots only for markup changes.
    - Keep every `aria-*`, role and label shown in the markup. Never remove focus outlines.

## Appendix A — `create-forms.css` (base; create it only if the file does not exist)

```css
.cf{
  --c-bg:#0F1A22;
  --c-field:#111E27;
  --c-card:#152733;
  --c-card-2:#1B3140;
  --c-border:#23394A;
  --c-border-hover:#2E4A5D;
  --c-hair:#1E3240;
  --c-btn-border:#2B4556;
  --c-text:#E6EEF2;
  --c-text-2:#A7B8C4;
  --c-muted:#86A0AE;
  --c-placeholder:#7890A0;
  --c-accent:#2DD4BF;
  --c-accent-strong:#5EEAD4;
  --c-accent-tint:rgba(45,212,191,.12);
  --c-accent-ring:rgba(45,212,191,.16);
  --c-on-accent:#062A26;
  --c-link:#7DBBFF;
  --c-err:#F4A3A3;
  --c-err-tint:rgba(244,163,163,.10);
  --c-err-line:rgba(244,163,163,.55);
  --c-warn:#F6C177;
  --c-warn-tint:rgba(246,193,119,.13);
  --c-info:#A9D1FF;
  --c-info-tint:rgba(125,187,255,.12);
  --c-bar:rgba(15,26,34,.92);
  --c-focus:#5EEAD4;
  --ease:cubic-bezier(.22,1,.36,1);
  --spring:cubic-bezier(.34,1.36,.64,1);
  --font:'Manrope',system-ui,-apple-system,'Segoe UI',sans-serif;
  --mono:'JetBrains Mono',ui-monospace,'Cascadia Mono',Consolas,monospace;
  --fs-xs:10.5px;
  --fs-sm:11.5px;
  --fs-md:12.5px;
  --fs-lg:13.5px;
  --fs-mono:12px;
  --fs-h1:19px;
  --fs-h2:13.5px;
  --h-input:30px;
  --h-btn:30px;
  --h-btn-sm:26px;
  --h-chip:22px;
  --h-bar:52px;
  --h-row:38px;
  --r:7px;
  --r-card:10px;
  --gap-x:12px;
  --gap-y:12px;
  --gap-label:5px;
  --gap-sec:12px;
  --sec-pad:14px 16px 16px;
  --sec-head-pad:12px 16px 0;
  --page-pad-l:16px;
  --page-pad-r:24px;
  --page-pad-top:16px;
  --rail-w:256px;
  --center-w:720px;
  --page-max:1320px}
@media (min-width:1440px) and (min-height:820px){
  .cf{--fs-xs:11px;--fs-sm:12px;--fs-md:13px;--fs-lg:14px;--fs-mono:12.5px;--fs-h1:22px;--fs-h2:14.5px;--h-input:34px;--h-btn:34px;--h-btn-sm:28px;--h-chip:24px;--h-bar:60px;--h-row:42px;--r:8px;--r-card:12px;--gap-x:16px;--gap-y:14px;--gap-label:6px;--gap-sec:16px;--sec-pad:18px 20px 20px;--sec-head-pad:16px 20px 0;--page-pad-l:16px;--page-pad-r:32px;--page-pad-top:20px;--rail-w:288px;--center-w:780px;--page-max:1360px}
}
@media (min-width:1880px) and (min-height:980px){
  .cf{--fs-md:13.5px;--fs-lg:14.5px;--fs-mono:13px;--fs-h1:24px;--fs-h2:15px;--h-input:36px;--h-btn:36px;--h-row:44px;--h-bar:64px;--gap-x:20px;--gap-y:16px;--gap-sec:20px;--sec-pad:20px 24px 24px;--sec-head-pad:18px 24px 0;--page-pad-r:40px;--page-pad-top:24px;--rail-w:320px;--center-w:820px;--page-max:1480px}
}
.cf{display:flex;flex-direction:column;min-height:100%;background:var(--c-bg);color:var(--c-text);font-family:var(--font);font-size:var(--fs-md);line-height:1.45;color-scheme:dark;-webkit-font-smoothing:antialiased}
.cf *,.cf *::before,.cf *::after{box-sizing:border-box}
.cf button{font-family:inherit;cursor:pointer}
.cf h1,.cf h2,.cf h3,.cf p{margin:0}
.cf :focus-visible{outline:2px solid var(--c-focus);outline-offset:2px}
.cf a{color:var(--c-link);text-decoration:none}
.cf a:hover{color:#B3D7FF}
.cf svg{display:block;flex-shrink:0}
.cf .cf-mono{font-family:var(--mono);font-size:var(--fs-mono)}
.cf-inner{width:100%;max-width:var(--page-max);display:flex;flex-direction:column;flex:1 0 auto;padding:0 var(--page-pad-r) 24px var(--page-pad-l)}
.cf-head{display:flex;align-items:flex-start;gap:14px;padding:var(--page-pad-top) 0 calc(var(--gap-sec) - 2px)}
.cf-back{width:var(--h-btn);height:var(--h-btn);flex-shrink:0;display:flex;align-items:center;justify-content:center;margin-top:14px;border-radius:9px;border:1px solid var(--c-btn-border);background:var(--c-card);color:var(--c-text-2)}
.cf-back:hover{background:var(--c-card-2);color:var(--c-text)}
.cf-head-text{display:flex;flex-direction:column;min-width:0}
.cf-crumb{font-family:var(--mono);font-size:var(--fs-xs);color:var(--c-muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-crumb a{color:var(--c-muted)}
.cf-crumb a:hover{color:var(--c-text)}
.cf-title{font-size:var(--fs-h1);font-weight:800;letter-spacing:-.015em;line-height:1.25;margin-top:3px}
.cf-sub{font-size:var(--fs-md);color:var(--c-text-2);margin-top:2px}
.cf-head-right{margin-left:auto;display:flex;align-items:center;gap:10px;margin-top:14px}
.cf-ledger{display:grid;grid-template-columns:minmax(0,1fr) var(--rail-w);gap:calc(var(--gap-sec) + 8px);align-items:start}
.cf-stack{display:flex;flex-direction:column;gap:var(--gap-sec);min-width:0}
@media (max-width:1099px){
  .cf-ledger{grid-template-columns:minmax(0,1fr)}
  .cf-rail{display:none !important}
  .cf-bar-progress{display:inline-flex !important}
}
.cf-sec{background:var(--c-card);border:1px solid var(--c-border);border-radius:var(--r-card);container-type:inline-size;scroll-margin-top:16px}
.cf-sec-head{display:flex;align-items:flex-start;gap:12px;padding:var(--sec-head-pad)}
.cf-sec-num{width:24px;height:24px;flex-shrink:0;display:flex;align-items:center;justify-content:center;margin-top:1px;border-radius:7px;background:var(--c-card-2);border:1px solid var(--c-btn-border);color:var(--c-accent-strong);font:600 11.5px var(--mono);transition:background-color .25s,border-color .25s,color .25s}
.cf-sec-num.is-done{background:var(--c-accent);border-color:var(--c-accent);color:var(--c-on-accent)}
.cf-sec-num.is-error{background:var(--c-err-tint);border-color:var(--c-err-line);color:var(--c-err)}
.cf-sec-titles{flex:1;min-width:0}
.cf-sec-title{font-size:var(--fs-h2);font-weight:800;line-height:1.35}
.cf-sec-desc{font-size:var(--fs-sm);color:var(--c-muted);margin-top:1px}
.cf-sec-body{padding:var(--sec-pad)}
.cf-sec-foot{display:flex;align-items:center;gap:10px;padding:12px 20px;border-top:1px solid var(--c-hair);font-size:var(--fs-xs);color:var(--c-muted)}
.cf-grid{display:grid;grid-template-columns:repeat(12,minmax(0,1fr));gap:var(--gap-y) var(--gap-x)}
.cf-span-4{grid-column:span 4}
.cf-span-5{grid-column:span 5}
.cf-span-6{grid-column:span 6}
.cf-span-7{grid-column:span 7}
.cf-span-8{grid-column:span 8}
.cf-span-12{grid-column:span 12}
@container (max-width:600px){
  .cf-grid>[class*="cf-span-"]{grid-column:span 12}
}
.cf-field{display:flex;flex-direction:column;gap:var(--gap-label);min-width:0}
.cf-label{display:flex;align-items:center;gap:6px;font-size:var(--fs-sm);font-weight:700;color:var(--c-text-2);line-height:1.3}
.cf-req{color:var(--c-err);font-weight:800;margin-left:-3px}
.cf-opt{font-size:var(--fs-xs);font-weight:600;color:var(--c-muted)}
.cf-count{margin-left:auto;font:500 10.5px var(--mono);color:var(--c-muted)}
.cf-count.is-over{color:var(--c-err)}
.cf-input{display:block;width:100%;height:var(--h-input);padding:0 10px;margin:0;border-radius:var(--r);border:1px solid var(--c-border);background:var(--c-field);color:var(--c-text);font:inherit;font-size:var(--fs-md);outline:none;transition:border-color .15s,box-shadow .15s,background-color .15s}
.cf-input:hover{border-color:var(--c-border-hover)}
.cf-input::placeholder{color:var(--c-placeholder);opacity:1}
.cf-input:focus,.cf-control:focus-within{border-color:var(--c-accent);box-shadow:0 0 0 3px var(--c-accent-ring)}
.cf-input--mono,.cf-input--mono input{font-family:var(--mono);font-size:var(--fs-mono)}
.cf-input.is-invalid,.cf-control.is-invalid{border-color:var(--c-err-line);background:linear-gradient(var(--c-err-tint),var(--c-err-tint)),var(--c-field)}
.cf-input.is-invalid:focus,.cf-control.is-invalid:focus-within{box-shadow:0 0 0 3px var(--c-err-tint)}
.cf-input[readonly],.cf-input.is-readonly{background:transparent;border-style:dashed;color:var(--c-text-2)}
.cf-input:disabled,.cf-control.is-disabled{opacity:.5;cursor:not-allowed}
textarea.cf-input{height:auto;min-height:calc(var(--h-input) * 2.2);padding:8px 10px;line-height:1.5;resize:vertical}
.cf-select{position:relative;display:block}
.cf-select select.cf-input{appearance:none;-webkit-appearance:none;padding-right:32px;cursor:pointer}
.cf-select select.cf-input:invalid,.cf-select select.cf-input.is-empty{color:var(--c-placeholder)}
.cf-select>svg{position:absolute;right:10px;top:50%;margin-top:-8px;color:var(--c-muted);pointer-events:none}
.cf-select option{background:#172A36;color:var(--c-text)}
.cf-control{display:flex;align-items:center;gap:8px;width:100%;height:var(--h-input);padding:0 8px 0 10px;border-radius:var(--r);border:1px solid var(--c-border);background:var(--c-field);color:var(--c-text);transition:border-color .15s,box-shadow .15s}
.cf-control:hover{border-color:var(--c-border-hover)}
.cf-control>svg{color:var(--c-muted)}
.cf-control input{flex:1;min-width:0;height:100%;padding:0;border:0;outline:0;background:transparent;color:inherit;font:inherit;font-size:var(--fs-md)}
.cf-control input::placeholder{color:var(--c-placeholder);opacity:1}
.cf-control-sub{font:500 11px/1.5 var(--mono);color:var(--c-muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:45%}
.cf-control-btn{width:22px;height:22px;flex-shrink:0;display:inline-flex;align-items:center;justify-content:center;padding:0;border:0;border-radius:5px;background:transparent;color:var(--c-muted)}
.cf-control-btn:hover{background:var(--c-card-2);color:var(--c-text)}
.cf-menu{position:absolute;z-index:60;left:0;right:0;top:calc(100% + 4px);max-height:280px;overflow:auto;padding:6px;background:#172A36;border:1px solid #2B4556;border-radius:10px;box-shadow:0 20px 44px -16px rgba(0,0,0,.75)}
.cf-menu-item{display:flex;align-items:center;gap:10px;width:100%;min-height:var(--h-input);padding:0 10px;border:0;border-radius:7px;background:transparent;color:var(--c-text);font-size:var(--fs-md);font-weight:600;text-align:left}
.cf-menu-item:hover,.cf-menu-item.is-active{background:var(--c-card-2)}
.cf-menu-item>svg{color:var(--c-accent)}
.cf-menu-item .cf-menu-meta{margin-left:auto;font:500 11px/1.5 var(--mono);color:var(--c-muted);white-space:nowrap}
.cf-menu-empty{padding:10px;font-size:var(--fs-sm);color:var(--c-muted)}
.cf-combo{position:relative}
.cf-help{font-size:var(--fs-xs);color:var(--c-muted);line-height:1.4}
.cf-error{display:flex;align-items:center;gap:5px;font-size:var(--fs-xs);color:var(--c-err);line-height:1.4}
.cf-chip{display:inline-flex;align-items:center;height:var(--h-chip);max-width:100%;padding:0 9px;border-radius:6px;background:var(--c-card-2);border:1px solid var(--c-btn-border);color:var(--c-text);font-size:var(--fs-sm);font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-chip--mono{font-family:var(--mono);font-size:11.5px}
.cf-switch{position:relative;display:inline-flex;align-items:center;gap:9px;font-size:var(--fs-sm);font-weight:600;color:var(--c-text-2);cursor:pointer;white-space:nowrap}
.cf-switch input{position:absolute;opacity:0;width:1px;height:1px}
.cf-switch-track{position:relative;width:30px;height:18px;flex-shrink:0;border-radius:999px;background:#2A4253;transition:background-color .2s}
.cf-switch-track::after{content:"";position:absolute;left:2px;top:2px;width:14px;height:14px;border-radius:50%;background:#C9D6DD;transition:transform .22s var(--spring),background-color .2s}
.cf-switch input:checked + .cf-switch-track{background:var(--c-accent)}
.cf-switch input:checked + .cf-switch-track::after{transform:translateX(12px);background:#fff}
.cf-switch input:focus-visible + .cf-switch-track{outline:2px solid var(--c-focus);outline-offset:2px}
.cf-btn{display:inline-flex;align-items:center;justify-content:center;gap:7px;height:var(--h-btn);padding:0 14px;border-radius:var(--r);border:1px solid transparent;font-size:var(--fs-md);font-weight:700;white-space:nowrap;transition:background-color .15s,border-color .15s,color .15s}
.cf-btn--primary{background:var(--c-accent);color:var(--c-on-accent)}
.cf-btn--primary:hover{background:var(--c-accent-strong)}
.cf-btn--secondary{background:transparent;border-color:var(--c-btn-border);color:var(--c-text)}
.cf-btn--secondary:hover{background:var(--c-card)}
.cf-btn--ghost{background:transparent;color:var(--c-text-2);padding:0 10px}
.cf-btn--ghost:hover{background:var(--c-card);color:var(--c-text)}
.cf-btn--sm{height:var(--h-btn-sm);padding:0 10px;font-size:var(--fs-sm);border-radius:7px}
.cf-btn:disabled{opacity:.45;cursor:not-allowed}
.cf-btn.is-loading{pointer-events:none;opacity:.8}
.cf-icon-btn{width:28px;height:28px;flex-shrink:0;display:inline-flex;align-items:center;justify-content:center;padding:0;border:0;border-radius:7px;background:transparent;color:var(--c-muted)}
.cf-icon-btn:hover{background:var(--c-card-2);color:var(--c-text)}
.cf-icon-btn--danger:hover{background:var(--c-err-tint);color:var(--c-err)}
.cf-badge{display:inline-flex;align-items:center;gap:5px;height:20px;padding:0 7px;border-radius:5px;font-size:10.5px;font-weight:800;letter-spacing:.06em;text-transform:uppercase;white-space:nowrap}
.cf-badge::before{content:"";width:6px;height:6px;border-radius:50%;background:currentColor}
.cf-badge--draft{background:var(--c-warn-tint);color:var(--c-warn)}
.cf-badge--new{background:var(--c-info-tint);color:var(--c-info)}
.cf-alert{display:flex;align-items:flex-start;gap:10px;padding:10px 12px;border-radius:9px;font-size:var(--fs-sm);line-height:1.45}
.cf-alert>svg{margin-top:1px}
.cf-alert--error{background:var(--c-err-tint);color:#F8CACA}
.cf-alert--error>svg{color:var(--c-err)}
.cf-rail{position:sticky;top:16px;display:flex;flex-direction:column;gap:12px}
.cf-rail-card{display:flex;flex-direction:column;gap:12px;padding:14px 16px;background:var(--c-card);border:1px solid var(--c-border);border-radius:var(--r-card)}
.cf-rail-row{display:flex;align-items:center;justify-content:space-between;gap:8px}
.cf-rail-title{font-size:var(--fs-xs);font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--c-muted)}
.cf-rail-count{font-size:var(--fs-sm);font-weight:700}
.cf-progress{height:6px;border-radius:999px;background:var(--c-field);border:1px solid var(--c-hair);overflow:hidden}
.cf-progress i{display:block;height:100%;border-radius:999px;background:var(--c-accent);transition:width .4s var(--ease)}
.cf-toc{display:flex;flex-direction:column;gap:2px;margin:0 -8px}
.cf-toc-item{display:flex;align-items:center;gap:10px;min-height:36px;padding:4px 8px;border:0;border-radius:8px;background:transparent;color:var(--c-text);font-size:var(--fs-md);font-weight:600;text-align:left;width:100%}
.cf-toc-item:hover,.cf-toc-item.is-current{background:var(--c-card-2)}
.cf-toc-dot{width:18px;height:18px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;border:1.5px solid #3D5A6C;color:var(--c-muted);font:600 10px var(--mono)}
.cf-toc-item.is-done .cf-toc-dot{background:var(--c-accent);border-color:var(--c-accent);color:var(--c-on-accent)}
.cf-toc-item.is-error .cf-toc-dot{background:var(--c-err-tint);border-color:var(--c-err-line);color:var(--c-err)}
.cf-toc-name{flex:1;min-width:0;line-height:1.3}
.cf-toc-meta{font-size:var(--fs-xs);font-weight:600;color:var(--c-muted);white-space:nowrap}
.cf-toc-item.is-error .cf-toc-meta{color:var(--c-err)}
.cf-life{display:flex;flex-direction:column}
.cf-life-step{display:grid;grid-template-columns:18px minmax(0,1fr);gap:10px;font-size:var(--fs-sm)}
.cf-life-node{display:flex;flex-direction:column;align-items:center}
.cf-life-node i{width:10px;height:10px;margin-top:4px;flex-shrink:0;border-radius:50%;border:2px solid #3A5566;background:var(--c-card)}
.cf-life-node b{flex:1;width:2px;min-height:14px;background:var(--c-hair)}
.cf-life-step:last-child .cf-life-node b{display:none}
.cf-life-step.is-now .cf-life-node i{border-color:var(--c-warn);background:var(--c-warn)}
.cf-life-text{padding-bottom:10px;color:var(--c-text-2)}
.cf-life-step.is-now .cf-life-text{color:var(--c-text);font-weight:700}
.cf-life-text small{display:block;font-size:var(--fs-xs);font-weight:500;color:var(--c-muted)}
.cf-bar{position:sticky;bottom:0;z-index:30;margin-top:auto;background:var(--c-bar);border-top:1px solid var(--c-border);-webkit-backdrop-filter:blur(8px);backdrop-filter:blur(8px)}
.cf-bar-inner{display:flex;align-items:center;gap:10px;width:100%;max-width:var(--page-max);height:var(--h-bar);padding:0 var(--page-pad-r) 0 calc(var(--page-pad-l) + 8px)}
.cf-bar-meta{display:flex;align-items:center;gap:8px;min-width:0;font-size:var(--fs-sm);color:var(--c-muted);white-space:nowrap}
.cf-bar-meta .is-error{color:var(--c-err)}
.cf-bar-spacer{flex:1}
.cf-saved{display:inline-flex;align-items:center;gap:6px}
.cf-saved>svg{color:var(--c-accent)}
.cf-bar-progress{display:none;align-items:center;gap:6px}
@media (prefers-reduced-motion:reduce){
  .cf *,.cf *::before,.cf *::after{transition-duration:.01ms !important;animation-duration:.01ms !important}
}
```

## Appendix B — `create-forms-extra.css` (always; load it after A)

```css
.cf-link{display:inline-flex;align-items:center;gap:5px;padding:0;border:0;background:transparent;color:var(--c-link);font-size:var(--fs-sm);font-weight:700}
.cf-link:hover{color:#B3D7FF}
.cf-head-meta{font:500 var(--fs-xs)/1.5 var(--mono);color:var(--c-muted);margin-top:5px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-badge--ok{background:var(--c-accent-tint);color:var(--c-accent-strong)}
.cf-badge--pending{background:rgba(170,150,255,.12);color:#C9B6FF}
.cf-badge--version{background:var(--c-card-2);color:var(--c-text-2);font-family:var(--mono);font-weight:600;letter-spacing:0;text-transform:none}
.cf-badge--version::before{display:none}
.cf-edited{display:inline-flex;align-items:center;gap:4px;font-size:var(--fs-xs);font-weight:700;color:var(--c-accent-strong)}
.cf-edited::before{content:"";width:6px;height:6px;border-radius:50%;background:var(--c-accent)}
.cf-input.is-changed,.cf-control.is-changed{border-color:rgba(45,212,191,.45)}
.cf-control.is-readonly{background:transparent;border-style:dashed;color:var(--c-text-2)}
.cf-control.is-readonly:hover{border-color:var(--c-border)}
.cf-control.is-readonly input{cursor:default}
.cf-toc-meta.is-changed,.cf-bar-meta .is-changed{color:var(--c-accent-strong)}
.cf-skel{display:block;height:var(--h-input);border-radius:var(--r);background:linear-gradient(90deg,#142430 0%,#1B3140 50%,#142430 100%);background-size:200% 100%;animation:cf-shimmer 1.4s ease-in-out infinite}
.cf-skel--label{height:10px;width:35%;border-radius:5px}
@keyframes cf-shimmer{from{background-position:100% 0}to{background-position:-100% 0}}
.cf-tokens{display:flex;flex-wrap:wrap;align-items:center;gap:6px;min-height:var(--h-input);padding:3px 4px;border-radius:var(--r);border:1px solid var(--c-border);background:var(--c-field);transition:border-color .15s,box-shadow .15s}
.cf-tokens:hover{border-color:var(--c-border-hover)}
.cf-tokens:focus-within{border-color:var(--c-accent);box-shadow:0 0 0 3px var(--c-accent-ring)}
.cf-tokens.is-invalid{border-color:var(--c-err-line);background:linear-gradient(var(--c-err-tint),var(--c-err-tint)),var(--c-field)}
.cf-tokens.is-changed{border-color:rgba(45,212,191,.45)}
.cf-tokens.is-disabled{opacity:.5;pointer-events:none}
.cf-tokens input{flex:1 0 110px;min-width:110px;height:var(--h-chip);padding:0 6px;border:0;outline:0;background:transparent;color:var(--c-text);font-family:var(--mono);font-size:var(--fs-mono)}
.cf-tokens input::placeholder{color:var(--c-placeholder);opacity:1;font-family:var(--font);font-size:var(--fs-md)}
.cf-chip--removable{gap:6px;padding:0 3px 0 9px}
.cf-chip-name{min-width:0;overflow:hidden;text-overflow:ellipsis}
.cf-chip-id{font:500 10.5px/1.5 var(--mono);color:var(--c-muted)}
.cf-chip-x{width:18px;height:18px;flex-shrink:0;display:inline-flex;align-items:center;justify-content:center;padding:0;border:0;border-radius:4px;background:transparent;color:var(--c-muted)}
.cf-chip-x:hover{background:var(--c-err-tint);color:var(--c-err)}
.cf-tokens-add{display:inline-flex;align-items:center;gap:5px;height:var(--h-chip);padding:0 8px;border-radius:6px;border:1px dashed var(--c-btn-border);background:transparent;color:var(--c-accent-strong);font-size:var(--fs-sm);font-weight:700;white-space:nowrap}
.cf-tokens-add:hover{background:var(--c-accent-tint);border-color:rgba(45,212,191,.45)}
.cf-empty{display:flex;align-items:center;gap:10px;min-height:var(--h-input);padding:6px 6px 6px 12px;border-radius:var(--r);border:1px dashed var(--c-btn-border);color:var(--c-muted);font-size:var(--fs-sm)}
.cf-empty>svg{color:var(--c-muted)}
.cf-empty-text{flex:1;min-width:0}
.cf-empty.is-invalid{border-color:var(--c-err-line)}
.cf-conds{display:flex;flex-direction:column;gap:8px}
.cf-cond-head,.cf-cond{display:grid;grid-template-columns:minmax(0,1.1fr) minmax(120px,160px) minmax(0,1.7fr) 28px;gap:8px;align-items:start}
.cf-cond-head{padding:0 2px;font-size:var(--fs-xs);font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--c-muted)}
.cf-cond-head .cf-req{margin-left:2px}
.cf-cond{position:relative}
.cf-cond>.cf-icon-btn{margin-top:calc((var(--h-input) - 28px) / 2)}
.cf-cond.is-changed::before{content:"";position:absolute;left:-11px;top:calc(var(--h-input) / 2 - 3px);width:6px;height:6px;border-radius:50%;background:var(--c-accent)}
.cf-cond>.cf-error{grid-column:3 / 4}
.cf-cond-join{display:flex;align-items:center;gap:8px;font:600 10.5px/1 var(--mono);letter-spacing:.08em;color:var(--c-muted)}
.cf-cond-join::before,.cf-cond-join::after{content:"";flex:1;height:1px;background:var(--c-btn-border)}
.cf-cond-foot{display:flex;align-items:center;flex-wrap:wrap;gap:10px;margin-top:4px;font-size:var(--fs-xs);color:var(--c-muted)}
@container (max-width:600px){
  .cf-cond-head{display:none}
  .cf-cond{grid-template-columns:minmax(0,1fr) minmax(0,1fr) 28px}
  .cf-cond>.cf-tokens{grid-column:1 / 3}
  .cf-cond>.cf-error{grid-column:1 / -1}
}
.cf-check{position:relative;display:inline-flex;align-items:center;gap:8px;cursor:pointer;font-size:var(--fs-md)}
.cf-check input{position:absolute;opacity:0;width:1px;height:1px}
.cf-check-box{width:16px;height:16px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:4px;border:1.5px solid #3D5A6C;background:var(--c-field);color:transparent;transition:background-color .15s,border-color .15s}
.cf-check input:checked + .cf-check-box{background:var(--c-accent);border-color:var(--c-accent);color:var(--c-on-accent)}
.cf-check input:focus-visible + .cf-check-box{outline:2px solid var(--c-focus);outline-offset:2px}
.cf-seg{display:inline-flex;align-self:flex-start;gap:2px;padding:3px;border-radius:9px;background:var(--c-field);border:1px solid var(--c-border)}
.cf-seg button{height:calc(var(--h-btn-sm) - 2px);padding:0 12px;border:0;border-radius:6px;background:transparent;color:var(--c-text-2);font-size:var(--fs-sm);font-weight:700}
.cf-seg button[aria-pressed="true"]{background:var(--c-card-2);color:var(--c-text);box-shadow:inset 0 0 0 1px var(--c-btn-border)}
.cf-portal{display:contents}
.cf-scrim{position:fixed;inset:0;z-index:1000;background:rgba(3,8,11,.42)}
.cf-sheet{position:fixed;top:0;right:0;bottom:0;z-index:1001;width:clamp(380px,32vw,480px);display:flex;flex-direction:column;background:#111E27;color:var(--c-text);border-left:1px solid var(--c-border);box-shadow:-32px 0 64px -28px rgba(0,0,0,.7);animation:cf-sheet-in .28s var(--ease)}
@keyframes cf-sheet-in{from{transform:translateX(24px);opacity:0}to{transform:none;opacity:1}}
@media (max-width:899px){
  .cf-sheet{width:100vw}
}
.cf-sheet-head{display:flex;align-items:center;gap:10px;height:56px;flex-shrink:0;padding:0 10px 0 18px;border-bottom:1px solid var(--c-hair)}
.cf-sheet-title{flex:1;min-width:0;font-size:15px;font-weight:800}
.cf-sheet-tools{display:flex;flex-direction:column;gap:10px;padding:14px 18px 10px;flex-shrink:0}
.cf-sheet-row{display:flex;align-items:center;justify-content:space-between;gap:10px}
.cf-sheet-note{font-size:var(--fs-xs);color:var(--c-muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-sheet-list{flex:1;min-height:0;overflow:auto;padding:0 10px 10px;display:flex;flex-direction:column;gap:4px}
.cf-option{display:grid;grid-template-columns:24px minmax(0,1fr) auto;gap:10px;align-items:center;padding:9px 8px;border-radius:9px;border:1px solid transparent;cursor:pointer}
.cf-option:hover{background:var(--c-card-2)}
.cf-option.is-selected{background:var(--c-accent-tint);border-color:rgba(45,212,191,.28)}
.cf-option-text{min-width:0}
.cf-option-name{display:block;font-size:var(--fs-md);font-weight:700;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-option-meta,.cf-option-id{display:block;font:500 11px/1.5 var(--mono);color:var(--c-muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.cf-sheet-foot{display:flex;align-items:center;gap:10px;height:60px;flex-shrink:0;padding:0 18px;border-top:1px solid var(--c-hair)}
.cf-sheet-count{font-size:var(--fs-sm);color:var(--c-text-2)}
.cf-sheet-count b{color:var(--c-text)}
.cf-bar-meta{flex:0 1 auto;overflow:hidden}
.cf-bar-inner>.cf-btn{flex-shrink:0}
@media (prefers-reduced-motion:reduce){
  .cf *,.cf *::before,.cf *::after{transition-duration:.01ms !important;animation-duration:.01ms !important}
}
```

## Appendix C — markup

Use the app's framework (`class` → `className` in React).
- **Values are sample data** — bind them, and every state class (`is-done`, `is-invalid`, `is-changed`), to real state.
- Counters appear only where a max length exists.

**C.1 Page shell** — header, layout, rail, bar:
```html
<form class="cf" novalidate>
<div class="cf-inner">
<header class="cf-head">
<button type="button" class="cf-back" aria-label="Back to Data Profiles">[icon:chevron-left 17]</button>
<div class="cf-head-text">
<nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">Data Profiles</a> / <span>New</span></nav>
<h1 class="cf-title">Create data profile</h1>
<p class="cf-sub">Define profile details, application scope and access conditions.</p>
</div>
<div class="cf-head-right"><span class="cf-badge cf-badge--new">New</span></div>
</header>
<div class="cf-ledger">
<div class="cf-stack">
<!-- section cards (C.3, C.5) -->
</div>
<aside class="cf-rail" aria-label="Form progress">
<div class="cf-rail-card">
<div class="cf-rail-row"><span class="cf-rail-title">Progress</span><span class="cf-rail-count">5 of 6 required</span></div>
<div class="cf-progress" role="progressbar" aria-label="Required fields complete" aria-valuemin="0" aria-valuemax="6" aria-valuenow="5"><i style="width: 83%"></i></div>
<nav class="cf-toc" aria-label="Sections">
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Profile details</span><span class="cf-toc-meta">Done</span></button>
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Data source</span><span class="cf-toc-meta">Done</span></button>
<button type="button" class="cf-toc-item is-error"><span class="cf-toc-dot">!</span><span class="cf-toc-name">Profile conditions</span><span class="cf-toc-meta">1 issue</span></button>
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
<div class="cf-bar-meta"><span class="cf-saved">[icon:check-circle 15]Draft saved · 2 min ago</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 5 of 6 required</span></div>
<span class="cf-bar-spacer"></span>
<button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Reset</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
</div>
</footer>
</form>
```

**C.2 Field patterns** — error, help, optional, dependent select, switch, open search select:
```html
<!-- help line + error line (error replaces help) -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f1">Profile name<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input is-invalid" id="f1" type="text" required aria-invalid="true" aria-describedby="f1-err">
<span class="cf-error" id="f1-err">[icon:alert-triangle 13]Enter a profile name</span>
</div>
<!-- optional field with help -->
<div class="cf-field cf-span-4">
<label class="cf-label" for="f2">Data source attribute name<span class="cf-opt">Optional</span></label>
<input class="cf-input" id="f2" type="text" aria-describedby="f2-help">
<span class="cf-help" id="f2-help">Shown to users instead of the attribute code.</span>
</div>
<!-- dependent select before its parent is chosen -->
<div class="cf-field cf-span-4">
<label class="cf-label" for="f3">Data source attribute<span class="cf-opt">Optional</span></label>
<span class="cf-select"><select class="cf-input cf-input--mono is-empty" id="f3" disabled><option value="">Select data source first</option></select>[icon:chevron-down 16]</span>
</div>
<!-- switch (Global policy) -->
<div class="cf-field cf-span-12"><label class="cf-switch"><input type="checkbox"><span class="cf-switch-track"></span>Global policy</label></div>
<!-- search select, open -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f4">Application<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-control">[icon:app-window 15]<input id="f4" class="cf-mono" type="text" value="FM" role="combobox" aria-expanded="true" aria-controls="f4-list" aria-autocomplete="list">[icon:chevron-down 16]</div>
<div class="cf-menu" id="f4-list" role="listbox">
<button type="button" class="cf-menu-item is-active" role="option" aria-selected="true">[icon:check 14 bold]<span class="cf-mono">FMCES</span><span class="cf-menu-meta">10 features</span></button>
</div>
</div>
</div>
```

**C.3 Sections 1–2 of Create data profile** — the policy page uses the same patterns:
```html
<section class="cf-sec" id="sec-1" aria-labelledby="sec-1-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-1-t">Profile details</h2><p class="cf-sec-desc">Name the profile and pick the application it applies to.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-6">
<label class="cf-label" for="pr-name">Profile name<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input" id="pr-name" type="text" value="AR-AU Client Coverage" required>
</div>
<div class="cf-field cf-span-6">
<label class="cf-label" for="pr-app">Application<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-control">[icon:app-window 15]<input id="pr-app" class="cf-mono" type="text" value="FMCES" role="combobox" aria-expanded="false" aria-autocomplete="list"><span class="cf-control-sub">10 features</span><button type="button" class="cf-control-btn" aria-label="Clear application">[icon:x 13]</button>[icon:chevron-down 16]</div>
</div>
</div>
<div class="cf-field cf-span-12">
<label class="cf-label" for="pr-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-count">60 / 500</span></label>
<textarea class="cf-input" id="pr-desc" rows="2" required>Client data for clients domiciled in Argentina or Australia.</textarea>
</div>
</div>
</div>
</section>
<section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Data source</h2><p class="cf-sec-desc">Where the data comes from and which attribute the conditions filter on.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-4">
<label class="cf-label" for="pr-src">Data source name<span class="cf-req" aria-hidden="true">*</span></label>
<span class="cf-select"><select class="cf-input cf-input--mono" id="pr-src"><option>CLIENT_MASTER</option></select>[icon:chevron-down 16]</span>
</div>
<div class="cf-field cf-span-4">
<label class="cf-label" for="pr-attr">Data source attribute<span class="cf-opt">Optional</span></label>
<span class="cf-select"><select class="cf-input cf-input--mono" id="pr-attr"><option>CNTRY_DOMICILE</option></select>[icon:chevron-down 16]</span>
</div>
<div class="cf-field cf-span-4">
<label class="cf-label" for="pr-attr-name">Data source attribute name<span class="cf-opt">Optional</span></label>
<input class="cf-input" id="pr-attr-name" type="text" value="Country Domicile">
</div>
</div>
</div>
</section>
```

**C.4 Token field and empty states:**
```html
<div class="cf-field cf-span-12">
<span class="cf-label" id="po-prof-l">Profiles<span class="cf-req" aria-hidden="true">*</span></span>
<div class="cf-tokens" role="group" aria-labelledby="po-prof-l"><span class="cf-chip cf-chip--removable"><span class="cf-chip-name">Version Testing</span><span class="cf-chip-id">#98</span><button type="button" class="cf-chip-x" aria-label="Remove Version Testing">[icon:x 12]</button></span><button type="button" class="cf-tokens-add" aria-haspopup="dialog">[icon:plus 13 bold]Add profile</button></div>
</div>
<!-- Profiles, nothing picked yet -->
<div class="cf-empty">[icon:layers 16]<span class="cf-empty-text">No profiles selected</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm" aria-haspopup="dialog">[icon:plus 14]Add data profile</button></div>
<!-- Conditions, none added yet (replaces .cf-conds) -->
<div class="cf-empty">[icon:list-filter 16]<span class="cf-empty-text">No conditions added yet.</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add condition</button></div>
```

**C.5 Condition builder** — second row shown in its error state:
```html
<section class="cf-sec" id="sec-3" aria-labelledby="sec-3-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-error" aria-hidden="true">!</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-3-t">Profile conditions</h2><p class="cf-sec-desc">Rows are visible only when every condition matches.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-conds">
<div class="cf-cond-head" aria-hidden="true"><span>Key<span class="cf-req">*</span></span><span>Operator<span class="cf-req">*</span></span><span>Values<span class="cf-req">*</span></span><span></span></div>
<div class="cf-cond" role="group" aria-label="Condition 1">
<span class="cf-select"><select class="cf-input cf-input--mono" aria-label="Key, condition 1"><option>Country Domicile</option></select>[icon:chevron-down 16]</span>
<span class="cf-select"><select class="cf-input" aria-label="Operator, condition 1"><option>is any of</option></select>[icon:chevron-down 16]</span>
<div class="cf-tokens"><span class="cf-chip cf-chip--removable cf-chip--mono"><span class="cf-chip-name">AR</span><button type="button" class="cf-chip-x" aria-label="Remove value AR">[icon:x 12]</button></span><span class="cf-chip cf-chip--removable cf-chip--mono"><span class="cf-chip-name">AU</span><button type="button" class="cf-chip-x" aria-label="Remove value AU">[icon:x 12]</button></span><input type="text" placeholder="Add value" aria-label="Values, condition 1"></div>
<button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Remove condition 1">[icon:trash-2 15]</button>
</div>
<div class="cf-cond-join" aria-hidden="true">AND</div>
<div class="cf-cond" role="group" aria-label="Condition 2">
<span class="cf-select"><select class="cf-input cf-input--mono" aria-label="Key, condition 2"><option>Booking_Location</option></select>[icon:chevron-down 16]</span>
<span class="cf-select"><select class="cf-input" aria-label="Operator, condition 2"><option>is not</option></select>[icon:chevron-down 16]</span>
<div class="cf-tokens is-invalid"><input type="text" placeholder="Type a value, press Enter" aria-label="Values, condition 2" aria-invalid="true" aria-describedby="cond-2-err"></div>
<button type="button" class="cf-icon-btn cf-icon-btn--danger" aria-label="Remove condition 2">[icon:trash-2 15]</button>
<span class="cf-error" id="cond-2-err">[icon:alert-triangle 13]Add at least one value</span>
</div>
<div class="cf-cond-foot"><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add condition</button><span>All conditions must match.</span></div>
</div>
</div>
</section>
```

**C.6 Picker sheet:**
```html
<div class="cf-scrim"></div>
<aside class="cf-sheet" role="dialog" aria-modal="true" aria-labelledby="pick-title">
<div class="cf-sheet-head"><h2 class="cf-sheet-title" id="pick-title">Select data profiles</h2><button type="button" class="cf-icon-btn" aria-label="Close">[icon:x 16]</button></div>
<div class="cf-sheet-tools">
<div class="cf-control">[icon:search 15]<input type="search" placeholder="Search by name, ID or owner" aria-label="Search profiles"></div>
<div class="cf-sheet-row"><div class="cf-seg" role="group" aria-label="Show"><button type="button" aria-pressed="true">All 2</button><button type="button" aria-pressed="false">Selected 1</button></div><span class="cf-sheet-note">Profiles in FMCES</span></div>
</div>
<div class="cf-sheet-list">
<label class="cf-option">
<span class="cf-check"><input type="checkbox" aria-label="Select Profile-Ent"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Profile-Ent</span><span class="cf-option-meta">Owner 2023504 · FMCES</span></span>
<span class="cf-option-id">#71</span>
</label>
<label class="cf-option is-selected">
<span class="cf-check"><input type="checkbox" checked aria-label="Select Version Testing"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Version Testing</span><span class="cf-option-meta">Owner g.iam.fmces.002.dev · FMCES</span></span>
<span class="cf-option-id">#98</span>
</label>
</div>
<div class="cf-sheet-foot"><span class="cf-sheet-count"><b>1</b> selected</span><button type="button" class="cf-link">Clear</button><span class="cf-bar-spacer"></span><button type="button" class="cf-btn cf-btn--ghost">Cancel</button><button type="button" class="cf-btn cf-btn--primary">[icon:check 15 bold]Confirm</button></div>
</aside>
```

**C.7 Applications as a multi-select** — only if several are allowed today:
```html
<div class="cf-field cf-span-6">
<label class="cf-label" for="po-apps">Applications<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-tokens"><span class="cf-chip cf-chip--removable cf-chip--mono"><span class="cf-chip-name">FMCES</span><button type="button" class="cf-chip-x" aria-label="Remove FMCES">[icon:x 12]</button></span><span class="cf-chip cf-chip--removable cf-chip--mono"><span class="cf-chip-name">DMF</span><button type="button" class="cf-chip-x" aria-label="Remove DMF">[icon:x 12]</button></span><input id="po-apps" type="text" placeholder="Search applications" role="combobox" aria-expanded="true" aria-controls="po-apps-list" aria-autocomplete="list"></div>
<div class="cf-menu" id="po-apps-list" role="listbox">
<button type="button" class="cf-menu-item is-active" role="option" aria-selected="false"><span class="cf-mono">RATAN_ENTITLEMENT_RULE</span><span class="cf-menu-meta">ITAM 51358</span></button>
</div>
</div>
</div>
```

**C.8 Edit-mode differences:**
```html
<!-- header: meta line under the subtitle + status/version badges -->
<p class="cf-head-meta">ID 71 · Owner 2023504 · Last modified 29 Sep 2026 by g.fmces.001.dev</p>
<div class="cf-head-right"><span class="cf-badge cf-badge--ok">Approved</span><span class="cf-badge cf-badge--version">V1</span></div>
<!-- read-only (locked) search select -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="pr-app">Application</label>
<div class="cf-control is-readonly">[icon:app-window 15]<input id="pr-app" class="cf-mono" type="text" value="FMCES" readonly aria-describedby="pr-app-help"><span class="cf-control-sub">10 features</span>[icon:lock 14]</div>
<span class="cf-help" id="pr-app-help">A profile can’t move to another application.</span>
</div>
<!-- changed field -->
<div class="cf-field cf-span-12">
<label class="cf-label" for="pr-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-edited">Edited</span><span class="cf-count">79 / 500</span></label>
<textarea class="cf-input is-changed" id="pr-desc" rows="2" required>Profile for entitlement testing — clients domiciled in Argentina or Australia.</textarea>
</div>
<!-- changed token field: label gets the Edited tag, box gets is-changed -->
<div class="cf-tokens is-changed" role="group" aria-labelledby="po-prof-l">…chips…</div>
<!-- changed condition row: add is-changed to .cf-cond -->
<div class="cf-cond is-changed" role="group" aria-label="Condition 1">…</div>
<!-- rail item with changes -->
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Profile details</span><span class="cf-toc-meta is-changed">1 change</span></button>
<!-- bar -->
<div class="cf-bar-meta"><span class="is-changed">3 unsaved changes</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 6 of 6 required</span></div>
<button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button>
```

**C.9 Loading placeholders** — one per field, with the same spans, inside each `.cf-sec-body`:
```html
<div class="cf-grid" aria-busy="true" aria-label="Loading">
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-12"><span class="cf-skel cf-skel--label"></span><span class="cf-skel" style="height: 76px"></span></div>
</div>
```

## Appendix D — icon paths

All icons use a 24×24 viewBox, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.75"` (2.25 where marked `bold`), and round line caps and joins.

- `alert-triangle`: `M12 4 2.8 20h18.4zM12 10v4.5M12 17.2h.01`
- `app-window`: `M3 5h18v14H3zM3 9h18M6.5 7h.01M9 7h.01`
- `check`: `M5 12.5l4.5 4.5L19 7.5`
- `check-circle`: `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM8 12l3 3 5-6`
- `chevron-down`: `M6 9l6 6 6-6`
- `chevron-left`: `M15 6l-6 6 6 6`
- `layers`: `M12 3 3 7.5l9 4.5 9-4.5zM3 12l9 4.5 9-4.5M3 16.5 12 21l9-4.5`
- `list-filter`: `M3 6h18M7 12h10M10 18h4`
- `lock`: `M6 11h12v10H6zM8.5 11V7.5a3.5 3.5 0 0 1 7 0V11`
- `plus`: `M12 5v14M5 12h14`
- `rotate-ccw`: `M9 14 4 9l5-5M4 9h10a6 6 0 0 1 0 12h-3`
- `save`: `M5 4h11l3 3v13H5zM8 4v5h7V4M8 20v-6h8v6`
- `search`: `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4`
- `send`: `M4 12 20 4l-6 16-3-7z`
- `trash-2`: `M4 7h16M9 7V4h6v3M6 7l1 13h10l1-13`
- `x`: `M6 6l12 12M18 6 6 18`
