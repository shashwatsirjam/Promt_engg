# FMCES: restyle every Create and Edit page (styling + a few listed additions, self-verified on localhost)

You are working in the FMCES codebase ("Markets Operations One", dark navy + teal). The app runs locally at **http://localhost:8001** and you can open it in the browser.

The list pages were restyled in an earlier job with `fmces-pages.css`. Now the **create and edit pages** must get the same look:
- Add application
- Add ITAM
- Add features
- Add actions
- Create data profile
- Create data policy
- Create entitlement (with the Select data policies / Select data profiles panels)
- Create account
- and the **Edit** page of each.

Today these pages already have the right structure: numbered section cards, a Progress panel, an "After you submit" panel, an action bar, a spreadsheet-style table for features and actions, and the picker panels. But they still use the **old look**: grey flat cards, blue buttons, library inputs.

Your job:
1. Make every one of these pages look **exactly** like the reference pages in `fmces-forms-kit/reference/`.
2. Add the few new pieces listed in §6.
3. Make every **Edit** page the same page as its Create page, filled with the record (§3).
4. **Prove it** with the included self-check on every page and state (§10).

**Do not stop until every page and state passes.** Open the pages yourself, run the self-check, compare with the references, fix, and repeat. Only stop for the three reasons in §11.3.

---

## 0. The kit

Put the folder `fmces-forms-kit/` in the project root. If it is missing, that is the one thing to ask me about before starting.

| File | What it is | What you do with it |
|---|---|---|
| `PROMPT.md` | This document. | Follow it. |
| `fmces-forms.css` | The stylesheet for the create and edit pages (about 45 KB). Every rule is scoped under `.lp-root`. | Copy it **byte for byte** into the app. Load it globally **after `fmces-pages.css`** and after every other stylesheet. Never edit, reformat, translate or split it. |
| `fmces-pages.css` | The list-pages stylesheet (v2), because the forms reuse its tokens, buttons, status dots, selects and keyframes. | The app should already have it from the list-pages job. Keep the app's copy. Use this file only if the app has no copy, or the self-check says it is missing or outdated. |
| `markup.md` | The HTML of every component, copied from the reference pages, with the final class names. | Translate to your framework. Bind your existing state and handlers. |
| `reference/` | 20 reference pages (`.html`) with full-page PNGs, plus `index.html`. | Your visual target. Open them in the browser at 1440 × 900 next to your page. Read their source when unsure. They link `../fmces-pages.css` and `../fmces-forms.css`, so open them from inside the kit folder: serve it (`npx serve fmces-forms-kit`) or open the files directly. |
| `fmces-forms-verify.js` | The self-check. | Run it on every page and state (§10). |

The reference pages show the **look only**:
- Their data (application names, IDs, people, conditions), the top bar and the dock are stand-ins.
- **Never copy sample data into the app.** The app's own fields, labels, placeholders, validation messages and data always win.
- The PNGs were rendered without the web fonts. The HTML pages load Manrope and JetBrains Mono, so trust the HTML for typography.

Keep these files up to date as you work:
- `fmces-forms-kit/out/INVENTORY.md`: written **before** you change anything. For each create and edit page, list every field (label, required or optional, control type, placeholder, hint, validation rules and messages), every button, every cascade or auto-fill, every picker and table feature, and every message and toast that exists today. It is your checklist at the end.
- `fmces-forms-kit/out/DECISIONS.md`: every default you used (§11.4) and every judgement call, one line each.
- `fmces-forms-kit/out/REPORT.md`: the final report (§12).

---

## 1. Rules

### 1.1 Keep everything

Everything that works today must keep working exactly as before, on create **and** edit:

| Area | What stays the same |
|---|---|
| Plumbing | Routes, API calls, payloads, auth, role checks. |
| Fields | Every field, its order inside its section, its validation rules and messages, its required / optional status, and its default value. |
| Lookups | Cascades and auto-fill: ITAM ID → ITAM name; Data source → Data source attribute; ITAM → Application → Entitlement. |
| Buttons | **Reset**, **Cancel**, **Save draft** and **Submit for approval**, with what each one does. Create account's **Create account**. |
| Grid editor (Features and Actions) | Add row; Paste from Excel; Import CSV; Tab = next cell; Enter = next row; Ctrl V pastes rows; "Use a different application per row"; the Check column's validations; removing rows. |
| Conditions (data profiles and data policies) | Add condition (Key / Operator / Value), removing conditions, the operators and value pickers. |
| Entitlement | Add policy / Add profile with their pickers (search, All / Selected, expandable details with conditions, Cancel, Add); Feature configuration with Application, Privileged, List / Grid preview, Add feature. |
| Data policy | Select data profiles (the same picker), Global policy. |

