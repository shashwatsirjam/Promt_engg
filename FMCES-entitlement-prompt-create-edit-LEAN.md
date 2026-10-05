# FMCES: rebuild the Entitlement pages (create and edit)

Rebuild the look and layout of 2 pages in FMCES, an internal "Markets Operations" app with a dark navy theme and teal accent:

- **Create entitlement**
- **Edit entitlement**

Match this spec exactly, and do not change how anything behaves. There are no images — everything you need is here. Use the given numbers; do not adjust them by eye.

## 0. Rules

**Keep unchanged:**
- routes;
- API calls, payload shapes and field names;
- the form-state approach;
- validation rules (required, max length, formats, server errors) — restyle them only;
- what Save and Submit do, their toasts, and the navigation after submit;
- permissions;
- any ids, names or data-testids used by tests.

**Do not touch:** the header, the sidebar dock, the list pages, the detail drawer, or any other page. Add no new endpoints and change nothing in the backend.

**[IF DATA EXISTS]** means: build it only if the page already has that data, or an existing API returns it. Otherwise leave it out completely, with no placeholder.

**Remove from these pages:**
- glass or blur panels;
- blue left bars;
- ALL-CAPS section titles;
- floating labels, and placeholders used as labels;
- `**`;
- mixed field heights;
- full-width buttons;
- the old Feature / Action(s) / +Add / Privileged entry bar.

**Sizing:** do not use `zoom`, `transform: scale()`, vw font sizes or root-rem tricks. All sizes come from the CSS variables (px values, switched by media queries).

**CSS** — copy each file verbatim as a plain global stylesheet, even if the app uses CSS modules or Tailwind. Do not translate. Everything is scoped under `.cf`.

| File | When to add it |
|---|---|
| Appendix A → `create-forms.css` | Only if that file does not exist yet |
| Appendix B → `create-forms-extra.css` | Only if that file does not exist yet |
| Appendix C → `create-forms-entitlement.css` | Always. Load it after the other two. |

If the other create pages were done earlier, A and B already exist — keep them.

**Components**
- Reuse these if they exist: FormPage, PageHeader, FormSection, Field, SearchSelect, ProgressRail, ActionBar, TokenField, PickerSheet. Otherwise create them.
- Build a new generic **FeatureRows** component (a row editor).

**Fonts** — add once if missing:
`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

### Step 0 — map the code before changing it

Put this in your first message or the PR description:

1. **Components and fields.** The component for each page. For every field: its state key, whether it is required, max length or format, and control type.
2. **Feature configuration.**
   - The shape of one item in state and in the payload (application? feature, `actions[]` or a single `action`, `privileged`).
   - Where Feature options come from (per application?) and where Action options come from (per feature?).
   - Whether the same feature may appear twice.
   - Whether `privileged` is stored per item.
   - Whether at least one feature is required.
   - What happens to items when the application changes.
3. **Data policies and profiles.**
   - How each picker works today, and where its options come from.
   - Whether the options are filtered by application.
   - Whether the payload uses IDs or objects.
   - Whether each field is optional.
4. **Buttons.** The Save, Submit and Reset handlers.
5. **Edit page.**
   - The load call and which fields are locked.
   - The update handler and payload.
   - Whether there is Save as well as Submit.
   - Whether the record has status, version and last-modified data.
   - Whether editing an approved record creates a new draft version or continues an open draft.

**Order of work:** shared parts → Create entitlement → Edit entitlement → the checks in §7.

## 1. Look

**Density tiers**, picked by viewport — Compact (default) / Comfortable (≥1440×820) / Large (≥1880×980):

| | Compact | Comfortable | Large |
|---|---|---|---|
| Inputs, selects, search selects, buttons | 30px | 34px | 36px |
| Small buttons | 26px | 28px | 28px |
| Chips (incl. action chips) | 22px | 24px | 24px |
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
- JetBrains Mono (`.cf-input--mono` / `.cf-mono`) for IDs, codes, the entitlement name, application names and owners.

**Icons:**
- 24×24 stroke icons, stroke width 1.75 (2.25 where marked `bold`), colour `currentColor`.
- If the app uses lucide, use: ChevronLeft, ChevronDown, X, Search, Plus, Trash2, Check, CheckCircle2, AlertTriangle, RotateCcw, Save, Send, AppWindow, Lock, Layers, FileCheck, Box, Key. Otherwise use the paths in Appendix E.
- In the markup, `[icon:name 16]` means that icon at 16px.

## 2. Page shell (both pages) — markup D.1, D.2

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

## 3. Create entitlement — markup D.3, D.4

- Breadcrumb: `FMCES / Entitlement Management / New`
- Title: `Create entitlement`
- Subtitle: `Fill in the details below and submit for approval.`

**1. Basic information** — "How this entitlement is named and described across FMCES."
- Entitlement name — span 6, mono.
- Display name — span 6. Optional, unless it is required today.
- Description — span 12, textarea, 2 rows.

**2. Data permissions** — "Optional. Limit which rows a user with this entitlement can see." Drop the word "Optional." if either field is required today.
- **Data policies** (span 6) and **Data profiles** (span 6) are token fields:
  - one chip per item: name, mono `#ID`, and ×;
  - a dashed `+ Add policy` / `+ Add profile` button (`aria-haspopup="dialog"`) that opens that field's picker sheet (§5).
