# FMCES: rebuild the User Entitlement (account) pages — create and edit

Rebuild the look and layout of 2 pages in FMCES, an internal "Markets Operations" app with a dark navy theme and teal accent:

- **Create account** (today's "Create Account" under User Entitlements)
- **Edit account** (the edit page for a user entitlement)

Match this spec exactly, and do not change behaviour. There are no images; everything you need is here. Use the given numbers — never adjust them by eye.

## 0. Rules

**Keep unchanged:**
- Routes.
- API calls, payload shapes and field names.
- The form-state approach.
- Validation rules (required, formats, server errors) — restyle them only.
- What Create and Save do, the toasts, and navigation after submit.
- Permissions.
- Any ids, names or data-testids that tests use.

**Do not touch:** the header, the sidebar dock, the list pages, the detail drawer, or any other page. Add no new endpoints and change no backend code.

**[IF DATA EXISTS]** means: build it only if the page already has that data, or an existing API returns it. Otherwise leave it out completely, with no placeholder.

**Remove from these pages:**
- Glass and blur panels.
- Blue left bars.
- ALL-CAPS section titles.
- Floating labels, and placeholders used as labels.
- `**`.
- Mixed field heights.
- Full-width buttons.
- Large empty panels.

**Sizing:** no `zoom`, no `transform: scale()`, no vw-based font sizes, no root-rem tricks. All sizes come from the CSS variables (px values, switched by media queries).

**CSS:** copy each file verbatim as a plain global stylesheet, even if the app uses CSS modules or Tailwind. Do not translate. Everything is scoped under `.cf`.

| File | Contents | When |
|---|---|---|
| `create-forms.css` | Appendix A | Only if the file does not exist yet |
| `create-forms-extra.css` | Appendix B | Only if the file does not exist yet |
| `create-forms-account.css` | Appendix C | Always; load it after the other two |

If the other create pages were done earlier, A and B already exist — keep them as they are.

**Components:** reuse these if they exist — FormPage, PageHeader, FormSection, Field, SearchSelect, ProgressRail, ActionBar. Build a new generic **AssignmentRows** component (application → entitlement rows).

**Fonts:** add this once if it is missing:
`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

### Step 0 — map the current code first

Put this in your first message or the PR description:

1. **Fields.** The component for each page. For every field: its state key, whether it is required, its format rule and its control type. Today's create fields are Bank ID, Account Owner, Application ITAM, Application, Entitlement and Account Type.
2. **Bank ID.** Is the user looked up (validated) today? When does the lookup run, and what does it return (name, status, existing entitlements)?
3. **Account owner and type.** Where the Account owner options come from, and what the Account type options are.
4. **Application ITAM.** Is it sent in the payload, or is it only a filter? Can applications be loaded without picking an ITAM first?
5. **Entitlement.** Where its options come from (per application). Can one create request assign **one** or **several** application + entitlement pairs?
6. **Create.** The Cancel and Create handlers, and whether creating goes to an approval step.
7. **Edit page.**
   - Its route and load call.
   - Whether it edits one account (one entitlement) or all of a user's entitlements.
   - Which fields are locked.
   - The update handler and payload.
   - What status data exists.
   - Whether saving goes to approval.

**Order of work:** shared parts → Create account → Edit account → the checks in §6.

## 1. Look

**Density tiers**, picked by viewport — Compact (default) / Comfortable (≥1440×820) / Large (≥1880×980):

| | Compact | Comfortable | Large |
|---|---|---|---|
| Inputs, selects, search selects, buttons | 30px | 34px | 36px |
| Small buttons | 26px | 28px | 28px |
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
- JetBrains Mono (`.cf-input--mono` / `.cf-mono`) for Bank IDs, owners, ITAM IDs, application names and entitlement IDs.

**Icons:**
- 24×24 stroke icons, stroke width 1.75 (2.25 where marked `bold`), colour `currentColor`.
- If the app uses lucide, use: ChevronLeft, ChevronDown, X, Plus, Trash2, Check, AlertTriangle, RotateCcw, AppWindow, Lock, Key, User, Server. Otherwise use the paths in Appendix E.
- In the markup, `[icon:name 16]` means that icon at 16px.

## 2. Page shell (both pages) — markup D.1, D.2

**Mounting**
- Put the page inside the existing content area (right of the dock, below the header).
- The root is `<form class="cf" novalidate>`. If the page already has a `<form>`, add the `cf` class to it instead.
- The content area must be the scroll container, with a fixed height. `.cf` has `min-height: 100%`; without a fixed-height parent, use `min-height: calc(100vh - <header height in px>)`.
- `.cf-bar` is `position: sticky; bottom: 0`. No element between it and the scroll container may have `overflow: hidden` or `overflow: auto`.

**Header**
- Back button (to the list page), then a mono breadcrumb (every part except the last is a link), the title (sentence case) and a one-line subtitle.
- Right side: nothing on create (unless today's page has drafts: then `New`, which becomes `Draft` after the first Save). On edit, it shows the status badge (§5).

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
- **Card 2:** only when the page says so (§3, §5).

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
  - `* Required`;
  - after a failed attempt: `N fields need attention` (`.is-error`);
  - only if a Save exists: after a successful save, `✓ Draft saved · 2 min ago` (relative time, refreshed every 60s).
- **Right:** the buttons given in §3 and §5. The primary button is always last.
- While a request runs, its button gets `.is-loading` and the others are disabled.

**Validation**
- Never show errors on untouched fields. Validate on blur.
- On Create or Save: validate everything, show the count in the bar, then scroll to and focus the first invalid field.
- Server errors for a field go under that field. Other server errors go in `.cf-alert cf-alert--error` at the top of the first section.

## 3. Create account — markup D.3, D.4

- Breadcrumb: `FMCES / User Entitlements / New`
- Title: `Create account`
- Subtitle: `Provision a new entitlement account and assign roles.`

**1. Account** — "Who the account is for and who owns it."

- **Bank ID** — span 6, mono.
  - **[IF DATA EXISTS]** If today's page looks up the user, show the result under the field with the same trigger as today:
    - While it runs: the help line `Looking up…`.
    - Found: a `.cf-lookup` strip with only the facts returned, e.g. `Found <b>8216018</b> · Active user · 1 existing entitlement`.
    - Not found: `.cf-lookup.is-error` with `No user found with Bank ID <b>…</b>`.
- **Account owner** — span 6. Use today's control: a user search select (user icon, mono, sub-label `Bank ID`) or a mono text field.
- **Account type** — span 6. A select with today's options.

**2. Access** — "The application and entitlement this account gets." In multi mode, say "applications and entitlements".
- This section holds the assignment rows (§4).
- Rail meta: `1 entitlement` / `2 entitlements`, or the issue count.

**Rail:** only the Progress card.
- Add the `AFTER YOU SUBMIT` card only if creating an account goes to approval today. It has 4 steps: Draft "You can keep editing" (now) · Pending approval "A checker reviews it" · Approved · Active.

**Bar:**
- Left: `* Required`, then errors after a failed attempt.
- Right: `Cancel` (ghost; same as the back button) · primary `Create account` (check icon), which runs today's Create.
- If today's page also has a Save, add `Save draft` (outline) before the primary.

## 4. Assignment rows — markup D.4, D.5

**Header row** (`.cf-assign-head`), shown once: `APPLICATION *` · `ENTITLEMENT *`.

**Each row** (`.cf-assign`, `role="group"`, `aria-label="Entitlement N"`) has three parts:

| Part | What it is |
|---|---|
| Application | Search select `.cf-assign-app`: app-window icon, mono. It searches **by name or ITAM ID**. **[IF DATA EXISTS]** sub-label `ITAM 51358`. |
| Entitlement | Search select `.cf-assign-ent`: key icon. Its options are the chosen application's entitlements. Disabled, with placeholder `Select application first`, until an application is chosen. **[IF DATA EXISTS]** sub-label `#100`. |
| Remove | Trash button `.cf-assign-del` — multi mode only. |

Every control has an `aria-label`, for example `Entitlement, row 2`.

**ITAM**
- If the payload includes Application ITAM, fill it in from the chosen application's ITAM.
- If applications can only be loaded per ITAM, use the ITAM-column variant instead (D.5):
  - add `.cf-assigns--itam` on the container;
  - put an `ITAM` head cell first;
  - put a `.cf-assign-itam` search select first in each row;
  - keep Application disabled until an ITAM is chosen, with placeholder `Select ITAM first`.
- When a parent changes, clear its children (ITAM → application → entitlement), as today.

**Single mode** — today's create assigns one entitlement:
- Exactly one row.
- No trash button, no add button.

**Multi mode** — several are allowed:
- Under the rows: a small outline `+ Add entitlement` button and the count `2 entitlements`.
- Add appends a row and focuses its Application.
- Trash removes the row with no confirm, and focuses the next row's Application (or the add button).
- With no rows: the `.cf-empty` box (key icon) `No entitlements added yet.` with `+ Add entitlement`.

**Validation** — on touched rows, or on all rows after a submit attempt. The first problem found wins:

| Order | Message | Shown under |
|---|---|---|
| 1 | `Pick an application` | Application (`.cf-error.cf-assign-err-app`) |
| 2 | `Pick an entitlement` | Entitlement (`.cf-error.cf-assign-err-ent`) |
| 3 | `Already in the list` (the same pair twice) | Entitlement |
| 4 | **[IF DATA EXISTS]** `This user already has this entitlement` | Entitlement |

The invalid control gets `.is-invalid`, `aria-invalid` and `aria-describedby`.

**Payload:** today's shape, with rows in order.

**Narrow cards** (< 600px): the controls stack one per line, and each error sits directly under its control. This is already in the CSS.

## 5. Edit account — markup D.6, D.7

**Setup**
- Use the same component with `mode: 'create' | 'edit'`; both existing routes render it.
- Load the record with the existing fetch.
- Save with the existing update call and **exactly the payload the old edit page sends**.

**Header**
- Title: `Edit account`, or the name today's edit page uses.
- Breadcrumb: `FMCES / User Entitlements / <Bank ID> / Edit`.
- Subtitle: `Update the details below.` If saving goes to approval, add ` Changes are reviewed before they replace the live version.`
- **[IF DATA EXISTS]** a `.cf-head-meta` line, e.g. `Bank ID 8216018 · User · FMCES · 1 entitlement`.
- One status badge, using the app's own text:
  - Active → `cf-badge--ok`
  - Pending → `cf-badge--pending`
  - Draft → `cf-badge--draft`
  - Inactive or any other status → `cf-badge--version` (neutral)

**Prefill**
- Fill in every field and row.
- Load in order: ITAM → application → entitlement. Do not run the "clear children" logic during the initial load.

**Rows**
- If today's edit page edits all of a user's entitlements, use multi mode with all of them.
- Otherwise show the single account's row, exactly as on create.

**Read-only fields**
- Lock **exactly** the fields today's edit page locks.
- A locked field is a `.cf-control.is-readonly` with a trailing `[icon:lock 14]` (a text field just gets `readonly`).
- Locked fields have no `*`, and one help line, e.g. `The Bank ID can’t be changed.`

**Changes** (compared with the loaded values, trimmed)
- A changed field: the label gets `<span class="cf-edited">Edited</span>` and the control gets `.is-changed`.
- A new or changed row gets `.cf-assign.is-changed` (a teal dot). Removed rows also count.
- Change count: each field = 1, and each added, changed or removed row = 1.

**Bar**
- Left: `N unsaved changes` (`<span class="is-changed">`, or "1 unsaved change"), or `No changes` in muted text. Then `* Required`, then any errors.
- Right:
  - `Discard changes` (ghost, rotate icon) — goes back to the loaded values. Disabled when there are no changes.
  - Primary `Save changes` (check icon), or `Submit for approval` (send icon) if saving goes to approval. Disabled while there are no changes.
- Keep any existing unsaved-changes guard; do not add a new one.

**Rail**
- Each section's meta: an error wins over changes (teal `1 change`, `.cf-toc-meta.is-changed`), and changes win over the create rules.
- No lifecycle card, unless saving goes to approval.

**Loading**
- While loading: the same layout with `.cf-skel` placeholders (D.7), and the bar buttons disabled.
- On failure: `.cf-alert cf-alert--error` with the existing message, plus a small outline `Retry` button that calls the load again.
- Nothing may jump when the data arrives.

## 6. Checks

Test in Chrome at 1366×768, 1440×900, 1920×1080 and 1024×768.

1. **Sizes:** controls are 30 / 34 / 36px and text is 12.5 / 13 / 13.5px at the first three sizes. Measure them.
2. **Bar:** it stays pinned at the bottom while scrolling, and its buttons align with the content's right edge. On narrow windows the left text gets an ellipsis and the buttons never overlap.
3. **Layout:** no horizontal page scroll. At 1024 the rail is hidden and the bar shows `· X of Y required`.
4. **Fields:** labels sit above fields, with one red `*` on required fields and `Optional` on optional ones. None of the old styles from §0 remain.
5. **Bank ID lookup** (if it exists): the found and not-found strips show as described.
6. **Rows:**
   - The chain ITAM → application → entitlement disables and clears as today.
   - Search by ITAM ID finds the application.
   - Errors appear under the right column.
   - In multi mode, add and remove work and focus moves correctly.
7. **Payload:** Create and Save send payloads identical to the old pages, including Application ITAM (check the Network tab).
8. **Edit:**
   - Stored values show, with placeholders while loading and no jump.
   - The Edited tag, row dots and change count update, and clear when a change is reverted.
   - Discard restores everything.
   - The primary button is disabled with no changes.
   - Locked fields match the old page.
9. **Final:**
   - Cancel, Create and Save behave as before (toasts and navigation).
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

## Appendix B — `create-forms-extra.css` (create only if it doesn't exist; load it after A)

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

## Appendix C — `create-forms-account.css` (always; load it after A and B)

```css
.cf-lookup{display:flex;align-items:center;gap:10px;min-height:var(--h-input);padding:6px 10px;border-radius:9px;background:var(--c-accent-tint);font-size:var(--fs-sm);color:#BFF3EA}
.cf-lookup.is-error{background:var(--c-err-tint);color:#F8CACA}
.cf-lookup-av{width:26px;height:26px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:50%;background:#173040;border:1px solid #2B5160;color:var(--c-accent-strong);font:700 10.5px/1 var(--mono)}
.cf-lookup-text{flex:1;min-width:0}
.cf-lookup b{color:var(--c-text);font-family:var(--mono);font-weight:600}
.cf-assigns{display:flex;flex-direction:column;gap:8px}
.cf-assign-head,.cf-assign{display:grid;grid-template-columns:minmax(0,1fr) minmax(0,1fr) 28px;column-gap:8px;align-items:start}
.cf-assigns--itam .cf-assign-head,.cf-assigns--itam .cf-assign{grid-template-columns:minmax(0,.8fr) minmax(0,1.1fr) minmax(0,1.1fr) 28px}
.cf-assign-head{padding:0 2px;font-size:var(--fs-xs);font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--c-muted)}
.cf-assign-head .cf-req{margin-left:2px}
.cf-assign{position:relative;grid-template-areas:"a e d" "ma me ."}
.cf-assigns--itam .cf-assign{grid-template-areas:"i a e d" "ma ma me ."}
.cf-assign-itam{grid-area:i;min-width:0}
.cf-assign-app{grid-area:a;min-width:0}
.cf-assign-ent{grid-area:e;min-width:0}
.cf-assign-del{grid-area:d;margin-top:calc((var(--h-input) - 28px)/2)}
.cf-assign>.cf-error{margin-top:5px}
.cf-assign-err-app{grid-area:ma}
.cf-assign-err-ent{grid-area:me}
.cf-assign.is-changed::before{content:"";position:absolute;left:-11px;top:calc(var(--h-input)/2 - 3px);width:6px;height:6px;border-radius:50%;background:var(--c-accent)}
.cf-assign-foot{display:flex;align-items:center;flex-wrap:wrap;gap:10px;margin-top:4px;font-size:var(--fs-xs);color:var(--c-muted)}
@container (max-width:600px){
  .cf-assign-head{display:none}
  .cf-assign,.cf-assigns--itam .cf-assign{grid-template-columns:minmax(0,1fr) 28px;row-gap:6px}
  .cf-assign>.cf-error{margin-top:0}
  .cf-assign{grid-template-areas:"a d" "ma ma" "e e" "me me"}
  .cf-assigns--itam .cf-assign{grid-template-areas:"i d" "a a" "ma ma" "e e" "me me"}
}
```

## Appendix D — markup

Use the app's framework (`class` → `className` in React).
- **Values are sample data.** Bind them, and every state class (`is-done`, `is-invalid`, `is-changed`), to real state.

**D.1 Page shell** (header, layout, rail, bar):
```html
<form class="cf" novalidate>
<div class="cf-inner">
<header class="cf-head">
<button type="button" class="cf-back" aria-label="Back to User Entitlements">[icon:chevron-left 17]</button>
<div class="cf-head-text">
<nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">User Entitlements</a> / <span>New</span></nav>
<h1 class="cf-title">Create account</h1>
<p class="cf-sub">Provision a new entitlement account and assign roles.</p>
</div>
<div class="cf-head-right"></div>
</header>
<div class="cf-ledger">
<div class="cf-stack">
<!-- section cards (D.3, D.4) -->
</div>
<aside class="cf-rail" aria-label="Form progress">
<div class="cf-rail-card">
<div class="cf-rail-row"><span class="cf-rail-title">Progress</span><span class="cf-rail-count">4 of 5 required</span></div>
<div class="cf-progress" role="progressbar" aria-label="Required fields complete" aria-valuemin="0" aria-valuemax="5" aria-valuenow="4"><i style="width: 80%"></i></div>
<nav class="cf-toc" aria-label="Sections">
<button type="button" class="cf-toc-item is-done"><span class="cf-toc-dot">[icon:check 11 bold]</span><span class="cf-toc-name">Account</span><span class="cf-toc-meta">Done</span></button>
<button type="button" class="cf-toc-item is-error"><span class="cf-toc-dot">!</span><span class="cf-toc-name">Access</span><span class="cf-toc-meta">1 issue</span></button>
</nav>
</div>
</aside>
</div>
</div>
<footer class="cf-bar">
<div class="cf-bar-inner">
<div class="cf-bar-meta"><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 4 of 5 required</span></div>
<span class="cf-bar-spacer"></span>
<button type="button" class="cf-btn cf-btn--ghost">Cancel</button><button type="submit" class="cf-btn cf-btn--primary">[icon:check 15 bold]Create account</button>
</div>
</footer>
</form>
```

**D.2 Field patterns** (error, open search select):
```html
<!-- error line (replaces the help line) -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f1">Bank ID<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input cf-input--mono is-invalid" id="f1" type="text" required aria-invalid="true" aria-describedby="f1-err">
<span class="cf-error" id="f1-err">[icon:alert-triangle 13]Enter a Bank ID</span>
</div>
<!-- search select, open -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="f2">Account owner<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo">
<div class="cf-control">[icon:user 15]<input id="f2" class="cf-mono" type="text" value="2023" role="combobox" aria-expanded="true" aria-controls="f2-list" aria-autocomplete="list">[icon:chevron-down 16]</div>
<div class="cf-menu" id="f2-list" role="listbox">
<button type="button" class="cf-menu-item is-active" role="option" aria-selected="false"><span class="cf-mono">2023504</span><span class="cf-menu-meta">Bank ID</span></button>
</div>
</div>
</div>
```

**D.3 Section 1 — Account** (with the Bank ID lookup strip):
```html
<section class="cf-sec" id="sec-1" aria-labelledby="sec-1-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-1-t">Account</h2><p class="cf-sec-desc">Who the account is for and who owns it.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-grid">
<div class="cf-field cf-span-6">
<label class="cf-label" for="ac-bank">Bank ID<span class="cf-req" aria-hidden="true">*</span></label>
<input class="cf-input cf-input--mono" id="ac-bank" type="text" value="8216018" required aria-describedby="ac-bank-found">
<div id="ac-bank-found"><div class="cf-lookup" role="status"><span class="cf-lookup-av">82</span><span class="cf-lookup-text">Found <b>8216018</b> · Active user · 1 existing entitlement</span></div></div>
</div>
<div class="cf-field cf-span-6">
<label class="cf-label" for="ac-owner">Account owner<span class="cf-req" aria-hidden="true">*</span></label>
<div class="cf-combo "><div class="cf-control">[icon:user 15]<input class="cf-mono" type="text" value="2023504" role="combobox" aria-expanded="false" id="ac-owner"><span class="cf-control-sub">Bank ID</span><button type="button" class="cf-control-btn" aria-label="Clear Account owner">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
</div>
<div class="cf-field cf-span-6">
<label class="cf-label" for="ac-type">Account type<span class="cf-req" aria-hidden="true">*</span></label>
<span class="cf-select"><select class="cf-input" id="ac-type"><option>User</option></select>[icon:chevron-down 16]</span>
</div>
</div>
</div>
</section>
```

**D.4 Section 2 — Access**, single mode, showing a missing entitlement:
```html
<section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-error" aria-hidden="true">!</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Access</h2><p class="cf-sec-desc">The application and entitlement this account gets.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-assigns">
<div class="cf-assign-head" aria-hidden="true"><span>Application<span class="cf-req">*</span></span><span>Entitlement<span class="cf-req">*</span></span><span></span></div>
<div class="cf-assign" role="group" aria-label="Entitlement 1">
<div class="cf-combo cf-assign-app"><div class="cf-control">[icon:app-window 15]<input class="cf-mono" type="text" value="RATAN_ENTITLEMENT_RULE" role="combobox" aria-expanded="false" aria-label="Application, row 1"><span class="cf-control-sub">ITAM 51358</span><button type="button" class="cf-control-btn" aria-label="Clear Application, row 1">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<div class="cf-combo cf-assign-ent"><div class="cf-control is-invalid">[icon:key 15]<input class="cf-mono" type="text" placeholder="Search entitlements" role="combobox" aria-expanded="false" aria-label="Entitlement, row 1" aria-invalid="true" aria-describedby="as-1-err">[icon:chevron-down 16]</div></div>
<span class="cf-error cf-assign-err-ent" id="as-1-err">[icon:alert-triangle 13]Pick an entitlement</span>
</div>
</div>
</div>
</section>
```

**D.5 Row variants and empty states:**
```html
<!-- Bank ID not found (only if the page already looks the user up) -->
<div class="cf-lookup is-error" role="status"><span class="cf-lookup-av">?</span><span class="cf-lookup-text">No user found with Bank ID <b>8216019</b></span></div>
<!-- Entitlement before an application is chosen -->
<div class="cf-combo cf-assign-ent"><div class="cf-control is-disabled">[icon:key 15]<input class="cf-mono" type="text" placeholder="Select application first" disabled role="combobox" aria-expanded="false" aria-label="Entitlement, row 1">[icon:chevron-down 16]</div></div>
<!-- ITAM column (only if applications can only be loaded per ITAM): add cf-assigns--itam to .cf-assigns,
an ITAM head cell first, and this cell first in each row -->
<div class="cf-combo cf-assign-itam"><div class="cf-control">[icon:server 15]<input class="cf-mono" type="text" value="51358" role="combobox" aria-expanded="false" aria-label="ITAM, row 1"><span class="cf-control-sub">RATAN Entitlement Rules</span><button type="button" class="cf-control-btn" aria-label="Clear ITAM, row 1">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<!-- multi mode: under the rows -->
<div class="cf-assign-foot"><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add entitlement</button><span>2 entitlements</span></div>
<!-- multi mode, no rows yet (replaces the header row and rows) -->
<div class="cf-empty">[icon:key 16]<span class="cf-empty-text">No entitlements added yet.</span><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add entitlement</button></div>
```

**D.6 Edit-mode differences:**
```html
<header class="cf-head">
<button type="button" class="cf-back" aria-label="Back to User Entitlements">[icon:chevron-left 17]</button>
<div class="cf-head-text">
<nav class="cf-crumb" aria-label="Breadcrumb"><a href="#">FMCES</a> / <a href="#">User Entitlements</a> / <span>8216018</span> / <span>Edit</span></nav>
<h1 class="cf-title">Edit account</h1>
<p class="cf-sub">Update the details below.</p>
<p class="cf-head-meta">Bank ID 8216018 · User · FMCES · 1 entitlement</p>
</div>
<div class="cf-head-right"><span class="cf-badge cf-badge--ok">Active</span></div>
</header>
<!-- locked field -->
<div class="cf-field cf-span-6">
<label class="cf-label" for="ac-bank">Bank ID</label>
<div class="cf-control is-readonly">[icon:user 15]<input id="ac-bank" class="cf-mono" type="text" value="8216018" readonly aria-describedby="ac-bank-help">[icon:lock 14]</div>
<span class="cf-help" id="ac-bank-help">The Bank ID can’t be changed.</span>
</div>
<!-- Access in multi mode; the second row was added (is-changed) -->
<section class="cf-sec" id="sec-2" aria-labelledby="sec-2-t">
<div class="cf-sec-head">
<span class="cf-sec-num is-done" aria-hidden="true">[icon:check 13 bold]</span>
<div class="cf-sec-titles"><h2 class="cf-sec-title" id="sec-2-t">Access</h2><p class="cf-sec-desc">The applications and entitlements this account has.</p></div>
</div>
<div class="cf-sec-body">
<div class="cf-assigns">
<div class="cf-assign-head" aria-hidden="true"><span>Application<span class="cf-req">*</span></span><span>Entitlement<span class="cf-req">*</span></span><span></span></div>
<div class="cf-assign" role="group" aria-label="Entitlement 1">
<div class="cf-combo cf-assign-app"><div class="cf-control">[icon:app-window 15]<input class="cf-mono" type="text" value="FMCES" role="combobox" aria-expanded="false" aria-label="Application, row 1"><button type="button" class="cf-control-btn" aria-label="Clear Application, row 1">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<div class="cf-combo cf-assign-ent"><div class="cf-control">[icon:key 15]<input class="cf-mono" type="text" value="CES Application View" role="combobox" aria-expanded="false" aria-label="Entitlement, row 1"><span class="cf-control-sub">#100</span><button type="button" class="cf-control-btn" aria-label="Clear Entitlement, row 1">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<button type="button" class="cf-icon-btn cf-icon-btn--danger cf-assign-del" aria-label="Remove row 1">[icon:trash-2 15]</button>
</div>
<div class="cf-assign is-changed" role="group" aria-label="Entitlement 2">
<div class="cf-combo cf-assign-app"><div class="cf-control">[icon:app-window 15]<input class="cf-mono" type="text" value="FMCES" role="combobox" aria-expanded="false" aria-label="Application, row 2"><button type="button" class="cf-control-btn" aria-label="Clear Application, row 2">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<div class="cf-combo cf-assign-ent"><div class="cf-control">[icon:key 15]<input class="cf-mono" type="text" value="CES Application User" role="combobox" aria-expanded="false" aria-label="Entitlement, row 2"><button type="button" class="cf-control-btn" aria-label="Clear Entitlement, row 2">[icon:x 13]</button>[icon:chevron-down 16]</div></div>
<button type="button" class="cf-icon-btn cf-icon-btn--danger cf-assign-del" aria-label="Remove row 2">[icon:trash-2 15]</button>
</div>
<div class="cf-assign-foot"><button type="button" class="cf-btn cf-btn--secondary cf-btn--sm">[icon:plus 14]Add entitlement</button><span>2 entitlements</span></div>
</div>
</div>
</section>
<!-- changed field: Edited tag in the label + is-changed on the control -->
<label class="cf-label" for="ac-type">Account type<span class="cf-req" aria-hidden="true">*</span><span class="cf-edited">Edited</span></label>
<!-- bar -->
<div class="cf-bar-meta"><span class="is-changed">1 unsaved change</span><span>·</span><span><span class="cf-req">*</span> Required</span><span class="cf-bar-progress">· 5 of 5 required</span></div>
<button type="button" class="cf-btn cf-btn--ghost">[icon:rotate-ccw 15]Discard changes</button>
<button type="submit" class="cf-btn cf-btn--primary">[icon:check 15 bold]Save changes</button>
```

**D.7 Loading placeholders** (one per field, with the same spans, inside each `.cf-sec-body`):
```html
<div class="cf-grid" aria-busy="true" aria-label="Loading">
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-6"><span class="cf-skel cf-skel--label"></span><span class="cf-skel"></span></div>
<div class="cf-field cf-span-12"><span class="cf-skel cf-skel--label"></span><span class="cf-skel" style="height: 76px"></span></div>
</div>
```

## Appendix E — icon paths

Every icon uses a 24×24 viewBox, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.75"` (2.25 where marked `bold`), and round line caps and joins.

- `alert-triangle`: `M12 4 2.8 20h18.4zM12 10v4.5M12 17.2h.01`
- `app-window`: `M3 5h18v14H3zM3 9h18M6.5 7h.01M9 7h.01`
- `check`: `M5 12.5l4.5 4.5L19 7.5`
- `chevron-down`: `M6 9l6 6 6-6`
- `chevron-left`: `M15 6l-6 6 6 6`
- `key`: `M15 4a5 5 0 1 0 0 10 5 5 0 0 0 0-10zM11.5 12.5 3 21M6 18l2.5 2.5M8.5 15.5 11 18`
- `lock`: `M6 11h12v10H6zM8.5 11V7.5a3.5 3.5 0 0 1 7 0V11`
- `plus`: `M12 5v14M5 12h14`
- `rotate-ccw`: `M9 14 4 9l5-5M4 9h10a6 6 0 0 1 0 12h-3`
- `server`: `M4 4h16v6H4zM4 14h16v6H4zM8 7h.01M8 17h.01`
- `trash-2`: `M4 7h16M9 7V4h6v3M6 7l1 13h10l1-13`
- `user`: `M12 12a4 4 0 1 0 0-8 4 4 0 0 0 0 8zM4 21a8 8 0 0 1 16 0`
- `x`: `M6 6l12 12M18 6 6 18`