You may add classes and wrappers, swap an element's **visual** form while it keeps the same handler and state, and add the visual pieces in §6.

You may **not**:
- change backend code or add endpoints;
- rename labels;
- remove or reorder fields;
- change validation;
- change what a button does.

### 1.2 Names and words

Keep the app's page titles, subtitles, section titles, section descriptions, labels, placeholders, hints and button labels **exactly**. Where the reference shows wording the app doesn't have yet (for example the issues banner, the Changes panel, "Auto-filled", the Summary sentence), use the reference wording.

The breadcrumb is the app's own (for example "FMCES / App Management / New"). The module number in the eyebrow follows dock order:

| Number | Module | Pages |
|---|---|---|
| 01 | App Management | application, ITAM, features, actions |
| 02 | Data Profiles | data profile |
| 03 | Data Policies | data policy |
| 04 | Entitlement Management | entitlement |
| 05 | Entitlement Users | account |

### 1.3 No library look on these pages (this is what kept the old blue look)

On these pages, render these as native HTML elements with the `lp-` classes from `markup.md`, calling the same handlers and reading the same state:
- buttons;
- text inputs and textareas;
- select triggers and their option lists;
- switches;
- the drop zone;
- segmented controls;
- section cards;
- the header;
- the rail cards;
- the action bar;
- the picker panel.

Keep a library component **only** where it holds logic you would otherwise rewrite:
- a date picker's calendar;
- an autocomplete's async search;
- a modal's focus trap;
- the file input.

For those:
- put the `lp-` classes on their visible parts;
- neutralise the library's own colours, borders, heights and fonts **only inside `.lp-root`**, in a separate file `fmces-forms-overrides.css` loaded after `fmces-forms.css`;
- **never change the global theme**.

Anything portaled to `<body>` (dropdown lists, the picker panel, tooltips, toasts) must render inside an `.lp-root`. Portal it into the page's `.lp-root`, or wrap it in its own `<div class="lp-root">`.

### 1.4 Don't touch

- The top app bar.
- The workspace tabs.
- The dock (its gliding highlight already exists from the list-pages job).
- The list pages.
- The record drawers.

### 1.5 Fonts

Manrope 200–800 and JetBrains Mono 400–600 must be loaded once, globally. The list-pages job should already have done this. If not:

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@200;300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">
```

---

## 2. One page shell for every form

Every create / edit page has the same shell. See `markup.md` → "Page skeleton".

```html
<div class="lp-root" data-lp-form="{application|itam|feature|action|profile|policy|entitlement|account}" data-lp-mode="{create|edit}">
  <div class="lp-pg">
    <div class="lp-aur" aria-hidden="true"><i class="lp-a1"></i><i class="lp-a2"></i></div>
    <div class="lp-pga">
      <header class="lp-ph lp-cf-hd lp-rz">…</header>
      <div class="lp-cf-wrap">
        <div class="lp-cf-main">[issues banner] · section cards · action bar</div>
        <aside class="lp-cf-rail">rail cards</aside>
      </div>
    </div>
  </div>
  [picker panel]