- **Nothing attached:** show an empty box instead — `No data policies attached` (file-check icon) or `No data profiles attached` (layers icon), with a small outline `+ Add policy` / `+ Add profile` button.
- **Filtered by application:** if the options depend on the application and none is chosen, disable the add buttons and show the help line `Pick an application first.`
- **Rail meta:** `2 policies`, `2 policies · 1 profile`, or `Optional` when both are empty.

**3. Feature configuration** — "Pick the application, then what this entitlement can do in each feature." Use the feature rows in §4.
- **Application** — span 6, a search select with the `app-window` icon, in mono. **[IF DATA EXISTS]** add a sub-label such as `10 features`.
  - If changing the application clears the feature items today, show the help line `Changing the application clears the feature rows.` and clear them the same way.
- **Rail meta:** `5 features` when valid, or `1 issue` / `N issues`.

## 4. Feature rows — markup D.4, D.5

Rows replace today's Feature / Action(s) / +Add bar. **Each row is exactly one item of today's payload.**

**Header row** (`.cf-perm-head`), shown once, never repeated per row: `FEATURE *` · `ACTIONS *` · `PRIVILEGED`.

**Each row** (`.cf-perm`, `role="group"`, `aria-label="Feature N"`) has four parts:

| Part | Control | Details |
|---|---|---|
| Feature | `.cf-select.cf-perm-feature` | Options: the chosen application's features. If duplicates are not allowed today, disable features already used in other rows. |
| Actions | `.cf-perm-actions` holding a `.cf-tokens` box | Teal chips (`cf-chip cf-chip--accent cf-chip--removable`), then a dashed `+ Action` button (`aria-haspopup="listbox"`). |
| Privileged | `.cf-switch.cf-perm-priv` | Text `No` / `Yes`, with `aria-label="Privileged, <feature>"`. |
| Remove | `.cf-icon-btn--danger.cf-perm-del` | Trash icon. |

**The `+ Action` menu**
- It opens a `.cf-menu` under the box, listing that feature's actions that are not already chosen. It closes when there are none left.
- Keys: ↑ / ↓ move, Enter adds, Esc closes.
- An × on a chip removes that action.
- If today's item holds a **single** action: allow one chip, then hide `+ Action`.
- Changing a row's feature removes its chips only if that action is not valid for the new feature.

**Below the rows** (`.cf-perm-foot`): a small outline `+ Add feature` button, then the muted count `5 features · 7 actions`.
- `+ Add feature` appends an empty row and focuses its Feature. It is disabled until an application is chosen.
- Trash removes the row (no confirm) and focuses the next row's Feature, or the add button.

**No rows:** show the empty box (box icon) `No features added yet. Pick the application, then add what this entitlement can do.` with `+ Add feature`. If at least one feature is required, after a failed submit give it `.is-invalid` and the error `Add at least one feature`.

**Validation** — runs on touched rows, or on all rows after a submit attempt. The first missing item wins:
1. `Pick a feature`
2. `Pick at least one action`
3. `This feature is already in the list` — only if today's code blocks duplicates.

Show one `.cf-error` line; the CSS places it under Actions. The broken control gets `.is-invalid` and `aria-invalid`.

**Section state:** `is-done` when every row is valid and the minimum count is met. `is-error` while any row shows an error.

**Payload:** keep the row order; it is today's item array, unchanged.

**Narrow cards** (< 600px): Feature, Privileged and Remove share the first line; Actions goes underneath. The CSS already does this.

## 5. Picker sheets (policies and profiles) — markup D.6

Use **one PickerSheet component** for both, restyling today's pickers but keeping their data, search, filtering and selection logic.