</div>
```

- `data-lp-form` and `data-lp-mode` are required. The self-check reads them.
- The shell replaces the page's current outer container and its padding. `.lp-pg` provides the padding, so make sure it isn't doubled.
- `.lp-cf-wrap` is two columns: the main column and a **316px sticky rail**, with a 20px gap. Under 1100px the rail moves below the main column. Under 900px every field takes the full width. The CSS does all of this.
- `lp-rz` blocks rise in on load. `style="--d: N"` sets their order: header 0, sections 1, 2, 3; rail cards 1, 2, 3.

---

## 3. Create and Edit are the same page

Use **one component per record type** with a `mode` (create | edit).
- **Edit** loads the record (with the existing API call) and fills every field.
- Sections, fields, order and controls are **identical** in both modes.

In Edit mode only:

| Piece | Create | Edit |
|---|---|---|
| Title | "Add application" (the app's create title) | "Edit application" (the app's edit title, or "Edit …") |
| Crumb last item | New | Edit |
| Under the title | subtitle | **record chip** `.lp-cf-rec` (monogram, record name, status dot + word, version), then the subtitle |
| Right of the header | status pill `.lp-cf-st.lp-new` "New" (or `.lp-draft` "Draft" where the app saves drafts, e.g. application) | `.lp-cf-st.lp-draft` "Editing V2" + `.lp-cf-sv` "Draft saved 2 min ago" (only if the app knows when the draft was saved) |
| Changed fields | — | label gets `<span class="lp-cf-ch">Changed</span>`, the control gets `lp-chg` (amber border), and `.lp-cf-was` shows the approved value struck through ("Was …"). Grid rows: `lp-chgr` + Check "Changed". Conditions / attached items added in this edit: `lp-new`. Matrix cells changed: `lp-chg`. |
| Section pill | "3 of 6" / "Optional" / "1 issue" | "2 changed" (`lp-chg`) when it has changes, otherwise "Done" (`lp-ok`) |
| Rail, first card | — | **Your changes** (`.lp-cf-chl`): one `.lp-cf-chr` per changed field (field name, old struck through → new), "Compared with the approved version" and **Discard all** |
| Rail lifecycle | **After you submit**: Draft (You are here) → Pending approval → Approved "Locked as V1" → Active | **When you submit**: Approved · V2 (current, "Live now; stays live until V3 is approved") → Draft · V3 (You are here) → Pending approval → Approved · V3 ("Replaces V2") |
| Action bar | Reset (or Cancel) · Save draft · Submit for approval; message "Not saved yet" | **Discard changes** · Save draft · Submit for approval; message `.lp-msg.lp-chg` "2 unsaved changes" |
| Features / Actions grid | tools (Add row, Paste from Excel, Import CSV), keyboard hint, per-row switch | one row (the record), **no** tools, **no** keyboard hint, **no** per-row switch |
| Account | Create account | Save changes; rail = Your changes + Progress + Summary |

**Change tracking** (client-side only):
- When the record loads, keep a copy of it.
- A field is "changed" when its current value differs from the copy. Compare trimmed strings; lists by their item IDs.
- **Discard all** / **Discard changes** restores the copy.
- The count drives the bar message and the "Your changes" card.

**Versions:**
- Take the current version from the record, for example `version: 2` → "V2". The next version is V+1.
- If a record type has no versions (for example accounts), omit the version chip and use the Summary card instead of the lifecycle (as in `08-account-edit`).

---

## 4. Shared components (all in `markup.md`; sizes are in the CSS)

### 4.1 Header (`.lp-ph.lp-cf-hd`)

- **Eyebrow `.lp-eb`:**
  - back button `.lp-cf-bk` (28px; your existing back / cancel navigation);
  - module number `<b>` (mono teal);
  - a short line;
  - the crumb `.lp-cf-crumb` (`<s>/</s>` separators, last item in `<em>`).
- **Watermark:** `.lp-bign`, the same number, very faint behind the title.
- **Title:** `.lp-pt`, 36px / 300.
- **Subtitle:** `.lp-ps`.
- **Right side:** `.lp-cf-hr` holding the status pill `.lp-cf-st` and the saved line `.lp-cf-sv`.

### 4.2 Section card (`.lp-cf-sc`)

- **Header `.lp-cf-sh`:**
  - step circle `.lp-cf-sn`;
  - `.lp-cf-stt` with an `h2` (15.5px / 700) and a one-line `p`;
  - on the right, `.lp-cf-shr` with optional buttons (Add condition, Select data profiles…) and the status pill `.lp-cf-pl`.
- **Body:** `.lp-cf-sb`, holding a 12-column grid `.lp-cf-g`.
- **Step circle states** (compute them from the section's fields):
  - (none) not started;
  - `lp-cur`: the section holds the focused field;
  - `lp-done`: every required field in it is valid (shows a check);
  - `lp-bad`: it has an error after a submit attempt (shows "!");
  - `lp-op2`: the whole section is optional (dashed).
- **Card states:** add `lp-act` to the card holding the focused field (teal glow; only one at a time), and `lp-bad` when it has errors.
- **Pill `.lp-cf-pl`:**
  - plain "3 of 6" (required fields valid / total);
  - `lp-ok` with a check "3 of 3" / "Done" / "2 selected";
  - `lp-op2` "Optional";
  - `lp-bad` "1 issue";
  - `lp-chg` "2 changed".

### 4.3 Fields (`.lp-cf-f` with `style="--s: N"`)

**Label** `.lp-cf-l`: 12px / 700 / `#A7B8C4`, followed by:
- `<span class="lp-cf-rq">*</span>` when required;
- `<span class="lp-cf-op">Optional</span>` when the app marks the field optional;
- `<span class="lp-cf-ch">Changed</span>` in Edit.

**Controls:**

| Control | Class | Notes |
|---|---|---|
| Text input | `.lp-cf-in` | 44px, radius 10, border `#2B4556`. Add `lp-mono` for IDs, codes, DLs, Bank IDs. Add `lp-pl` with a left icon `.lp-cf-ic.lp-l` (calendar for dates). Focus: teal border + ring. |
| Textarea | `.lp-cf-in.lp-cf-ta` | Min 92px. Show the counter `.lp-cf-cnt` only if the field has a max length today. |
| Select trigger | `.lp-cf-sl` | Optional leading icon, mono tag `.lp-tg2` (ID), monogram `.lp-mgx`, value `.lp-v` (placeholder: `.lp-v.lp-phd`), chevron. `lp-on` while open. `lp-dis` (dashed) while it depends on another field, e.g. "Select ITAM ID first". |
| Option list | `.lp-cf-dd` | Search `.lp-psr`; rows `.lp-opt` with `.lp-cf-otag` (mono ID), name and `<small>` meta; selected row `lp-on` + check; keyboard hint at the bottom. Keep your option source and search logic. |
| Auto-filled value | `.lp-cf-af` | Lock icon, value, "Auto-filled" tag. Empty: `.lp-cf-af.lp-empty` with the dependency text. Read-only. |
| Switch | `label.lp-cf-sw` | Native checkbox + `.lp-k` + label / sub-line. |
| Drop zone | `.lp-cf-dz` | Icon tile, "Drop the … here, or browse", the accepted types line, PDF / DOC / TXT badges. Keep drag-and-drop and the file input. |
| File card | `.lp-cf-fc` | Shown once a file is chosen: type badge, name, size / meta, Replace, remove. |

**Under the control** (one of these):
- hint `.lp-cf-h`;
- error `.lp-cf-e` (with the alert icon), and the control gets `lp-inv`;
- Edit "Was" `.lp-cf-was`.

Errors show the app's existing validation messages, on blur and after submit, as today.

### 4.4 Issues banner (`.lp-cf-bn`)

- Shown at the top of the main column **after a submit attempt** while required fields are missing or invalid.
- It names the fields as links (`.lp-lk2`). Clicking one scrolls to that field and focuses it.
- It disappears when everything is valid.

### 4.5 Action bar (`.lp-cf-ab`)

- Sticky at the bottom of the main column (CSS).
- **Left:** `.lp-rqn` "* Required", a separator, and the message `.lp-msg`:
  - "Not saved yet";
  - `lp-ok` "Ready to submit";
  - `lp-bad` "Fix 2 issues to submit";
  - `lp-chg` "N unsaved changes".
- **Right:** quiet buttons `lp-btn lp-btn-q` (Reset / Cancel / Discard changes), `lp-btn lp-btn-g` Save draft, and **one** primary `lp-btn lp-btn-p` (Submit for approval / Create account / Save changes), always last.
- Keep each button's existing handler and enabled / disabled rules.

### 4.6 Rail cards (`.lp-cf-rc`)

**Progress** (always first, except in Edit where "Your changes" is first):
- `.lp-cf-rh` "PROGRESS" + "4 of 7 required" (or `lp-ok` "All set", or `lp-bad` "2 issues").
- `.lp-cf-pn`: the big count, 40px / 200.
- `.lp-cf-pb`: **one `<i>` per required field**. `lp-d` = valid, `lp-b` = has an error; the rest are empty.
- `.lp-cf-pr` rows, **one per section**: step mark, section name, status ("3 of 6", "Optional", `lp-bad` "1 issue", `lp-chg` "1 changed", `lp-ok`). The current section gets `lp-cur`. Clicking a row scrolls to its section.
- Count only the fields the app requires today. A section that needs at least one item (for example profile conditions) counts as one required item.

**Features / Actions (create):** the progress card counts **rows** instead ("ROWS · 2 / 3 rows ready to submit"), one `.lp-cf-pr` per grid row: Ready / Duplicate / 1 issue / Empty.

**Lifecycle** `.lp-cf-tl` (§3): four `.lp-cf-tlr` steps. Dot colours: `lp-draft` amber (current), `lp-pend` lavender, `lp-appr` teal, `lp-live` light teal, `lp-past` filled teal. The current step has `lp-cur` + `<span class="lp-yh">You are here</span>`.
- If the app's approval flow differs (for example no "Active" step), use the app's steps and tell me in DECISIONS.md.

**Your changes** (Edit, §3).

**Paste from Excel** (Features / Actions create): a mini sheet `.lp-cf-xl` showing column A = Name, B = Description, plus the shortcut lines. Static help.

**Summary** (account): `.lp-cf-sum`, one sentence built from the form: "Person account `1466717`, owned by `1466717`, will get `CES Consumer` in `FMCES`." Empty parts show as dashed placeholders (`b.lp-phd`).

### 4.7 Picker panel (Select data policies / Select data profiles)

The same design as the record-drawer picker, at 640px:
- **Scrim and panel:** `.lp-cf-pks` + `.lp-cf-pk`.
- **Header:** the context label ("New entitlement" / "Edit entitlement" / "New data policy"), the title, Esc, close.
- **Tools:** search + `.lp-vseg` All N / Selected M.
- **Rows `.lp-cf-pki`:** a tick box `.lp-cf-pkc` that only toggles selection, and the body button `.lp-cf-pki-m` that expands the details. Row states: `lp-on` (ticked), `lp-open` (details shown).
- **Details:** `.lp-kv3` chips (ID, Owner, Modified, Active), then the **Conditions** table `.lp-cf-ctt`: Field / Operator (`.lp-op` chip) / Values (`.lp-vc` chips) / Logic.
- **Footer:** the chosen items as chips `.lp-cf-chq`, Cancel, and the primary "Add N policies" (disabled with `lp-dis` when nothing changed).
- Keep the existing picker API, search, and pre-ticking of the current selection.