**Container**
- `.cf-scrim` plus `.cf-sheet` (`role="dialog"`, `aria-modal`, `aria-labelledby`).
- Opens from the right, full height, width `clamp(380px, 32vw, 480px)`; full width below 900px.
- Render it inside the `.cf` form. If dialogs portal out, wrap it in `<div class="cf cf-portal">`.

**Head:** `Select data policies` / `Select data profiles`, and a close × button.

**Tools**
- Search box: `Search by name, ID or owner`. Adjust the placeholder if today's search covers fewer fields.
- Segmented filter `.cf-seg`: `All N` / `Selected N`, using `aria-pressed`.
- **[IF DATA EXISTS]** a note on the right: `Policies in <application>`.

**List:** one `.cf-option` per item. The whole row is a label, so clicking anywhere toggles it. Each row shows:
- a checkbox;
- the name (bold, ellipsis);
- mono meta: `Owner <owner> · <application>` (only the parts that exist);
- mono `#ID`.

Selected rows get `.is-selected`. While loading, show three `.cf-skel` placeholders 44px tall. With no matches, show muted `No policies match "<query>"`.

**Foot:** `N selected` · `Clear` (`.cf-link`) · Cancel (ghost) · Confirm (primary, check icon).
- Confirm applies the selection the same way today's picker does.
- Cancel, ×, Esc and the scrim all close without changes.

**Focus:** goes to the search box on open, is trapped inside the sheet, and returns to the add button on close.

## 6. Edit entitlement — markup D.7, D.8

**Setup**
- Use the same page component with `mode: 'create' | 'edit'`. Both existing routes render it.
- Load with the existing fetch.
- Save with the existing update call and **exactly the payload the old edit page sends**.

**Header**
- Title `Edit entitlement`. Breadcrumb `FMCES / Entitlement Management / <name> / Edit`.
- Subtitle `Update the details below.` If edits need approval, add ` Changes are reviewed before they replace the live version.`
- **[IF DATA EXISTS]** a `.cf-head-meta` line, for example `ID 94 · FMCES · Last modified 30 Sep 2026 by internal-api-call`.
- Badges:
  - the current status — `cf-badge--ok` (Approved), `--draft` or `--pending`, with the app's own text;
  - **[IF DATA EXISTS]** the version, `cf-badge--version` (`V1`).

**Prefill**
- Fill in every field, chip and row.
- Load the application before the feature rows, policies and profiles. Do not run the "clear rows" logic on load.

**Read-only fields**
- Lock exactly the fields today's edit page locks — no more, no fewer.
- A search select becomes `.cf-control.is-readonly` with a trailing `[icon:lock 14]`. A text field gets `readonly`.
- No `*` on locked fields. Add one help line, for example `The name can’t be changed after approval.` or `An entitlement can’t move to another application.`

**Changes** (compared with the loaded values, trimmed)
- A changed field: its label gets `<span class="cf-edited">Edited</span>` after the `*` / Optional and before the counter, and the control gets `.is-changed`. Both are removed when the value goes back.
- Policies and profiles: compare the sets of IDs. A changed box gets `.cf-tokens.is-changed`.
- A new or changed feature row (feature, actions or privileged) gets `.cf-perm.is-changed` (a teal dot). Removed rows also count.
- Change count: each field = 1, each ID set = 1, each added, changed or removed row = 1.

**Action bar**
- Left side: `N unsaved changes` (`<span class="is-changed">`; "1 unsaved change" for one) or `No changes` in muted text, then `* Required`, then any errors. After Save draft: `✓ Draft saved · just now`.
- Buttons:
  - `Discard changes` — ghost, rotate icon. Resets to the loaded values. Disabled when there are no changes.
  - `Save draft` — only if the edit page has Save today.
  - Primary — `Submit for approval` if it starts an approval, otherwise `Save changes`. Disabled while there are no changes.
- Keep any existing unsaved-changes guard; do not add one.

**Rail**
- Section priority: error > changes (teal `1 change`, `.cf-toc-meta.is-changed`) > the create rules.
- **[IF DATA EXISTS: versioning]** for an approved record, the steps are:
  1. Draft V2 — "You can keep editing" (now)
  2. Pending approval
  3. Approved — "Replaces V1"
  4. Active — "V1 stays live until then"

  Use the real version numbers.
- If an open draft exists (the drawer shows "Continue editing"), edit that draft. The badges show `Draft` and its version, and the "now" step is that draft.