---

## 5. Page by page

Fields are listed in the order the app shows them today (seen on 27 Oct). If the code has a field that isn't listed here, keep it: put it in the section where it is today, write it in INVENTORY.md, and carry on. `sN` = the `--s` span out of 12.

### 5.1 Application: `data-lp-form="application"`

References: `01-application-create`, `-create-itam-open`, `-create-issues`, `01-application-edit`.

| Section | Fields |
|---|---|
| 1 **ITAM & ownership**: "Pick the ITAM first; its details fill in automatically." | ITAM ID * (s4): select with the mono ID tag, searchable, rows "55508 · FMCES". ITAM name * (s4): **auto-filled** from the ITAM, read-only. Application name * (s4, mono). App owner * (s6, mono). Application dev DL * (s6, mono). Description * (s12, textarea). |
| 2 **Change & onboarding** (optional): "Change request and go-live details." | CR number (s6, Optional, mono). FMCES onboard date (s6, Optional, calendar icon; keep your date picker). |
| 3 **OLA document**: "The signed operating-level agreement for this application." | Drop zone (PDF, Word or text) → file card. |

- **Rail:** Progress (7 required = 6 + OLA) and After you submit.
- **Status pill:** Draft.
- **Action bar:** Reset · Save draft · Submit for approval.

### 5.2 ITAM: `itam`

References: `02-itam-create`, `02-itam-edit`.

**1 ITAM details**: "All five fields are needed to onboard an application under this ITAM."
- ITAM name * (s6, mono);
- ITAM ID * (s6, mono, hint "The ID from the ITAM register.");
- SNOW group * (s6, hint "The ServiceNow assignment group that supports this application.");
- Support DL * (s6, mono, placeholder "e.g. DL-FMCES-L2-SUPPORT");
- Description * (s12).

**Rail and bar:**
- Rail: Progress (5) and lifecycle.
- Status pill: New.
- Action bar: Cancel · Save draft · Submit for approval.

### 5.3 Features: `feature` / 5.4 Actions: `action`

References: `03-feature-create`, `03-feature-edit`, `04-action-create-per-row`, `04-action-edit`.

**1 Application:**
- The band `.lp-cf-band`: label, the big trigger `.lp-scf` (monogram, name, meta "ITAM 51024 · 10 features · 52 actions" if you have the counts), and `.lp-kv3` facts (ITAM, Features / Actions count, Status). This is the same as the Features list page.
- Under it, the switch **"Use a different application per row"** ("Every row below is added to this application." / "Pick the application in each row.").
- With the switch on, the band label reads "Default application".

**2 Features to add / Actions to add** (`.lp-cf-ge`):
- **Header:** the title, the description "One row per feature. Type, or paste straight from Excel.", the tools **Add row**, **Paste from Excel**, **Import CSV** (`lp-btn lp-btn-g lp-btn-sm`), and the rows pill.
- **Table:**
  - Header row `.lp-cf-gr.lp-hd`: # · [Application *] · Feature name * · Description * · Check · (remove). `--gc` holds the column template, shown in the reference.
  - Rows `.lp-cf-gr`, with inline inputs `.lp-cf-ci` (name bold; description `.lp-d`; `lp-fo` = focused; `lp-inv` = missing). With per-row on, each row also has the application picker `.lp-cf-gsl`.
- **Check column `.lp-cf-rs`:**
  - `lp-new` "New row" (empty);
  - `lp-ok` "Ready" (required cells filled and valid);
  - `lp-warn` + row `lp-warn` for a duplicate (for example "Already in SAWFL"), **only if the app already detects duplicates**;
  - `lp-bad` + row `lp-bad` for a missing required cell ("Description needed").
- **Footer:** `.lp-cf-gk` "Tab next cell · Enter next row · Ctrl V paste rows from Excel: name in column 1, description in column 2".

**Rail and bar:**
- Rail: Rows progress (§4.6), Paste from Excel tips, After you submit.
- Action bar: Cancel · Save draft · Submit for approval, with the message "4 features · 1 needs a look".
- Edit: §3 (one row, no tools / hint / switch).

### 5.5 Data profile: `profile`

References: `05-profile-create`, `05-profile-edit`.