**Loading**
- Show the same layout with `.cf-skel` placeholders and the bar buttons disabled.
- On failure: `.cf-alert cf-alert--error` with the existing message, plus a small outline `Retry` that calls the load again.
- Nothing may jump when the data arrives.

## 7. Checks (Chrome at 1366×768, 1440×900, 1920×1080, 1024×768)

1. **Sizes:** controls are 30 / 34 / 36px and body text 12.5 / 13 / 13.5px at the first three sizes. Measure them.
2. **Action bar:** it stays pinned while you scroll at 1366×768, and its buttons align with the right edge of the content. In narrow windows its left text gets an ellipsis and the buttons never overlap.
3. **Layout:** no horizontal page scroll. At 1024 the rail is hidden and the bar shows `· X of Y required`.
4. **Fields:** labels sit above fields, required fields have one red `*`, optional ones have `Optional`. None of the old styles from §0 remain.
5. **Feature rows:**
   - Add a row and the focus moves to its Feature. Remove one and the focus moves on.
   - The `+ Action` menu lists only unchosen actions and works with ↑ ↓ Enter Esc.
   - The chip × and the Privileged switch work.
   - A row with no actions shows `Pick at least one action` under Actions.
   - The empty state shows when there are no rows.
   - The payload is identical to the old page's (Network tab).
6. **Application change:** behaves as today (clears the rows if it did before).
7. **Pickers:**
   - Both open from the right, full height, with the scrim.
   - Search and the Selected filter work.
   - Confirm fills the chips; Cancel, ×, Esc and the scrim change nothing.
   - Focus returns to the add button.
   - The chip × removes an item.
8. **Edit:**
   - Stored values show, including chips and rows.
   - Placeholders show while loading, with no jump.
   - Edited tags, row dots and the change count update, and clear on revert.
   - Discard restores everything.
   - The primary button is disabled with no changes.
   - Locked fields are the same as before.
   - The payload matches the old edit page's.
9. **Buttons and tests:**
   - Save, Submit and Reset behave as before (toasts, navigation).
   - No console errors. Tests pass; update snapshots only for markup.
   - Keep every `aria-*`, role and label from the markup, and never remove focus outlines.

## Appendix A — `create-forms.css` (create only if it doesn't exist)

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

## Appendix B — `create-forms-extra.css` (create only if it doesn't exist; load after A)

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

## Appendix C — `create-forms-entitlement.css` (always; load after A and B)

```css
.cf-chip--accent{background:var(--c-accent-tint);border-color:rgba(45,212,191,.32);color:var(--c-accent-strong)}
.cf-chip--accent .cf-chip-x{color:var(--c-accent-strong)}
.cf-chip--accent .cf-chip-x:hover{background:var(--c-err-tint);color:var(--c-err)}
.cf-perms{display:flex;flex-direction:column;gap:8px}
.cf-perm-head,.cf-perm{display:grid;grid-template-columns:minmax(150px,210px) minmax(0,1fr) 104px 28px;column-gap:8px;align-items:start}
.cf-perm-head{padding:0 2px;font-size:var(--fs-xs);font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--c-muted)}
.cf-perm-head .cf-req{margin-left:2px}
.cf-perm{position:relative;grid-template-areas:"f a p d" ". e . ."}
.cf-perm-feature{grid-area:f;min-width:0}
.cf-perm-actions{grid-area:a;position:relative;min-width:0}
.cf-perm-priv{grid-area:p;height:var(--h-input)}
.cf-perm-del{grid-area:d;margin-top:calc((var(--h-input) - 28px)/2)}
.cf-perm>.cf-error{grid-area:e;margin-top:5px}
.cf-perm-actions>.cf-menu{right:auto;min-width:220px}
.cf-perm.is-changed::before{content:"";position:absolute;left:-11px;top:calc(var(--h-input)/2 - 3px);width:6px;height:6px;border-radius:50%;background:var(--c-accent)}
.cf-perm-foot{display:flex;align-items:center;flex-wrap:wrap;gap:10px;margin-top:4px;font-size:var(--fs-xs);color:var(--c-muted)}
@container (max-width:600px){
  .cf-perm-head{display:none}
  .cf-perm{grid-template-columns:minmax(0,1fr) auto 28px;grid-template-areas:"f p d" "a a a" "e e e";row-gap:6px}
}
```

## Appendix D — markup

Use the app's framework (`class` → `className` in React).
- **The values are sample data.** Bind them, and every state class (`is-done`, `is-invalid`, `is-changed`), to real state.
- Counters only appear where a max length exists.

**D.1 Page shell** — header, layout, rail and bar:
```html
<form class="cf" novalidate>
<div class="cf-inner">
<header class="cf-head">
<button type="button" class="cf-back" aria-label="Back to Entitlement Management">[icon:chevron-left 17]</button>
<div class="cf-head-text">
<nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">Entitlement Management</a> / <span>New</span></nav>
<h1 class="cf-title">Create entitlement</h1>
<p class="cf-sub">Fill in the details below and submit for approval.</p>
</div>
<div class="cf-head-right"><span class="cf-badge cf-badge--new">New</span></div>
</header>
<div class="cf-ledger">
<div class="cf-stack">
<!-- section cards (D.3, D.4) -->
</div>
<aside class="cf-rail" aria-label="Form progress">
<div class="cf-rail-card">
<div class="cf-rail-row"><span class="cf-rail-title">Progress</span><span class="cf-rail-count">3 of 4 required</span></div>
<div class="cf-progress" role="progressbar" aria-label="Required fields complete" aria-valuemin="0" aria-valuemax="4" aria-valuenow="3"><i style="width: 75%"></i></div>
<nav class="cf-toc" aria-label="Sections">
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Basic information</span><span class="cf-toc-meta">Done</span></button>
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Data permissions</span><span class="cf-toc-meta">2 policies</span></button>
<button type="button" class="cf-toc-item is-error"><span class="cf-toc-dot">!</span><span class="cf-toc-name">Feature configuration</span><span class="cf-toc-meta">1 issue</span></button>
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
<div class="cf-bar-meta"><span class="cf-saved">[icon:check-circle 15]Draft saved · 2 min ago</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 3 of 4 required</span></div>
<span class="cf-bar-spacer"></span>
<button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Reset</button><button type="button" class="cf-btn cf-btn--secondary">[icon:save 15]Save draft</button><button type="submit" class="cf-btn cf-btn--primary">[icon:send 15 bold]Submit for approval</button>
</div>
</footer>
</form>
```

**D.2 Field patterns** — error, help line, open search select:
```html
<!-- error line (replaces the help line) -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f1">Entitlement name<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input cf-input--mono is-invalid" id="f1" type="text" required aria-invalid="true" aria-describedby="f1-err">
<span class="cf-error" id="f1-err">[icon:alert-triangle 13]Enter an entitlement name</span>
</div>
<!-- optional field with a help line -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f2">Display name<span class="cf-opt">Optional</span></label>
<input class="cf-input" id="f2" type="text" aria-describedby="f2-help">
<span class="cf-help" id="f2-help">Shown to users instead of the name.</span>
</div>
<!-- search select, open -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f3">Application<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-control">[icon:app-window 15]<input id="f3" class="cf-mono" type="text" value="FM" role="combobox" aria-expanded="true" aria-controls="f3-list" aria-autocomplete="list">[icon:chevron-down 16]</div>
<div class="cf-menu" id="f3-list" role="listbox">
<button type="button" class="cf-menu-item is-active" role="option" aria-selected="true">[icon:check 14 bold]<span class="cf-mono">FMCES</span><span class="cf-menu-meta">10 features</span></button>
</div>
</div>
<span class="cf-help">Changing the application clears the feature rows.</span>
</div>
```

**D.3 Sections 1–2** — Basic information and Data permissions:
```html
<section class="cf-sec" id="sec-1" aria-labelledby="sec-1-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-1-t">Basic information</h2><p class="cf-sec-desc">How this entitlement is named and described across FMCES.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-6">
<label class="cf-label" for="en-name">Entitlement name<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input cf-input--mono" id="en-name" type="text" value="FMCES_POLICY_APPROVER" required>
</div>
<div class="cf-field cf-span-6">
<label class="cf-label" for="en-display">Display name<span class="cf-opt">Optional</span></label>
<input class="cf-input" id="en-display" type="text" value="CES Policy Approver">
</div>
<div class="cf-field cf-span-12">
<label class="cf-label" for="en-desc">Description<span class="cf-req" aria-hidden="true">*</span><span class="cf-count">137 / 500</span></label>
<textarea class="cf-input" id="en-desc" rows="2" required>Reviews and approves data policy and data profile changes raised by makers in FMCES. Read-only access to entitlements and reports.</textarea>
</div>
</div>
</div>
</section>
<section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Data permissions</h2><p class="cf-sec-desc">Optional. Limit which rows a user with this entitlement can see.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-6">
<span class="cf-label" id="en-pol-l">Data policies<span class="cf-opt">Optional</span></span>
<div class="cf-tokens" role="group" aria-labelledby="en-pol-l"><span class="cf-chip cf-chip--removable"><span class="cf-chip-name">Policy Versioning Testing</span><span class="cf-chip-id">#164</span><button type="button" class="cf-chip-x" aria-label="Remove Policy Versioning Testing">[icon:x 12]</button></span><span class="cf-chip cf-chip--removable"><span class="cf-chip-name">Entitlement Policy</span><span class="cf-chip-id">#143</span><button type="button" class="cf-chip-x" aria-label="Remove Entitlement Policy">[icon:x 12]</button></span><button type="button" class="cf-tokens-add" aria-haspopup="dialog">[icon:plus 13 bold]Add policy</button></div>
</div>
<div class="cf-field cf-span-6">
<span class="cf-label" id="en-prof-l">Data profiles<span class="cf-opt">Optional</span></span>
<div class="cf-empty" role="group" aria-labelledby="en-prof-l">[icon:layers 16]<span class="cf-empty-text">No data profiles attached</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm" aria-haspopup="dialog">[icon:plus 14]Add profile</button></div>
</div>
</div>
</div>
</section>
```