| Section | Fields |
|---|---|
| 1 **Profile details**: "Name the profile and pick the application it applies to." | Profile name * (s6, mono). Application * (s6, select, "Search applications"). Description * (s12). |
| 2 **Data source**: "Where the data comes from and which attribute the conditions filter on." | Data source name * (s4, DB icon, mono). Data source attribute * (s4; `lp-dis` "Select data source first" until a source is picked). Data source attribute name (s4, Optional). |
| 3 **Profile conditions**: "Rows are visible only when every condition matches." | **Add condition** button in the section header. Below it: the empty box `.lp-cf-empty` ("No conditions added yet"), or the list `.lp-cf-conds`, plus the **New condition** box `.lp-cf-cb` while adding (§6). |

- **Rail:** Progress (6 = 3 + 2 + at least one condition).
- **Action bar:** Reset · Save draft · Submit for approval.

### 5.6 Data policy: `policy`

References: `06-policy-create`, `06-policy-edit`.

| Section | Fields |
|---|---|
| 1 **Policy details**: "Name this policy and pick the application it applies to." | Policy name * (s6). Application * (s6). Description * (s12). Global policy (s12, Optional): switch "Apply this policy globally". |
| 2 **Data profiles**: "Pick the data profiles this policy grants access to." | **Select data profiles** button in the header (opens the picker, §4.7). Chosen profiles as `.lp-cf-its` rows (monogram, name, `#id · first condition`, owner, remove). Empty: `.lp-cf-ie` "No data profiles selected". |
| 3 **Policy conditions**: "Access is granted only when every condition matches." | Same builder as profiles. Optional if the app allows none (pill "Optional"). |

### 5.7 Entitlement: `entitlement`

References: `07-entitlement-create`, `-create-pick-policies`, `-create-pick-profiles`, `07-entitlement-edit-grid`.

| Section | Fields |
|---|---|
| 1 **Basic information**: "How this entitlement is named and described across FMCES." | Entitlement name * (s6, mono). Display name * (s6). Description * (s12). |
| 2 **Data permissions** (optional): "Optional. Limit which rows a user with this entitlement can see." | Two cards `.lp-cf-dp2 > .lp-cf-dpc`: **Data policies** and **Data profiles**. Each has an icon tile, title, count, "Optional", the **Add policy** / **Add profile** button (opens the picker) and the attached items, or the empty line "No data policies attached". |
| 3 **Feature configuration**: "Pick the application, then what this entitlement can do in each feature." | Application band (§5.3). A second row with **Privileged** (Optional switch), the `.lp-vseg` **List / Grid preview** and **Add feature**. **List**: `.lp-cf-fl` rows (feature monogram + name, action chips `.lp-acp`, Privileged `.lp-yn`, remove), and while adding, the box `.lp-cf-cb` (Feature select, action toggle chips — on = `.lp-acp`, off = `.lp-acp.lp-off` — Privileged, Add, Cancel). **Grid preview**: `.lp-cf-mx` features × actions (View, Edit, Approve, Download, Upload, Delete). Cells: `lp-on` granted, `lp-avl` available but not granted, `lp-na` "–" not available for that feature, `lp-chg` changed in this edit. Legend `.lp-cf-lgd`. If the grid preview lets you click cells today, keep that; otherwise it stays read-only and you remove the "Click a cell" legend text. |

### 5.8 Account: `account`

References: `08-account-create`, `08-account-edit`.

| Section | Fields |
|---|---|
| 1 **Account**: "Who the account is for and who owns it." | Bank ID * (s4, mono). Account owner * (s4, mono). Account type * (s4, select with a person / server icon). |
| 2 **Access**: "The application and entitlement this account gets." | The cascade `.lp-cf-cas`: three big selects `.lp-sel2` (numbered badges that turn into checks when chosen). ITAM (optional, "narrows the list of applications") → Application * ("Select ITAM first") → Entitlement * ("Select application first"). The same look as the Entitlement Users finder. |

- **Rail:** Progress, then Summary (§4.6).
- **Action bar:** Cancel · **Create account** (Edit: **Save changes**).

---

## 6. New pieces and their logic (build them; they are part of the design)

| # | Piece | Logic |
|---|---|---|
| 1 | Section step states and pills | Computed from the section's required fields and errors (§4.2). |
| 2 | Active section glow | `lp-act` on the section card holding `document.activeElement` (focusin / focusout). |
| 3 | Progress card | §4.6. Updates live as fields become valid. Clicking a row scrolls to that section (`scrollIntoView({behavior:'smooth', block:'start'})`). |
| 4 | Issues banner | §4.4. |
| 5 | Auto-filled ITAM name | Shows the name of the chosen ITAM, read-only. Today's logic already fills it; only the look is new. |
| 6 | Conditions as WHERE / AND rows | `.lp-cf-conds`: first row `WHERE`, then `AND` (the description says every condition must match). Field in mono with its label below, operator chip `.lp-op`, value chips `.lp-vc` (code in `<em>`), remove. |
| 7 | New condition box | **Add condition** opens `.lp-cf-cb` (Key, Operator, Value multi-select, **Add**, **Cancel**). Add appends to the list using your existing add-condition logic; Cancel closes. |
| 8 | Grid Check column states | §5.3. Only use the states your validation can really produce. |
| 9 | Paste from Excel tips card | Static help (§4.6). |
| 10 | Account Summary sentence | Built from the form values (§4.6). |
| 11 | Edit: record chip, Changed markers, Was values, Your changes card, When you submit lifecycle, Discard changes | §3. |