**D.4 Section 3** — Feature configuration; the third row shows the error state:
```html
<section class="cf-sec" id="sec-3" aria-labelledby="sec-3-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-error" aria-hidden="true">!</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-3-t">Feature configuration</h2><p class="cf-sec-desc">Pick the application, then what this entitlement can do in each feature.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-6">
<label class="cf-label" for="en-app">Application<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-control">[icon:app-window 15]<input id="en-app" class="cf-mono" type="text" value="FMCES" role="combobox" aria-expanded="false" aria-autocomplete="list"><span class="cf-control-sub">10 features</span><button type="button" class="cf-control-btn" aria-label="Clear application">[icon:x 13]</button>[icon:chevron-down 16]</div>
</div>
</div>
<div class="cf-field cf-span-12">
<div class="cf-perms">
<div class="cf-perm-head" aria-hidden="true"><span>Feature<span class="cf-req">*</span></span><span>Actions<span class="cf-req">*</span></span><span>Privileged</span><span></span></div>
<div class="cf-perm" role="group" aria-label="Feature 1">
<span class="cf-select cf-perm-feature"><select class="cf-input" aria-label="Feature, row 1"><option>Data Policy</option></select>[icon:chevron-down 16]</span>
<div class="cf-perm-actions"><div class="cf-tokens"><span class="cf-chip cf-chip--accent cf-chip--removable"><span class="cf-chip-name">View</span><button type="button" class="cf-chip-x" aria-label="Remove View from Data Policy">[icon:x 12]</button></span><span class="cf-chip cf-chip--accent cf-chip--removable"><span class="cf-chip-name">Approve</span><button type="button" class="cf-chip-x" aria-label="Remove Approve from Data Policy">[icon:x 12]</button></span><button type="button" class="cf-tokens-add" aria-haspopup="listbox" aria-expanded="false">[icon:plus 13 bold]Action</button></div></div>
<label class="cf-switch cf-perm-priv"><input type="checkbox" role="switch" aria-label="Privileged, Data Policy"><span class="cf-switch-track"></span>No</label>
<button type="button" class="cf-icon-btn cf-icon-btn--danger cf-perm-del" aria-label="Remove Data Policy">[icon:trash-2 15]</button>
</div>
<div class="cf-perm" role="group" aria-label="Feature 2">
<span class="cf-select cf-perm-feature"><select class="cf-input" aria-label="Feature, row 2"><option>Reports</option></select>[icon:chevron-down 16]</span>
<div class="cf-perm-actions"><div class="cf-tokens"><span class="cf-chip cf-chip--accent cf-chip--removable"><span class="cf-chip-name">View</span><button type="button" class="cf-chip-x" aria-label="Remove View from Reports">[icon:x 12]</button></span><span class="cf-chip cf-chip--accent cf-chip--removable"><span class="cf-chip-name">Download</span><button type="button" class="cf-chip-x" aria-label="Remove Download from Reports">[icon:x 12]</button></span><button type="button" class="cf-tokens-add" aria-haspopup="listbox" aria-expanded="false">[icon:plus 13 bold]Action</button></div></div>
<label class="cf-switch cf-perm-priv"><input type="checkbox" role="switch" aria-label="Privileged, Reports"><span class="cf-switch-track"></span>No</label>
<button type="button" class="cf-icon-btn cf-icon-btn--danger cf-perm-del" aria-label="Remove Reports">[icon:trash-2 15]</button>
</div>
<div class="cf-perm" role="group" aria-label="Feature 3">
<span class="cf-select cf-perm-feature"><select class="cf-input" aria-label="Feature, row 3"><option>IAM Reports</option></select>[icon:chevron-down 16]</span>
<div class="cf-perm-actions"><div class="cf-tokens is-invalid"><button type="button" class="cf-tokens-add" aria-haspopup="listbox" aria-expanded="false" aria-invalid="true" aria-describedby="perm-3-err">[icon:plus 13 bold]Action</button></div></div>
<label class="cf-switch cf-perm-priv"><input type="checkbox" role="switch" aria-label="Privileged, IAM Reports"><span class="cf-switch-track"></span>No</label>
<button type="button" class="cf-icon-btn cf-icon-btn--danger cf-perm-del" aria-label="Remove IAM Reports">[icon:trash-2 15]</button>
<span class="cf-error" id="perm-3-err">[icon:alert-triangle 13]Pick at least one action</span>
</div>
<div class="cf-perm-foot"><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add feature</button><span>3 features · 4 actions</span></div>
</div>
</div>
</div>
</div>
</section>
```

**D.5 Row details and empty states:**
```html
<!-- Actions cell with its add menu open (only actions not already chosen for this feature) -->
<div class="cf-perm-actions"><div class="cf-tokens"><span class="cf-chip cf-chip--accent cf-chip--removable"><span class="cf-chip-name">View</span><button type="button" class="cf-chip-x" aria-label="Remove View from Reports">[icon:x 12]</button></span><button type="button" class="cf-tokens-add" aria-haspopup="listbox" aria-expanded="true" aria-controls="perm-2-menu">[icon:plus 13 bold]Action</button></div>
<div class="cf-menu" id="perm-2-menu" role="listbox" aria-label="Actions for Reports"><button type="button" class="cf-menu-item is-active" role="option" aria-selected="false">Download</button></div></div>
<!-- privileged switch when on -->
<label class="cf-switch cf-perm-priv"><input type="checkbox" role="switch" checked aria-label="Privileged, Data Policy"><span class="cf-switch-track"></span>Yes</label>
<!-- no feature rows yet (replaces the header row and rows; the add button stays disabled until an application is chosen) -->
<div class="cf-empty">[icon:box 16]<span class="cf-empty-text">No features added yet. Pick the application, then add what this entitlement can do.</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add feature</button></div>
<!-- no data policies attached -->
<div class="cf-empty" role="group" aria-labelledby="en-pol-l">[icon:file-check 16]<span class="cf-empty-text">No data policies attached</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm" aria-haspopup="dialog">[icon:plus 14]Add policy</button></div>
```

**D.6 Picker sheet** (policies; profiles is the same):
```html
<div class="cf-scrim"></div>
<aside class="cf-sheet" role="dialog" aria-modal="true" aria-labelledby="pick-title">
<div class="cf-sheet-head"><h2 class="cf-sheet-title" id="pick-title">Select data policies</h2><button type="button" class="cf-icon-btn" aria-label="Close">[icon:x 16]</button></div>
<div class="cf-sheet-tools">
<div class="cf-control">[icon:search 15]<input type="search" placeholder="Search by name, ID or owner" aria-label="Search policies"></div>
<div class="cf-sheet-row"><div class="cf-seg" role="group" aria-label="Show"><button type="button" aria-pressed="true">All 4</button><button type="button" aria-pressed="false">Selected 2</button></div><span class="cf-sheet-note">Policies in FMCES</span></div>
</div>
<div class="cf-sheet-list">
<label class="cf-option is-selected">
<span class="cf-check"><input type="checkbox" checked aria-label="Select Entitlement Policy"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Entitlement Policy</span><span class="cf-option-meta">Owner 1466717 · FMCES</span></span>
<span class="cf-option-id">#143</span>
</label>
<label class="cf-option">
<span class="cf-check"><input type="checkbox" aria-label="Select Policy-Ent"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Policy-Ent</span><span class="cf-option-meta">Owner 1466717 · FMCES</span></span>
<span class="cf-option-id">#150</span>
</label>
<label class="cf-option">
<span class="cf-check"><input type="checkbox" aria-label="Select Policy Testing"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Policy Testing</span><span class="cf-option-meta">Owner g.iam.fmces.002.dev · FMCES</span></span>
<span class="cf-option-id">#163</span>
</label>
<label class="cf-option is-selected">
<span class="cf-check"><input type="checkbox" checked aria-label="Select Policy Versioning Testing"><span class="cf-check-box">[icon:check 12 bold]</span></span>
<span class="cf-option-text"><span class="cf-option-name">Policy Versioning Testing</span><span class="cf-option-meta">Owner g.iam.fmces.002.dev · FMCES</span></span>
<span class="cf-option-id">#164</span>
</label>
</div>
<div class="cf-sheet-foot"><span class="cf-sheet-count"><b>2</b> selected</span><button type="button" class="cf-link">Clear</button><span class="cf-bar-spacer"></span><button type="button" class="cf-btn cf-btn--ghost">Cancel</button><button type="button" class="cf-btn cf-btn--primary">[icon:check 15 bold]Confirm</button></div>
</aside>
```