---

## 7. Colours

Use only the tokens from `fmces-pages.css`:

| Use | Colour |
|---|---|
| Teal primary | `#2DD4BF` |
| Light teal | `#5EEAD4` |
| Text on teal | `#062A26` |
| Cards | `#152733` |
| Borders / dividers / button borders | `#23394A` / `#1E3240` / `#2B4556` |
| Text | `#E6EEF2` / `#A7B8C4` / `#86A0AE` / `#6F8896` |
| Amber (draft, changed) | `#F6C177` |
| Lavender (pending) | `#C9B6FF` |
| Red (errors) | `#F4A3A3` |
| Light info blue (operator chips, DOC badge) | `#93C5FD` |

**No other blue.** The old `#1E88E5` / `#2196F3` blue must be gone from these pages; the self-check scans every element.

---

## 8. Motion

All motion is in the CSS:
- blocks rise in;
- grid and condition rows stagger;
- checks draw themselves;
- the current lifecycle dot breathes;
- the drop-zone icon bobs;
- the picker slides in;
- the option list pops.

JavaScript only toggles the state classes (`lp-act`, `lp-on`, `lp-done`…).
Everything respects `prefers-reduced-motion`; the CSS handles it.

---

## 9. Accessibility

- Every control has a real `<label>` or `aria-label`.
- Required fields carry `aria-required="true"`.
- Fields with errors carry `aria-invalid="true"` and `aria-describedby` pointing at their `.lp-cf-e`.
- Select triggers: `aria-haspopup="listbox"` and `aria-expanded`. Options: `role="option"` and `aria-selected`.
- The picker: `role="dialog"`, Esc closes it, and focus returns to the button that opened it.
- The grid editor: `role="table"`.
- The switch is a real checkbox.
- Focus rings come from the CSS. Keep them.

---

## 10. Verification

### 10.1 Load the self-check

Evaluate `fmces-forms-kit/fmces-forms-verify.js` in the page (browser tool or console), then run `await lpFormVerify()` and read the result (`summary`, `problems`).

Or, in a **development build only**, include the script and open any page with `?lpformverify` in the URL: it runs after 2.5s and shows a panel bottom-right. Never ship the script to production.

Run it:
- with the page fully loaded;
- at **1440 × 900**;
- with the mouse outside the form;
- with no field focused, unless the state is about a focused field.

### 10.2 States (each must reach 0 FAIL)

| # | Page / state | How to get there | Reference |
|---|---|---|---|
| 1 | Add application, partly filled | App Management → Add New → Application; pick an ITAM, type a name, click App owner | `01-application-create` |
| 2 | … ITAM ID list open | click ITAM ID | `01-application-create-itam-open` |
| 3 | … submitted with issues | leave ITAM ID and the OLA empty, click Submit for approval | `01-application-create-issues` |
| 4 | Edit application, 2 changes | open an application → Edit; change App owner and Description | `01-application-edit` |
| 5 | Add ITAM | Add New → ITAM; fill two fields | `02-itam-create` |
| 6 | Edit ITAM, 1 change | | `02-itam-edit` |
| 7 | Add features, 4 rows | Add New → Feature; pick an application; type or paste rows | `03-feature-create` |
| 8 | Edit feature, 1 change | | `03-feature-edit` |
| 9 | Add actions, per-row on | Add New → Action; switch "Use a different application per row" on | `04-action-create-per-row` |
| 10 | Edit action, 1 change | | `04-action-edit` |
| 11 | Create data profile | fill section 1, pick a data source, focus the attribute | `05-profile-create` |
| 12 | Edit data profile, add a condition | | `05-profile-edit` |
| 13 | Create data policy, adding a condition | select data profiles; click Add condition | `06-policy-create` |
| 14 | Edit data policy, add a profile | | `06-policy-edit` |
| 15 | Create entitlement, list view, adding a feature | | `07-entitlement-create` |
| 16 | … Select data policies open, one expanded | click Add policy; tick one; open its details | `07-entitlement-create-pick-policies` |
| 17 | … Select data profiles open | click Add profile; open a profile with many values | `07-entitlement-create-pick-profiles` |
| 18 | Edit entitlement, Grid preview, 2 changes | | `07-entitlement-edit-grid` |
| 19 | Create account, entitlement not picked yet | | `08-account-create` |
| 20 | Edit account, entitlement changed | | `08-account-edit` |
| 21 | Every Create page **empty** (just opened) | open each Add / Create page | (same look as its reference, nothing filled) |
| 22 | Every page at **1100px** and **900px** wide | resize | rail below the main column; fields full width at 900 |