**D.7 Edit-mode differences:**
```html
<!-- header: meta line under the subtitle + status/version badges -->
<p class="cf-head-meta">ID 94 · FMCES · Last modified 30 Sep 2026 by internal-api-call</p>
<div class="cf-head-right"><span class="cf-badge cf-badge--ok">Approved</span><span class="cf-badge cf-badge--version">V1</span></div>
<!-- locked field -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="en-name">Entitlement name</label>
<div class="cf-control is-readonly">[icon:key 15]<input id="en-name" class="cf-mono" type="text" value="IAM Admin" readonly aria-describedby="en-name-help">[icon:lock 14]</div>
<span class="cf-help" id="en-name-help">The name can’t be changed after approval.</span>
</div>
<!-- changed token field -->
<div class="cf-field cf-span-6">
<span class="cf-label" id="en-pol-l">Data policies<span class="cf-opt">Optional</span><span class="cf-edited">Edited</span></span>
<div class="cf-tokens is-changed" role="group" aria-labelledby="en-pol-l">…chips…</div>
</div>
<!-- changed feature row: add is-changed to .cf-perm -->
<div class="cf-perm is-changed" role="group" aria-label="Feature 2">…</div>
<!-- rail item with changes -->
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Feature configuration</span><span class="cf-toc-meta is-changed">1 change</span></button>
<!-- bar -->
<div class="cf-bar-meta"><span class="is-changed">2 unsaved changes</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 4 of 4 required</span></div>
<button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button>
```

**D.8 Loading placeholders** — one per field with the same spans, inside each `.cf-sec-body`:
```html
<div class="cf-grid" aria-busy="true" aria-label="Loading">
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-12"><span class="cf-skel cf-skel--label"></span><span class="cf-skel" style="height: 76px"></span></div>
</div>
```

## Appendix E — icon paths

All icons: 24×24 viewBox, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.75"` (2.25 where marked `bold`), round line caps and joins.

- `alert-triangle`: `M12 4 2.8 20h18.4zM12 10v4.5M12 17.2h.01`
- `app-window`: `M3 5h18v14H3zM3 9h18M6.5 7h.01M9 7h.01`
- `box`: `M12 3 4 7v10l8 4 8-4V7zM4 7l8 4 8-4M12 11v10`
- `check`: `M5 12.5l4.5 4.5L19 7.5`
- `check-circle`: `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM8 12l3 3 5-6`
- `chevron-down`: `M6 9l6 6 6-6`
- `chevron-left`: `M15 6l-6 6 6 6`
- `file-check`: `M6 3h9l4 4v14H6zM14 3v5h5M9 14l2 2 4-4`
- `key`: `M15 4a5 5 0 1 0 0 10 5 5 0 0 0 0-10zM11.5 12.5 3 21M6 18l2.5 2.5M8.5 15.5 11 18`
- `layers`: `M12 3 3 7.5l9 4.5 9-4.5zM3 12l9 4.5 9-4.5M3 16.5 12 21l9-4.5`
- `lock`: `M6 11h12v10H6zM8.5 11V7.5a3.5 3.5 0 0 1 7 0V11`
- `plus`: `M12 5v14M5 12h14`
- `rotate-ccw`: `M9 14 4 9l5-5M4 9h10a6 6 0 0 1 0 12h-3`
- `save`: `M5 4h11l3 3v13H5zM8 4v5h7V4M8 20v-6h8v6`
- `search`: `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4`
- `send`: `M4 12 20 4l-6 16-3-7z`
- `trash-2`: `M4 7h16M9 7V4h6v3M6 7l1 13h10l1-13`
- `x`: `M6 6l12 12M18 6 6 18`