### 10.3 Visual comparison (after 0 FAIL)

For each state with a reference:
1. Screenshot your page and the reference HTML at 1440 × 900 (full page).
2. List every difference in **look**: colours, borders, radius, shadows, spacing, sizes, fonts, icons, positions, missing or extra elements.
3. Fix them, and screenshot again.

Data and wording may differ, because they come from the app. Everything else must match.

### 10.4 Behaviour (each page, create **and** edit)

| Area | Check |
|---|---|
| Fields and validation | Required validation and messages appear as before. The issues banner links focus the right field. |
| Cascades | ITAM ID → ITAM name auto-fills. Data source → attribute enables. ITAM → Application → Entitlement narrows. |
| OLA | Pick, replace and remove a file; drag and drop. |
| Grid editor | Add row, Tab / Enter, Ctrl V paste of several Excel rows, Import CSV, remove a row, per-row application switch, Check states. |
| Conditions | Add (Key / Operator / Value), remove, more than one value. |
| Pickers | Search, tick, untick, All / Selected, expand details, Cancel discards, Add saves, Esc closes. |
| Entitlement | List ↔ Grid preview, Add feature, Privileged, remove a feature. |
| Buttons | Reset / Cancel / Discard changes; Save draft (the draft really saves); Submit for approval (really submits, and a new record appears in its list); Create account. |
| Edit | Every field prefilled; changing a field shows the marker, Was, the Your changes entry and the bar count; Discard all restores; saving works. |
| Progress | The rail counts update live; clicking a progress row scrolls to its section. |

### 10.5 Done means

- Every state in §10.2 reports **0 FAIL** with fonts loaded; each WARN is fixed or explained.
- No look differences remain.
- §10.4 all works.
- Every INVENTORY.md line is present.

---

## 11. How you work

### 11.1 Setup

1. Read this file, `markup.md` and `reference/index.html`. Open three references in the browser.
2. Find the create / edit components, their routes on localhost:8001, how Edit loads a record, and where validation, cascades, pickers and the grid editor live.
3. Write `out/INVENTORY.md`, and take "before" screenshots into `out/before/`.
4. Make sure `fmces-pages.css` and the fonts are loaded, then load `fmces-forms.css` after it.
5. On any create page, add the shell (§2) and run `await lpFormVerify()`. The Stylesheet checks must pass before you continue.

### 11.2 Order

Build each page in **Create then Edit**, in this order:
1. Data profile — it has every common piece.
2. Application.
3. ITAM.
4. Data policy.
5. Entitlement, with the pickers.
6. Features.
7. Actions.
8. Account.

After each page:
- run the self-check on its states until 0 FAIL;
- compare it with the references;
- run its behaviour checks (§10.4);
- save "after" screenshots to `out/after/`;
- add its summary lines to `out/REPORT.md`.

Then move on.

### 11.3 Only stop and ask me if

1. the kit folder is missing;
2. keeping a feature, field or validation is impossible with this design;
3. you need a backend or API change.

For everything else, take the default below, write it in `out/DECISIONS.md`, and keep going.

### 11.4 Defaults

| Topic | Default |
|---|---|
| A field not in §5 | Keep it, in its current section, styled like the others. |
| Edit for a type that has no versions | No version chip; Summary or no lifecycle (as for the account). |
| "Draft saved …" time | Only if the app knows it; otherwise omit `.lp-cf-sv`. |
| Duplicate check in the grid | Only if the app already detects duplicates. |
| Textarea counter | Only if the field has a max length. |
| Grid preview cells | Clickable only if they are today. |
| Library date picker / autocomplete | Keep its logic and restyle it in `fmces-forms-overrides.css`. |
| Wording the app lacks | Use the reference wording. |

---

## 12. Final report (`out/REPORT.md`), then a short summary to me

1. One self-check summary line per state in §10.2, e.g. `FMCES forms verify v1 — profile · create — 72 PASS, 0 FAIL, 0 WARN`.
2. Before / after screenshots per page, in `out/before/` and `out/after/`.
3. INVENTORY.md with every line ticked.
4. DECISIONS.md, including any library parts you kept and how you styled them.
5. The files you changed, and how to run the self-check again.
6. Anything that doesn't match, and why. This should be empty.
