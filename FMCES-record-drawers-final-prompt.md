# FMCES: rebuild every record drawer in the final design, plus the New entitlement pickers

You are working in the FMCES codebase. FMCES is an internal "Markets Operations One" web app. It uses a dark navy theme with a teal accent.

When you click a row in a list page, a **detail drawer** opens on the right. There is a drawer today for each of these eight record types:

| # | Record type | Where its list lives |
|---|---|---|
| 1 | Application | App Management → Applications |
| 2 | ITAM record | App Management → ITAM records |
| 3 | Feature | App Management → Features |
| 4 | Action | App Management → Actions |
| 5 | Data profile | Data Profiles |
| 6 | Data policy | Data Policies |
| 7 | Entitlement | Entitlement Management |
| 8 | Account (user entitlement) | User Entitlements |

Your job:

1. **Replace all eight drawers with one shared drawer component in the new design.** The component configures itself per record type.
2. **Redesign the "Select data policies" and "Select data profiles" drawers** that open from the New entitlement and Edit entitlement pages. Each row gets an expandable details dropdown showing the item's conditions.

I am not sending screenshots. This prompt is the complete specification:
- The stylesheet is in **Appendix A**. Copy it verbatim.
- The markup for every piece is in **Appendix B**.
- The icons are in **Appendix C**.
- Everything else (behaviour, rules, wording, per-type content) is described below.

Use the numbers and words given here exactly. Do not adjust them by eye.

This replaces the drawer design from my earlier prompts. Where an older prompt and this one disagree, this one wins.

---

## How we work together (read this first)

**You may ask me questions at any time. I want you to.** If you are not sure whether something is possible, or how my data maps to a field, stop and ask. Don't guess. I would rather answer five questions than undo a wrong assumption.

**Before you write any code, send me a Step 0 report and wait for my reply.** It has three parts.

**Part 1: map of the current code.**
- The component(s) that render the eight drawers today, and how a drawer is opened.
- What data each drawer receives, and from which API call.
- Where the list rows come from, so that Prev / Next can follow the same order.
- The New entitlement and Edit entitlement pages.
- The current Select data policies / profiles drawers, and the API calls behind them.
- The framework and styling approach (React, Angular, Vue; CSS modules, SCSS, Tailwind…).
- Any global CSS (a UI library, resets, `button` / `table` styles) that could bleed into new markup.

**Part 2: feasibility table.** List every feature below with one status:
- ✅ **possible now** with data the app already has;
- ⚠️ **possible, but needs something** (a field the API doesn't return, a new endpoint, a backend change). Say exactly what;
- ❌ **not possible**, and why.

Cover at least these features:
- lifecycle hero;
- glass tabs;
- version story;
- Compare (version snapshots);
- draft changes shown in place;
- Used by, for each record type and each relationship;
- linked records opening on top;
- Prev / Next;
- quick actions + shortcuts;
- Share summary;
- PDF / CSV / JSON download;
- raw JSON;
- expanded centered view;
- the permissions table;
- account Access remove;
- picker details (conditions, operator, logical indicator, owner, modified date, status, related profiles / policies).

**Part 3: questions.** Everything you are unsure about, numbered, so I can answer in one go.

Do not build anything marked ⚠️ or ❌ until I answer. Build ✅ items only after I reply to Step 0.

**While you build:**
- Work in the phases listed in section 22.
- After each phase, send a short report:
  - what you built;
  - the files you touched;
  - how I can test it;
  - anything that didn't match this spec;
  - new questions.
- If you hit something unclear mid-phase, stop and ask. Do not quietly skip a feature or change the design.

**Never invent data.**
- **[IF DATA EXISTS]** in this prompt means: build it only if the app already has the data, or an existing API returns it. Otherwise follow the fallback written next to it, and tell me.
- My mockups contained a few sample numbers, such as people counts in Used by and sample conditions in the pickers. Production must only show real data.

---

## 0. Rules

**Keep unchanged:**
- Routes.
- API calls and payloads.
- Auth and permissions.
- Existing action handlers: Edit / Continue draft navigation, Compare data source, delete, remove access.
- Any ids, names or data-testids that tests rely on.

Do not add endpoints or change backend code without asking me.

**Do not touch:**
- The app header.
- The workspace tabs.
- The dock.
- The list pages, except that the list must pass its current row order to the drawer for Prev / Next.
- The rest of the New / Edit entitlement forms. Only restyle their Data policies and Data profiles sections (section 18.6).

**Replace** the eight drawers' markup and styles with the new shared component. Delete the old drawer styles only once the new drawer works for all eight types.

**CSS:**
- Copy **Appendix A verbatim** into one new stylesheet, `fmces-drawer.css`, and load it globally.
- Everything is scoped: every class starts with `fd-`, and the design tokens live on `.fd-root`.
- Do not translate it to Tailwind, CSS modules or inline styles. Do not rename classes.
- The only inline styles allowed are the ones in Appendix B:
  - `--i`, `--k`, `--n`, `--c`, `--dw`;
  - the `transform: translateX(...)` on sliding indicators;
  - the flex style on the toolbar wrapper;
  - the two legend dot colours in the Compare summary;
  - `width` on skeleton lines.

**Root wrapper:**
- Render every drawer, picker, scrim and toast inside one `<div class="fd-root">`.
- Ideally portal it to `document.body`, so the drawer covers the full screen height, header included.
- `.fd-root` itself has no size or position. The drawers inside it are `position: fixed`.

**Fonts:** the design needs Manrope weights 300–800 and JetBrains Mono. If the existing link lacks any weight, replace it with:
`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

---

## 1. How to read Appendix B

Appendix B is framework-neutral HTML. Translate it to your template syntax:

| In Appendix B | Means |
|---|---|
| `{name}` | A value you compute. Section names explain each one. |
| `<!-- if cond -->…<!-- /if -->` | Render only when `cond` is true. |
| `<!-- for each x in list -->…<!-- /for -->` | Repeat for each item. |
| `onClick={fn}`, `onKeyDown={fn}`, `aria-x={v}` | Bind the handler or attribute. |
| `class="fd-a {fd-b [fd-c]|fd-d}"` | Static classes, plus a computed part in braces. `[x]` = optional modifier. `a\|b` = one of the alternatives. |
| `<Icon name="search" size="15" bold class="fd-x"/>` | Your small Icon component (below). |
| `ref={dwRef}` | Keep a reference to the element (used for focus). |

**Icon component:**

```html
<svg class="fd-i [fd-b if bold] [extra classes]" width="{size}" height="{size}" viewBox="0 0 24 24" aria-hidden="true"><path d="{path from Appendix C}"></path></svg>
```

- Stroke icons only. The CSS sets fill none, stroke currentColor, width 1.75 (2.25 when bold).
- Where Appendix B says `name="{t.d}"` (or `m.d`, `x.d`, `c.d`, `expD`), the icon is chosen per item. Each section tells you which icon to use.

---

## 2. Colours, type, motion

All colours are tokens on `.fd-root` (Appendix A, first rule). Use only these.

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#0F1A22` | App background behind the scrim |
| `--surface` | `#111E27` | Inputs, segmented controls, table headers |
| `--card` | `#152733` | Cards, open picker rows |
| `--card2` | `#1B3140` | Raised fills, the active segment, counters |
| `--border` | `#23394A` | Card borders |
| `--hair` | `#1E3240` | Hairline dividers |
| `--btn` | `#2B4556` | Button borders, empty-state dashes |
| `--text` | `#E6EEF2` | Main text |
| `--text2` | `#A7B8C4` | Secondary text |
| `--muted` | `#86A0AE` | Labels and meta |
| `--faint` | `#6F8896` | Placeholders, "Not set" |
| `--accent` | `#2DD4BF` | Teal: Live, lifecycle bars, primary button, checkmarks |
| `--accent2` | `#5EEAD4` | Light teal: links, kind label, the "Live" word |
| `--on` | `#062A26` | Text on teal buttons |
| `--draft` | `#F6C177` | Amber: anything draft (stripes, note, Draft word) |
| `--pend` | `#C9B6FF` | Lavender: waiting for approval |
| `--err` | `#F4A3A3` | Removed values, remove-hover |
| `--blue` | `#93C5FD` | Operator chips, informational |

The drawer background is a vertical gradient, `#13232E` → `#101C25` (`--dw-bg`).

**Fonts:**
- Manrope for everything.
- JetBrains Mono (`fd-mono`) for:
  - IDs;
  - user / service names (`g.iam.fmces.002.dev`, `internal-api-call`, `1466717`);
  - field keys (`Booking_Location`);
  - country codes;
  - dates in the version story;
  - CODE_CASE names.

**Key sizes** (the CSS has them all):

| Element | Size / weight |
|---|---|
| Hero word "Live" | 68px / 300 (62px in the expanded view) |
| Record name | 22px / 700 (side) |
| Section titles | 14px / 700 |
| Body text | 13–13.5px |
| Facts numbers | 26px / 300 |

**Motion** (all in the CSS):
- The drawer slides in 40px from the right and fades in over 0.55s, ease `cubic-bezier(.22,1,.36,1)`.
- The lifecycle bars fill left to right, one after another (190ms stagger).
- Tab content fades up 8px.
- Lists stagger in.
- The tab indicator slides with a spring `cubic-bezier(.34,1.36,.64,1)`.
- The draft stripes move continuously.
- In Used by, a small dot travels along each connector.
- Everything respects `prefers-reduced-motion`.

**Replaying entrance animations:** some elements carry a pair of classes, `fd-xxa` / `fd-xxb`. When the trigger in the table happens, swap to the other class of the pair. That replays the animation. Remounting the element with a changing key is equally fine.

| Pair | Swap when |
|---|---|
| `fd-dwa`/`fd-dwb` on the drawer | the record changes |
| `fd-lca`/`fd-lcb` on the hero | the record changes |
| `fd-tpa`/`fd-tpb` on the tab content wrapper | the tab changes |
| `fd-cla`/`fd-clb` on lists | the Features/Actions segment or the Live/Draft/Changes mode changes |
| `fd-csa`/`fd-csb` on compare rows | the compared versions change |
| `fd-ska`/`fd-skb` on the linked-record panel | a linked record opens |

---

## 3. One record model for all eight types

Write a small adapter per record type that turns your existing API response into one shape. The drawer renders only this shape.

If a field doesn't exist for a type, leave it out. Ask me if you're unsure how a field maps.

```ts
type RecordType = 'application' | 'itam' | 'feature' | 'action' | 'profile' | 'policy' | 'entitlement' | 'account';

interface Version {
  v: number;                 // 1, 2, 4…
  status: 'Draft' | 'Pending' | 'Approved' | 'Deleted';   // map your statuses; ask if others exist
  isLive: boolean;           // the current active version ("Current" + "Active" in the old UI)
  date: string;              // ISO; shown as "13 Oct 2026" / "13 Oct"
  by: string;                // user or service id
  snapshot?: Record<string, string | string[] | null>;    // fields of that version, for Compare [IF DATA EXISTS]
}

interface LinkItem { name: string; kind: string; id?: string; extra?: string; open?: { type: RecordType; id: string } }

interface DrawerRecord {
  type: RecordType;
  id: string;                // what the header shows after the kind, e.g. "164"
  idLabel: string;           // "ID", "ITAM", or for accounts "App"
  kindLabel: string;         // "Application", "ITAM record", "Feature", "Action", "Data profile", "Data policy", "Entitlement", "Account"
  name: string;
  initials: string;          // first two letters of the name, upper case ("SAWFL" → "SA"); accounts show a person icon instead
  description: string | null;     // null or "-" → empty state
  summary: string;           // one plain-language sentence (section 17)
  facts: [value: string, label: string][];   // exactly 3 (section 17)
  groups: { title: string; rows: { label: string; value: string | null; mono?: boolean; copy?: boolean }[] }[];
  versions: Version[];       // empty for accounts
  createdOn?: string; createdBy?: string;
  editLabel: string;         // "Edit application", "Edit policy", …
  accountStatus?: 'Active' | 'Inactive';
  main?: MainData;           // per type, section 8
  used?: { up: LinkItem[]; down: LinkItem[]; downLabel: string } | null;   // section 10
}
```

**Date format everywhere:**
- "13 Oct 2026" (day, short month, year).
- Shorter places (version story dates, lifecycle captions) use "13 Oct".

**Country codes:**
- In location / domicile fields, show 2-letter codes next to their English names, e.g. `CN China`.
- Get the names from `new Intl.DisplayNames(['en'], { type: 'region' })`.
- Unknown codes, and values of other fields, show the raw value only.

---

## 4. Side panel: anatomy and behaviour

The drawer opens when you click a list row, exactly as today.

**Panel:**
- Fixed to the right edge, full viewport height (top 0 to bottom 0), so it also covers the app header.
- Width 720px (`--dw: 720px`, max 100% of the viewport). z-index 100.
- Behind it is a full-screen scrim (`fd-fx-scrim`): `rgba(5,10,14,.58)` with a 2px blur, z-index 90.
- Clicking the scrim closes the drawer.
- The list row of the open record stays highlighted, using the app's existing style.

From top to bottom (shell markup in Appendix B, "Side panel shell"):

### 4.1 Toolbar (`fd-tbar`, 54px)

Left side:
- **Prev** (`chevron-up` icon) and **Next** (`chevron-down` icon). They move to the previous / next record in the list's current order (sort, filter and page as shown). Disabled (`fd-dis`) at the ends.
- Position text **"2 of 4"**.

Right side:
- **Actions** button: `search` icon + "Actions" + a `/` key cap. Opens Quick actions (section 13).
- **Share** button (`share` icon). Toggles the Share popover (section 14).
- A thin separator.
- The **expand** icon button (`expand` icon). Opens the centered view (section 15).
- An **Esc** chip that closes the drawer.
- A **close** icon button.

**Prev / Next keeps the current tab**: if you are on Access rule, the next record opens on Access rule. Opening a record from the list starts on Overview.

### 4.2 Header (`fd-dh`)

Left:
- A 44px rounded square avatar (`fd-mg`) with the record's initials in light teal. Accounts show the `user` icon instead.

Then:
- The **kind label** in light teal (e.g. "Data policy"), followed by a hairline divider and the **ID** in mono, e.g. `ID 164` (`fd-kid`). Accounts show `App FMCES`.
- The **record name** below, 22px bold. Long names wrap.

Right:
- The primary action `fd-be` (teal outline, fills teal on hover), labelled `{editLabel}`. When a draft exists the label becomes **"Continue draft V4"** and it opens that draft.
- The **•••** button (section 12).

The menu opens from this header, never from inside the scrolling body, so it is never clipped.

### 4.3 Scrolling body (`fd-db`)

The body is the only part that scrolls. It contains, in order:
1. The **lifecycle hero** (section 5).
2. An empty marker `<div data-sec="tabs">`.
3. The **sticky tab strip** (`fd-tstrip`). It sticks flush to the top of the body (`top: -30px` offsets the body's 30px padding; it's in the CSS) and has an opaque blurred background so content never shows above or behind it.
4. The **tab content** (`fd-tpa`/`fd-tpb` wrapper).

Changing tab never jumps the page. If the body is scrolled below the tab strip, scroll back so the strip sits at the top. Otherwise keep the scroll position.

### 4.4 Overlays inside the panel

These render inside the `<aside>`, after the body:
- the Compare sheet;
- the Share popover;
- the Raw JSON sheet;
- the linked-record panel;
- the Quick actions palette;
- the shortcuts dialog.

Layering is in section 16.

---

## 5. The lifecycle hero

This is the part I like most. Build it exactly.

**Top row:**
- A big word, `{stage}`, 68px weight 300.
- Next to it, `{heroB}` in bold over `{heroS}`.

**Below it**, 4 equal bars with labels: **Draft · Pending approval · Approved · Active**.
- Each bar is 8px tall, 6px gap between bars.
- Each has a bold label and a small caption.
- Done bars fill teal (gradient `#2DD4BF` → `#5EEAD4`), one after another.
- The current stage's bar glows.

**Optional note line** under the bars (`fd-hero-dr`): a dot, a sentence and a link on the right.

**Rules.** Compute `live` (the version with `isLive`), `draft` (a non-deleted Draft version) and `pending` (a Pending version) [IF DATA EXISTS: pending].

| Situation | `stage` word (class) | `heroB` / `heroS` | Bars (`fd-lc-s` + state) | Note |
|---|---|---|---|---|
| live, no draft / pending | **Live** (`fd-hero-w`, light teal with glow) | "Version N" / "since 13 Oct 2026" | all 4 `fd-is-done`, last one `fd-is-now` | none |
| live + draft | **Live** | same | as above; Draft bar also gets `fd-has-draft` (moving amber stripes over it) | amber: "Version **4** is being drafted by `g.fmces.001.dev`." + link **See what changes** (opens Compare live → draft) |
| live + pending | **Live** | same | as above; Pending bar gets `fd-has-pend` (lavender stripes) | add `fd-pend` to the note: "Version 4 is waiting for approval. Sent by `{by}`." + link **See what changes** |
| no live, draft only | **Draft** (`fd-hero-w fd-dr`, amber) | "Version 1" / "not live yet" | Draft `fd-is-now fd-has-draft`; the other three `fd-is-todo` (grey, no fill) | amber: "Version 1 is a draft by `1466717`. Nothing is live yet." + link **Continue draft** (same as the edit button) |
| no live, pending | **Pending** (`fd-hero-w fd-pd`, lavender) | "Version N" / "not live yet" | Draft `fd-is-done`; Pending `fd-is-now fd-has-pend`; rest `fd-is-todo` | lavender `fd-pend` note: "Version N is waiting for approval. Nothing is live yet." |
| account | **Active** (or **Inactive** with `fd-off`) | "User account" / "in FMCES" (its applications) | no bars (`hasBars` false) | none |

**Bar captions:**

| Bar | Caption |
|---|---|
| Draft | "Created 29 Sep" (record created date), else "Started 14 Oct" (draft date when nothing is live), else "Started" |
| Pending approval | "Checker review" ("Waiting on checker" when pending) |
| Approved | "13 Oct by internal-api-call" (live version date + by), or "Not yet" |
| Active | "Since 13 Oct 2026", or "Not live yet" |

In the note, the person / service id is wrapped in `fd-mono`.

---

## 6. Tabs (glass segmented control)

The tab strip `fd-ptb` is a rounded glassy bar with equal-width tabs (`--n` = number of tabs). A raised indicator `fd-ind` slides under the current tab (`translateX(index × 100%)`). Counts sit in small mono text after the label.

Tabs per type:

| Type | Tabs |
|---|---|
| Application | Overview · **Components** `62` · History `1` · Used by |
| ITAM record | Overview · **Applications** `1` · History · Used by |
| Feature | Overview · History · Used by |
| Action | Overview · History · Used by |
| Data profile | Overview · **Access rule** `1` · History · Used by |
| Data policy | Overview · **Access rule** `1` · History · Used by |
| Entitlement | Overview · **Permissions** `9` · History · Used by |
| Account | Overview · **Access** `1` |

**Counts:**

| Tab | Count |
|---|---|
| Components | features + actions |
| Applications | applications |
| Access rule | conditions |
| Permissions | features in the table |
| Access | entitlements |
| History | versions that are not Deleted |

**Rules:**
- Hide **Used by** when no relationship data exists for that type (section 10). Tell me which types end up without it.
- When a data profile or policy has both a live version and a draft that changes its rule, add `fd-chg` to the Access rule tab. That draws a small amber dot after the count.

Tab buttons use `role="tab"` and `aria-selected`.

---

## 7. Overview tab

The content sits in `fd-ov` (a plain wrapper with no styles of its own). It is one column in both views.

### 7.1 About (`fd-sec`)

- **Description** (`fd-ab-d`), 17px.
  - If longer than 160 characters, clamp it to 3 lines (`fd-clamp`) with a **Read more / Show less** link (`fd-ab-more`).
  - If empty or "-": show `fd-ab-none`, "No description yet" (italic, faint) with an **Add one** link that runs the edit action.
- **Summary sentence** (`fd-ab-p`). Section 17 gives one template per type.
- **Three facts** (`fd-facts`): big light numbers with small labels, separated by hairlines. Section 17 lists them per type.

### 7.2 Details (`fd-sec`)

- Groups (`fd-dg`), each with a small title, and a 2-column definition list (`fd-dl`).
- Each row: a label on top, the value below.
- Empty values show **"Not set"** in italic faint (`fd-unset`).
- IDs and user / service names are `fd-mono`. Those marked copy get a copy button that appears on row hover. It copies the value and shows the toast "Copied “{value}”".

Section 17 has the exact groups per type.

---

## 8. The main tab, per type

### 8.1 Application → Components

- A **segmented control** `fd-seg2`: Features `N` | Actions `M`. Plus a **filter** input with placeholder "Filter 10 features" / "Filter 52 actions".
- The list (`fd-cl`) is two columns of rows, 42px tall, each with an icon (`box` for features, `zap` for actions). Names containing `_` use mono.
- If the API returns only part of the actions, show **"Show all 52 actions"** (`fd-more`) to load the rest [IF DATA EXISTS: paging]. Otherwise list them all.

**Empty states** (`fd-empty`):

| Situation | Text |
|---|---|
| Filter matches nothing | "Nothing matches “…”." |
| No features | "No features yet. Add them from Edit application." |
| No actions | "No actions yet. Add them from Edit application." |

### 8.2 Data profile / Data policy → Access rule

**Normal view:**
- Section header "Access rule" + count chip ("1 condition").
- One rule card per condition (`fd-rule`): "Rows where **Booking_Location** is any of", then value chips (`fd-val`). Each chip shows the code in a small teal box, then the country name.

**Draft changes shown in place.** This applies only when there is a live version **and** a draft whose rule differs. The rule card then shows the `fd-il` block instead:
- A 3-way segmented control `fd-segn` (`--n: 3`): **Live V1 · Draft V4 · Changes**. Changes is the default.
- A **Full compare** link on the right. It opens Compare live → draft.
- The rule card shows:

  | Mode | Values shown |
  |---|---|
  | Live | the live values |
  | Draft | the draft values; values new in the draft get `fd-add` |
  | Changes | live values, removed ones marked `fd-rem` + tag "Removed"; then added values marked `fd-add` + tag "New in V4" |

  - `fd-add` = dashed teal border, teal code box.
  - `fd-rem` = red strikethrough.
- A lead line under the chips (`fd-il-lead`):

  | Mode | Lead line |
  |---|---|
  | Live | "What people see today." |
  | Draft | "What people will see if V4 is approved." |
  | Changes | amber, `fd-dif`: "Draft V4 adds Japan." / "…removes Hong Kong." / "…adds Japan and removes Hong Kong." / "…changes nothing here." |

- A caption: "Draft by `g.fmces.001.dev`, 29 Sep. Changes in the draft appear here as they're made."
- With several conditions, repeat the card per condition.

**Profiles used** (policies only): a section "Profiles used" + count. Each row is a `fd-pr` button with the `layers` icon, name and `#id`. When it can open, it shows `chevron-right` and opens the profile as a linked record (section 11).

### 8.3 Entitlement → Permissions

**I like the current permissions table. Keep its layout and restyle it.**

- Header: "Permissions" + chip "9 features", and on the right the muted total "15 permissions".
- The table `fd-pmx`, `--c` = number of action columns:
  - **Columns**: Feature, plus one column per action type that appears in this entitlement, in the order **View, Edit, Approve, Download**. Only include a column if at least one feature grants that action.
  - **Rows**: one per feature, in the API's order.
  - A granted cell shows `fd-pm-on`: a 24px rounded square, teal tint, with a teal check. A missing cell shows `fd-pm-off`: a 5px grey dot.
  - Each cell has `aria-label`, e.g. "Edit allowed" / "Edit not allowed".
  - Rows highlight on hover.
- Below, two sections:
  - **Data policies** `N`
  - **Data profiles** `N`
- Each lists the assigned ones as `fd-pr` rows: name, `#id`, `chevron-right`. Clicking a row opens it as a linked record.
- When empty, show the dashed `fd-dash-e`: "No data policies assigned" (`shield` icon) / "No data profiles configured" (`layers` icon).

### 8.4 ITAM record → Applications

- Section "Applications" + count.
- Each row: `grid` icon, app name, `ITAM 51824` (mono), a status pill (`fd-pill fd-ok` Approved / `fd-draft` Draft), `chevron-right`.
- Clicking a row opens the application as a linked record.

### 8.5 Account → Access

- A teal-tinted summary bar `fd-acc-sum`, with the `key` icon: "**1 entitlement** in **1 application**".
- Then one group per application. The app name sits in a small mono chip `fd-app-chip`.
- Each entitlement row (`fd-ent-row`):
  - the `key` icon, the entitlement name, `#100` (mono) on the right;
  - a remove button `fd-rm`: an `close` icon that turns red on hover. It uses the existing remove-access flow, with whatever confirmation the app already has.
- Clicking the entitlement name opens it as a linked record [IF the entitlement can be loaded by id].

---

## 9. History tab (version story) and Compare

### 9.1 Version story (`fd-sy`)

- Header: "Version history" + count of all versions (`fd-ct`).
- On the right, a **Compare** link (`compare` icon). It is disabled (`fd-dis`) when there is only one version.

Events, newest first. Each row has three columns:
- the short date (mono, muted, right-aligned);
- a dot on a vertical rail;
- the title with an optional pill, and "by" in mono on the next line.

| Event | Row class | Title | Pill |
|---|---|---|---|
| Draft | `fd-is-draft` (amber dot) | "Version 4 draft started" | `fd-pill fd-draft` "Draft" |
| Pending [IF DATA EXISTS] | `fd-is-pend` | "Version 4 sent for approval" | `fd-pill fd-pend` "Waiting" |
| Deleted (newest first) | `fd-is-del` (muted) | "Version 3 draft deleted" | `fd-pill fd-del` "Deleted" |
| Live | `fd-is-ok fd-is-live` (solid teal dot with halo) | "Version 1 approved and made live" | `fd-pill fd-live` "Live" |
| Older approved, replaced | `fd-is-ok` (default grey dot; the class is only a hook) | "Version 2 approved" | `fd-pill fd-del` "Replaced" |
| Created | `fd-is-start` | "Created" | none |

- **Compare shortcut:** hovering a row that is a non-live version shows **"Compare with V1"** (`fd-tl-x`) on the right. It opens Compare from the live version to that one. Only when there are ≥2 versions and a live one.
- **Single version:** add `fd-one` to the list and show "Only one version so far. Compare becomes available after the next change."

### 9.2 Compare sheet

The sheet (`fd-cs-wrap` + `fd-cs`) slides in over the drawer from the right, with a dim scrim inside the drawer. It contains:

- **Header:** a back button, "Compare versions" in small teal, the record name, and a close button.
- **Pickers:**
  - **From** (red dot) and **To** (teal dot) rows of version pills (`fd-vpb`). Each pill shows `V1` with a small subtitle: Live / Draft / Deleted / Approved.
  - The selected From pill gets `fd-on-from`, the selected To pill `fd-on-to`.
  - The version picked on the other side is greyed out (`fd-off`), so you can't compare a version to itself.
  - A round **swap** button between the two rows. It rotates on hover and exchanges From and To.
- **Summary:** "**2 changes** from V1 to V4, 4 unchanged", plus a legend: Added (teal), Removed (red).
- **One row per changed field:**
  - Scalar values: old value (red, struck through) → arrow → new value (teal).
  - Lists: token chips. "+ JP" added (teal), "− HK" removed (red, struck through), unchanged neutral.
- **"Show 4 unchanged fields"** expander (`fd-cs-same` + `fd-dx` grid-rows animation): a 2-column list of the unchanged fields.
- If nothing differs: "These two versions are identical."

**Data:** use the snapshots your existing Compare feature already loads. Diff field by field; array fields diff as token sets. [IF DATA EXISTS]. If version snapshots aren't available, tell me in Step 0.

---

## 10. Used by tab

This tab shows what the record is built on and what depends on it.

**Flow strip** (`fd-uf`): three cards joined by arrows (`fd-uf-a`). A small light-teal dot travels along each arrow.
1. **Built on**: the first upstream item: name, kind. If the only upstream item is the creator (for imported records), the label becomes "Created by", the value is mono, and the kind reads "API import".
2. **This {type}**: highlighted teal card (`fd-me`) with the record's name and kind.
3. **Used by**: the count ("2 entitlements", "1 policy", "3 records") and the kind of the items.

**Impact line** (`fd-uf-imp`):
- With dependants: amber, "Changing this policy affects 2 entitlements."
- With none: add `fd-ok` (teal), "Nothing depends on this {type} yet."

**Then two lists:**
- "Built on" (or "Created by"), and "Used by", using `fd-pr` rows.
- Each row: icon, name, kind (muted), id (mono), extra text on the right (e.g. the granted actions "View, Download"), and `chevron-right` when it can be opened.
- An empty Used by list shows the dashed "Nothing uses this yet".

**Clicks:**
- A row opens its record as a linked record (section 11).
- The "Built on" card opens the first upstream item.
- The "Used by" card opens the single item if there is exactly one. Otherwise it scrolls to the list.

**Relationships per type** [IF DATA EXISTS: each one separately; show only the ones the app can load]:

| Type | Built on | Used by |
|---|---|---|
| Application | its ITAM record | entitlements of this application |
| ITAM record | Created by (creator) | applications that use this ITAM record |
| Feature | Created by | applications that include it, and entitlements that grant permissions on it (extra text = the granted actions) |
| Action | Created by | applications that include it |
| Data profile | its condition fields (kind "Field") | data policies that use the profile |
| Data policy | the profiles it applies to | entitlements that use the policy |
| Entitlement | its application | accounts that hold it (show only the count if names can't be listed) |
| Account | (no tab) | (no tab) |

**Never show made-up counts.** If a type has no loadable relationship, hide the tab for that type and tell me.

---

## 11. Linked records (open on top)

Clicking a linked record must not lose your place. Examples:
- a profile under Profiles used;
- an application in an ITAM record;
- any openable row in Used by;
- an assigned data policy in an entitlement;
- a "Linked" command in Quick actions.

The linked record opens **on top** of the current drawer:
- **The panel** (`fd-stk`) slides in from the right inside the drawer. It fills the drawer except a **64px strip on the left** (`fd-stk-peek`).
- **The peek strip** shows a vertical pill "← {parent name}" over the dimmed parent.
- **The parent** (header, toolbar and body) scales to 0.965, shifts 10px left and dims. Add `fd-has-stk` to the drawer.
- **Header:** a back button; a breadcrumb "{parent name} › **{child name}**" (the parent part is a button); and **Open full record ↗** on the right.
- **Body:**
  - The identity block: avatar, kind + ID, name, status line ("Version 1 live since 5 Oct 2026" teal / "Version 1 in draft" amber).
  - Four thin bars: all teal when live (`fd-stk-bars`). When draft-only, `fd-stk-bars fd-dr`: first bar amber-striped, the rest grey.
  - The summary sentence and the 3 facts.
  - The rule card, for profiles and policies.
  - The first 4 detail rows.
  - "Linked to: {parent} · You came from here" (clicking it goes back).

**Behaviour:**
- Back, the peek strip, the breadcrumb or Esc closes the panel and returns to exactly where you were: same tab, same scroll.
- **Open full record** makes the linked record the drawer's main record, keeping the tab where possible.
- Only one level deep: links inside the linked panel are plain text.
- Load the linked record with the API that type's drawer already uses [IF DATA EXISTS]. While loading, show skeleton lines (`fd-skel`).

---

## 12. The ••• menu

The menu is anchored under the ••• button in the header: 264px wide, a pop-in animation, z-index 60. A transparent full-screen button (`fd-mscrim`, z-index 55) sits behind it; clicking anywhere outside closes the menu.

Items:
- **Group "This {type}":**
  - Copy name (`copy` icon).
  - Copy {ID label} (`hash` icon, with the value shown on the right).
  - Copy all details (`doc` icon). Copies the summary text from section 14.
- **Separator.**
- **Group "Versions"** (only when the record has versions):
  - Compare versions (`compare` icon). Shows the count on the right. With one version it says "Only 1", is styled `fd-dis`, and clicking it shows the toast "Only one version so far, so there is nothing to compare yet".
  - Version history (`history` icon, count). Switches to the History tab and smooth-scrolls the tab strip to the top.

Copy actions show a toast (section 16).

---

## 13. Quick actions and keyboard shortcuts

**Quick actions palette** (`fd-pal`), opened by the Actions button or **/**:
- It appears 84px from the top of the drawer, centred, 560px wide, over a blurred scrim.
- **Search input:** autofocused, placeholder "Type a command…", with an Esc key cap.
- **Commands are grouped:**

  | Group | Commands |
  |---|---|
  | This {type} | **Edit** / Continue draft V4 (key E); Compare versions (C, only with ≥2 versions); Version history (H, only with versions); Copy summary; Copy {ID label} {id}; View raw JSON |
  | Linked | "Open profile Version Testing", "Open application SAWFL", … (every openable related record) |
  | Go to | each tab with its number key 1–4 |
  | Move | "Next: {next record name}" (J); "Previous: {name}" (K); Expand to centered view / Back to side panel (F); Close (Esc) |

- Use the matching icons from Appendix C.
- **Typing** filters by command text and group name.
- **↑ / ↓** move the highlight. It wraps, and the highlighted row gets `fd-on` with a teal inset bar on its left edge.
- **Enter** runs the command and closes the palette. Hovering a row highlights it.
- **Esc** closes only the palette.
- Empty result: "No command matches “…”."
- **Footer:** "↑↓ Move", "↵ Run", and a **? All shortcuts** link.

**Shortcuts.** They work when focus is inside the drawer, you are not typing in an input / textarea, no modifier key is held, and no menu / compare / palette / raw JSON is open:

| Key | Action |
|---|---|
| `/` | Quick actions |
| `?` | Shortcuts dialog (toggle) |
| `J` / `K` | Next / previous record |
| `F` | Expand / back to side panel |
| `E` | Edit / continue draft |
| `C` | Compare (≥2 versions) |
| `H` | Version history |
| `1`–`4` | Switch tab |
| `Esc` | Close the top layer (section 16) |

**Shortcuts dialog** (`fd-dlg fd-keys`): three columns, **Move around**, **Act**, **Tabs** (the tabs list uses the record's real tab names). Each row is a label plus a key cap. Footer: "Shortcuts work while the drawer is open and you're not typing in a field."

**Listen on the drawer's `<aside>` (`onKeyDown`), not on `window`.**

---

## 14. Share and raw JSON

**Share popover** (`fd-shp`):
- Opened by the Share button. It sits under the toolbar on the right: top 60px, right 16px, 400px wide. In the expanded view: top 84px, right 24px.
- A transparent scrim behind it closes it on outside click.

Contents:
1. Title **"Share {name}"**.
2. **Summary to paste**, with a **Copy** link and a mono preview box (`fd-shp-sum`, pre-wrap). Its lines:
   ```
   SAWFL — Application (ITAM 51824)
   Status: approved, version 1 live since 13 Oct 2026      (add "; version 4 in draft" when there is a draft; "Status: draft, not live yet" when nothing is live; accounts: "Status: active")
   Sales Workflow Tool                                     (description, cut to 90 characters; skip if empty)
   10 features, 52 actions                                 (profiles/policies: "Rule: Booking_Location is KR or SG")
   ```
3. **Download**, three tiles:
   - **PDF**, "One-page summary";
   - **CSV**, "Fields and versions";
   - **JSON**, "Full record".

   CSV and JSON can be generated in the browser from the loaded record. **For PDF, ask me.** If the app has no export, the options are the browser's print-to-PDF of a print view, or hiding the tile.
4. **View raw JSON** (dashed button, `code` icon). Opens the raw JSON sheet.

**Raw JSON sheet** (`fd-cs fd-raw`): the same slide-in sheet as Compare.
- Header: "Raw JSON · N lines", the name, a **Copy JSON** button.
- The JSON is pretty-printed with 2-space indent. Each line shows a line number and syntax-coloured tokens:

  | Token | Class | Colour |
  |---|---|---|
  | Keys | `fd-jk` | `#93C5FD` |
  | Strings | `fd-js` | `#5EEAD4` |
  | Numbers | `fd-jn` | `#F6C177` |
  | true / false / null | `fd-jb` | `#C9B6FF` |
  | Punctuation | `fd-jp` | `#6F8896` |
  | Whitespace | `fd-jw` | (none) |

- Show the record as your API returned it, with no secrets. If the response contains anything sensitive, ask me before showing it.

---

## 15. Expanded (centered) view

The **expand** icon (toolbar) or **F** switches the open drawer to the centered view. The collapse icon brings back the side panel. Both views show the same record and tab.

**Sheet:**
- `fd-dw fd-dw-sheet fd-fx`: fixed, top 44px, bottom 44px, horizontally centred.
- Width `min(1120px, 100% − 64px)`, radius 22px.
- Its own scrim (`fd-sh-scrim`, darker, 5px blur).

**Header:**
- The same identity block: avatar, kind + `ID 164`, name.
- On the right:
  - a compact quick-actions button (`fd-kb fd-ic`: search icon + `/` key cap, no text);
  - Share;
  - a separator;
  - the edit button;
  - •••;
  - the **collapse** icon;
  - close.

**Hero band** (`fd-xh`):
- The lifecycle hero laid out horizontally across the full sheet width: the big word (62px) and version on the left, the 4 bars on the right.
- The note spans the full width below.
- A faint teal top gradient and a bottom hairline.

**Body:** a grid of a 220px **left menu** (`fd-sh-nav`) and the main column.
- **Left menu:**
  - The tabs as a vertical list: icon, label, count on the right; the active tab has a teal inset bar.
  - At the bottom, **"At a glance"** (`fd-qf`), three rows:
    - ID (mono, copy on hover);
    - Owner / Application / Created by (whichever exists first);
    - "Last change": "29 Sep · V4 draft", or for accounts "Status Active".
- **Main column:** the same tab content as the side panel, with 26px / 36px padding. Overview stays one column (About, then Details), exactly as in the side panel.

**Overlays inside the sheet:**
- Compare, raw JSON, linked record, palette and shortcuts render inside the sheet with rounded corners.
- Compare and raw JSON open as 640px panels from the right edge of the sheet.

At ≤900px viewport width the sheet's body stacks to one column.

---

## 16. Focus, Esc, layering, toasts

**Focus:**
- The `<aside>` has `tabindex="-1"` and gets focus when the drawer opens (no scroll jump).
- After any layer closes, if focus is no longer inside the drawer, move it back to the `<aside>`. That way Esc and shortcuts keep working.
- The palette input takes focus while the palette is open.

**Esc closes one layer at a time**, in this order:
menu → shortcuts dialog → palette → raw JSON → share → linked record → compare → (picker) → the drawer.

**Layering:**

| Layer | z-index |
|---|---|
| Scrim | 90 |
| Drawer | 100 |
| Inside the drawer: toolbar | 8 |
| Header | 6 |
| Sticky tabs | 5 |
| Body | 1 |
| Compare / raw sheets | 40 |
| Linked panel | 45 |
| Menu scrim / menu | 55 / 60 |
| Share | 62 |
| Palette / dialogs | 70 |
| Toast | 120 (fixed) |

**Toasts** (`fd-toast`):
- A pill at the bottom with a teal check icon. Add `fd-show` to show it; it hides after 2.4s.
- Add `fd-in-dw` to centre it under the side panel, or `fd-in-pk` under the picker. In the expanded view, centre it on the screen.
- Wording:
  - "Copied “164”"
  - "Copied ID 164"
  - "Copied summary"
  - "Copied JSON"
  - "Only one version so far, so there is nothing to compare yet"
  - "2 policies added to the new entitlement"

---

## 17. Per-type reference

Real records from my app, so you can check your work. **Summary templates:** fill in real values; join lists with commas and "and" / "or" as shown.

### Application
- **Example:** SAWFL, `ITAM 51824`.
  - V1 approved and live since 13 Oct 2026, by `internal-api-call`.
  - Description "Sales Workflow Tool".
  - 10 features, 52 actions; CR number and onboard date not set.
  - Opens on **Live**, all bars teal.
- **Example (draft only):** AppTest Automation, `ITAM 55508`.
  - V1 Draft (not active) since 14 Oct 2026 by `1466717`.
  - Description "Created by Automation Script".
  - 0 features, 0 actions; CR number `CHG4523876`; onboard date 19 Sep 2026.
  - Opens on the amber **Draft** hero, Components shows the empty states, edit reads "Continue draft V1".
- **Summary:** "A … with {F} features and {A} actions. Version {v} has been live since {date}." Draft-only: "A new application with no features or actions yet. Version 1 is still a draft, so nothing is live." Adjust the wording when counts aren't zero.
- **Facts:** Features · Actions · Live version (or "Draft").
- **Details groups:**
  - Identity: ITAM (mono, copy) · Total features · Total actions.
  - Change control: CR number (mono, copy) · Onboard date.
  - Last change: Changed by (mono, copy) · Changed on.

### ITAM record
- **Example:** SAWFL, `ID 176`, ITAM ID 51824, V1 live 13 Oct 2026 by `internal-api-call`, Applications: SAWFL (Approved).
- **Summary:** "The ITAM entry for {name} (ITAM ID {itam}). It backs {n} application(s), and version {v} has been live since {date}."
- **Facts:** Application(s) count · ITAM ID · Live version.
- **Details:**
  - Identity: ITAM ID (mono, copy) · SNOW group · Support DL.
  - Last change: Created by · Created · Updated by.
- **No description** → empty state.

### Feature
- **Example:** Reports, `ID 384`, V1 live 13 Oct 2026, created by `internal-api-call`, no description.
- **Summary:** "Part of {applications}. {n} entitlements grant permissions on it." [IF DATA EXISTS]. Otherwise "Version {v} has been live since {date}."
- **Facts:** Applications · Entitlements · Live version. If those counts aren't available: Feature ID · Created · Live version.
- **Details:**
  - Identity: Feature ID (mono, copy).
  - Last change: Created by (mono, copy) · Created · Last modified.

### Action
- **Example:** Edit, `ID 424`, V1 live 13 Oct 2026, created by `internal-api-call`, no description.
- **Summary:** "An action added on {created}. Version {v} is live."
- **Facts:** Live version · Action ID · Added (short date).
- **Details:**
  - Identity: Action ID.
  - Last change: Created by · Created · Last modified.

### Data profile
- **Example:** Version Testing, `ID 96`.
  - V1 live since 5 Oct 2026 by `1466717`; V2 deleted.
  - Rule: Country Domicile in BR, BW.
  - Owner `g.iam.fmces.002.dev`; created 28 Sep 2026.
  - Description "Version Testing".
- **Summary:** "Covers only the rows where {Field} is {names joined with "or"}." For multiple conditions, join them with "and". For Country Domicile you may phrase it "where the client is domiciled in Brazil or Botswana".
- **Facts:** Conditions · values count (label "Countries" for location fields, else "Values") · Live version.
- **Details:**
  - Ownership: Profile owner · Created by · Updated by · Version.
  - Settings: Active · Global profile.
  - Dates: Created · Last modified.

### Data policy
- **Example A:** Entitlement Policy, `ID 143`.
  - V1 live since 1 Oct 2026 by `2023504`.
  - Rule: Booking_Location in AG, CN; applies to profile Entitlement Profile #43.
  - Owner `1466717`; created 7 Sep 2026; description "Entitlement Policy Testing".
- **Example B:** Policy Versioning Testing, `ID 164`.
  - V1 live (Booking_Location KR, SG).
  - **V4 draft** by `g.fmces.001.dev`; V3 and V2 deleted.
  - Profile Version Testing #96.
  - Opens with the amber note, the striped Draft bar, and the Access rule tab dot.
- **Summary:** "Limits data to rows where {Field} is {names joined with "or"}, through the profile {profile names}." For booking location: "Limits data to bookings in Antigua and Barbuda or China, through the profile Entitlement Profile."
- **Facts:** Conditions · Profiles · Live version.
- **Details:**
  - Ownership: Policy owner · Created by · Updated by · Version.
  - Settings: Active · Global policy.
  - Dates: Created · Last modified.

### Entitlement
- **Example:** RATAN_ENTITLEMENT_RULE Application Admin, `ID 168`, Approved.
  - Application RATAN_ENTITLEMENT_RULE; version "-" (shows "Not set").
  - Created 24 Sep 2026; last modified 14 Oct 2026; created / updated by `internal-api-call`.
  - No data policies, no data profiles.
  - Permissions:

    | Feature | Granted |
    |---|---|
    | Entitlement | Edit, Approve |
    | Action | Edit, Approve |
    | Data Policy | View, Edit, Approve |
    | Data Profile | Edit, Approve |
    | Feature | Edit, Approve |
    | Itam | View |
    | API | View |
    | Reports | View |
    | Application | Edit |

    → columns View · Edit · Approve, 9 features, 15 permissions.
- **Example 2:** IAM Admin `ID 94` (FMCES), with View / Edit / Approve / Download columns.
- **Summary:** "Gives {P} permissions across {F} features in {application}." Use a better sentence if the description says something important. For example, IAM Admin: "A super-user role: it can add, change or remove any user's entitlements in any application, with no maker–checker step." When unsure, use the template.
- **Facts:** Features · Permissions · Data policies.
- **Details:**
  - Ownership: Application (mono, copy) · Version · Created by · Updated by.
  - Dates: Created · Last modified.

### Account (user entitlement)
- **Example:** `8216018`, Active, app FMCES, type User, roles `FMCES:CES Application View`, one entitlement: CES Application View #100 in FMCES.
- **Hero:** "Active" (or "Inactive" with `fd-off`); "User account" / "in FMCES". No bars, no note.
- **Tabs:** Overview · Access.
- **Summary:** "A user account with {n} entitlement(s), in {applications}."
- **Facts:** Entitlements · Applications · Type.
- **Details:** Account: Owner (mono, copy) · Type · Status · Roles (mono, copy).
- **Header:** the ID part reads `App FMCES`.

---

## 18. New entitlement: Select data policies / Select data profiles

Today, on the New entitlement and Edit entitlement pages, "Add data policies" and "Add data profiles" open a right-hand picker. It has:
- a search box ("Search by name, ID or owner");
- **All 9 / Selected 0** tabs;
- rows with a checkbox, name, "Owner 1466717" and `#43`.

The old design had an expandable dropdown per row with the item's details:
- ID, owner, date, status;
- a **Policy Conditions** table: Key, Operator, Values, Logical indicator.

Rebuild the picker in the new style **and bring those details back as an expandable dropdown per row**. Markup: Appendix B, "Picker".

### 18.1 Panel
- Same side-panel shell: `fd-dw fd-dw-final fd-pk`, width 620px (`--dw: 620px`), full height, scrim, slide-in.
- **Header** (`fd-pk-h`): small teal "New entitlement" (or "Edit entitlement"), the title **"Select data policies"** / **"Select data profiles"** (22px bold), an **Esc** chip and a close button.
- **Tools row** (`fd-pk-tools`):
  - the search field (filters by name, ID or owner as you type);
  - a 2-way segmented control (`fd-segn fd-pk-seg`, `--n: 2`): **All `N`** | **Selected `M`**.

### 18.2 Rows (`fd-pki`)

A row has two separate buttons:
- **The tick box** (`fd-ckb`): 22px rounded square; `role="checkbox"`, `aria-checked`. It toggles selection only.
- **The row body** (`fd-pki-m`, `aria-expanded`). It toggles the details. Its contents:
  - the name (bold);
  - a sub-line: owner (mono) · a dot · the first condition in mono, e.g. `Booking_Location in CN, IN`, plus " +1" when there are more conditions;
  - `#150` (mono) on the right;
  - a chevron that turns teal and rotates when open.

Row states:
- Selected rows get `fd-on`: teal border, and a teal tint when closed.
- Open rows get `fd-open`: card background and border.
- Several rows can be open at once.

### 18.3 Details dropdown

The details expand with the `fd-dx` grid-rows animation. Contents:

1. **Key-value chips** (`fd-kvs` / `fd-kv`):
   - ID;
   - Owner (mono);
   - Modified (date, when known);
   - status: `fd-kv fd-ok` with a dot, "Active"; or `fd-kv fd-inact`, "Inactive".
2. **Note** (`fd-pkd-note`, amber), when the item has a draft or pending version, e.g. "Version 4 is in draft". [IF DATA EXISTS]
3. **"Conditions" table** (`fd-ctt`):
   - columns **Field | Operator | Values | Logic**, one row per condition;
   - Field: mono;
   - Operator: a blue mono chip (`fd-op`) with the raw operator, e.g. `IN`, `NOT IN`;
   - Values: chips (`fd-vc`), code + country name;
   - Logic: the logical indicator (AND / OR), or "—" when empty.

   These are the fields the old design showed: Key, Operator, Values, Logical indicator.
4. **Related records:**
   - for policies, **"Applies to profiles"**;
   - for profiles, **"Used by policies"** [IF DATA EXISTS];
   - shown as chips (`fd-rc`): `layers` icon, name, `#id`.

If the details need a separate request, load them the first time a row opens. Show two skeleton lines (`fd-skel`) while loading. If loading fails, show an error box (`fd-errbox`) with **Retry**.

### 18.4 Footer (`fd-pk-f`)

- **Left:**
  - with a selection: the selected items as teal chips (`fd-sc2`) with a small × to untick them. Show the first 3, then a "+N" chip;
  - with nothing ticked: "Tick the ones this entitlement should use".
- **Right:**
  - **Cancel**;
  - the primary button **"Add 2 policies"** / **"Add 1 profile"**. With nothing ticked it reads "Add policies" and is disabled (`fd-dis`), unless items were added before (then clearing them is allowed).

### 18.5 Behaviour

- **Opening:** the picker opens with the entitlement's current selection pre-ticked.
- **Selected tab:** shows only ticked items. Empty: "Nothing selected yet. Tick a policy in All."
- **No search matches:** "No policies match “…”."
- **Cancel, Esc or the scrim:** discard changes.
- **Add:** saves the selection to the form, closes the picker, and shows the toast "2 policies added to the new entitlement".
- **Data:** keep the existing picker API.

### 18.6 On the New / Edit entitlement page

Restyle only the Data policies and Data profiles sections (Appendix B, "New entitlement sections"). Each is a card (`fd-cf-card`) with:
- a teal icon tile (`shield` for policies, `layers` for profiles);
- the title + count;
- one line of help text:
  - "Limit which rows people with this entitlement can see." (policies);
  - "Pick the client populations this entitlement covers." (profiles);
- the **Add data policies / Add data profiles** button (`fd-be`, `plus` icon);
- the chosen items as rows (`fd-cf-i`): name, `#id · Booking_Location IN CN, IN`, owner (mono), and a remove ×;
- or the dashed empty line "No data policies yet." / "No data profiles yet."

These elements must sit inside an `.fd-root` wrapper to get the tokens. Leave the rest of the form alone.

---

## 19. Loading, empty and error states

- **Loading a record:** show the drawer immediately, with the name from the clicked row if you have it. In the body, show skeleton lines (`fd-skel`, e.g. widths 60%, 90%, 40%) in place of the hero and the tab content.
- **Load failure:** show `fd-errbox`, "Couldn't load this {type}." with **Retry**.
- **Empty states** are listed in each section. Keep their exact wording.
- **Unknown or "-" values:** "Not set" in Details; the description empty state in About.

---

## 20. Responsive

- **≤900px:**
  - the side panel becomes full width;
  - the Used by flow stacks vertically;
  - the expanded body becomes one column;
  - the permissions table columns shrink to 64px;
  - the conditions table drops its header row and wraps values under the field.
- The drawer never causes horizontal page scroll. Long names wrap. IDs truncate with an ellipsis where the CSS says so.

---

## 21. Accessibility

- Every clickable thing is a real `<button>`. Icon-only buttons have `aria-label`.
- The drawer is an `<aside aria-label="Record details">`.
- Sheets and dialogs use `role="dialog"` + `aria-label`.
- The menu uses `role="menu"` / `menuitem`.
- Tabs use `role="tablist"` / `tab` + `aria-selected`.
- Tick boxes use `role="checkbox"` + `aria-checked`.
- Expanders use `aria-expanded`.
- Visible focus ring: 2px light teal (in the CSS).
- Colour is never the only signal: removed values are also struck through, drafts also have stripes and a word.

---

## 22. Build phases (report after each)

1. **Foundation.**
   - Add `fmces-drawer.css`, `.fd-root`, the Icon component, the adapters for all 8 types, and the shared drawer shell.
   - The shell includes the toolbar with Prev / Next, the header + ••• menu, the hero, tabs, Overview and History.
   - Open it from all eight list pages.
2. **Main tabs**: Components, Access rule (with draft changes in place), Permissions, Applications, Access.
3. **Compare sheet**.
4. **Expanded centered view**.
5. **Used by + linked records**.
6. **Quick actions, shortcuts, Share and raw JSON**.
7. **Pickers + New / Edit entitlement sections**.
8. **Polish**: loading / empty / error states, responsive, a11y. Run the checklist below and send me the results.

---

## 23. Acceptance checklist

- [ ] All eight list pages open the new drawer. The old drawer is gone and its styles are removed.
- [ ] The drawer covers the full screen height (header included), 720px wide, with a scrim. Clicking the scrim or pressing Esc closes it.
- [ ] Prev / Next follows the list order, keeps the current tab, and is disabled at the ends. "N of M" is correct.
- [ ] SAWFL shows the teal **Live** hero with four filled bars.
- [ ] AppTest Automation shows the amber **Draft** hero, the first bar striped, the other three grey, the note with **Continue draft**, and the Components empty states.
- [ ] Policy Versioning Testing shows the amber note, the striped Draft bar, the amber dot on Access rule, and Live / Draft / Changes with "New in V4" on Japan.
- [ ] Tabs match the table in section 6 for every type. The indicator slides and the counts are right.
- [ ] The tab strip sticks flush to the top of the body with nothing visible above it.
- [ ] Overview: description clamp + Read more; empty description state; 3 facts; Details groups; "Not set"; copy buttons with toast.
- [ ] The permissions table keeps its layout with only the needed action columns; Data policies / Data profiles sections and their empty states.
- [ ] ITAM Applications and Account Access lists. Account remove uses the existing flow.
- [ ] History shows the story order and pills; hover shows "Compare with V1"; the single-version note.
- [ ] Compare: pills, swap, the summary line, scalar and token diffs, the unchanged expander, the identical state.
- [ ] Used by shows only real relationships; the impact line; rows open linked records.
- [ ] Linked records open on top with the peek strip, breadcrumb and Open full record. Back restores the same tab and scroll.
- [ ] Quick actions: /, filter, ↑↓ / Enter, groups. All shortcuts work, and none fire while typing.
- [ ] Share: the summary is correct per type; Copy works; CSV / JSON download (PDF per my answer); raw JSON with colours and line numbers.
- [ ] The expanded view: hero band, left menu + At a glance, overlays inside the sheet. Collapse returns to the same tab.
- [ ] Esc closes layers in the order of section 16. Focus returns to the drawer, so Esc still works after menus close.
- [ ] Pickers: search, All / Selected, tick vs expand, the details dropdown with the conditions table, footer chips, Add / Cancel behaviour, and the page sections.
- [ ] No invented numbers anywhere. Anything not available is hidden, and you have told me what.
- [ ] Reduced motion respected. Keyboard-only use works. Nothing overflows at 1024px and 1440px widths.

---

# Appendix A — `fmces-drawer.css` (copy verbatim)

Save this exactly as it is. Every class is prefixed `fd-` and every rule is scoped, so it cannot leak into the rest of the app. The first rule holds all the tokens. Resets are wrapped in `:where(.fd-root)`, so they have zero specificity.

```css
.fd-root{--bg:#0F1A22;--hdr:#0B141B;--surface:#111E27;--card:#152733;--card2:#1B3140;--border:#23394A;--hair:#1E3240;--btn:#2B4556;--text:#E6EEF2;--text2:#A7B8C4;--muted:#86A0AE;--faint:#6F8896;--accent:#2DD4BF;--accent2:#5EEAD4;--on:#062A26;--draft:#F6C177;--pend:#C9B6FF;--err:#F4A3A3;--blue:#93C5FD;--dw-bg:linear-gradient(180deg,#13232E 0%,#101C25 60%);--raise:rgba(27,49,64,.6);--pop:#172A36;--pop-b:#2E4A5C;--rail:#22394A;--dot:#132430;--bar:#183140;--b2:#3A5A6C;--scrim:rgba(6,12,16,.5);--mg-bg:linear-gradient(145deg,#1F3C4A,#142833);--mg-b:#2C4C5D;--lc-bg:linear-gradient(180deg,rgba(27,49,64,.6),rgba(21,39,51,.3));--lc-b:rgba(43,69,86,.65);--dock-bg:#13222D;--row-h:#172B38;--row-on:#183140;--toast-bg:#E6EEF2;--toast-fg:#0F1A22;--cs-bg:linear-gradient(180deg,#152833,#101C25 70%);--shadow:rgba(0,0,0,.85);--tint:rgba(45,212,191,.10);--tint2:rgba(45,212,191,.16);--dtint:rgba(246,193,119,.12);--etint:rgba(244,163,163,.08);--glow:rgba(45,212,191,.55);--ease:cubic-bezier(.22,1,.36,1);--spring:cubic-bezier(.34,1.36,.64,1);font-family:'Manrope',system-ui,sans-serif;font-size:13px;line-height:1.45;color:var(--text);-webkit-font-smoothing:antialiased;color-scheme:dark}
.fd-root,.fd-root *,.fd-root *::before,.fd-root *::after{box-sizing:border-box}
:where(.fd-root) button{font:inherit;cursor:pointer;color:inherit}
:where(.fd-root) :is(h1,h2,h3,h4,p,ol,ul,dl,dd,pre,blockquote){margin:0;padding:0}
:where(.fd-root) :is(ol,ul){list-style:none}
:where(.fd-root) :focus-visible{outline:2px solid var(--accent2);outline-offset:2px}
.fd-root kbd{display:inline-flex;align-items:center;justify-content:center;min-width:20px;height:20px;padding:0 5px;border-radius:5px;border:1px solid var(--btn);border-bottom-width:2px;background:var(--surface);font:600 10.5px/1 'JetBrains Mono',monospace;color:var(--text2)}
@media (prefers-reduced-motion:reduce){.fd-root *,.fd-root *::before,.fd-root *::after{animation-duration:.01ms!important;animation-iteration-count:1!important;transition-duration:.01ms!important}}
svg.fd-i{display:block;flex-shrink:0;fill:none;stroke:currentColor;stroke-width:1.75;stroke-linecap:round;stroke-linejoin:round}
svg.fd-i.fd-b{stroke-width:2.25}
svg.fd-i.fd-dots{stroke-width:3.2}
.fd-mono{font-family:'JetBrains Mono',ui-monospace,monospace}
.fd-grow{flex:1 1 auto;min-width:0}
.fd-toast{position:absolute;left:50%;bottom:26px;z-index:90;display:flex;align-items:center;gap:8px;height:40px;padding:0 16px 0 12px;border-radius:999px;background:var(--toast-bg);color:var(--toast-fg);font-weight:700;font-size:13px;transform:translate(-50%,16px);opacity:0;pointer-events:none;transition:transform .35s var(--spring),opacity .2s;white-space:nowrap}
.fd-toast.fd-show{transform:translate(-50%,0);opacity:1}
.fd-toast svg{color:var(--accent)}
@keyframes fd-fade{from{opacity:0}}
@keyframes fd-up8{from{opacity:0;transform:translateY(8px)}}
.fd-dw{position:absolute;top:0;right:0;bottom:0;z-index:20;width:var(--dw,620px);max-width:calc(100% - 32px);display:flex;flex-direction:column;background:var(--dw-bg);border-left:1px solid var(--border);box-shadow:-50px 0 90px -40px var(--shadow)}
.fd-dw.fd-dwa{animation:fd-dwa .55s var(--ease) backwards}
.fd-dw.fd-dwb{animation:fd-dwb .55s var(--ease) backwards}
@keyframes fd-dwa{from{opacity:0;transform:translateX(40px)}}
@keyframes fd-dwb{from{opacity:0;transform:translateX(40px)}}
.fd-dh{position:relative;z-index:6;flex-shrink:0;padding:22px 28px 0}
.fd-dh-top{display:flex;align-items:flex-start;gap:14px}
.fd-mg{width:44px;height:44px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:13px;background:var(--mg-bg);border:1px solid var(--mg-b);color:var(--accent2);font-size:14px;font-weight:800;letter-spacing:.04em}
.fd-kind{font-size:12px;font-weight:700;color:var(--accent2)}
.fd-nm{margin-top:3px;font-size:24px;font-weight:700;letter-spacing:-.02em;line-height:1.15;overflow-wrap:anywhere}
.fd-acts{position:relative;display:flex;align-items:center;gap:6px;flex-shrink:0}
.fd-be{display:inline-flex;align-items:center;justify-content:center;gap:8px;height:36px;padding:0 14px;border-radius:10px;border:1px solid var(--tint2);background:var(--tint);color:var(--accent2);font-size:13px;font-weight:700;white-space:nowrap;transition:background-color .2s,color .2s,border-color .2s,transform .3s var(--spring)}
.fd-be:hover{background:var(--accent);border-color:var(--accent);color:var(--on)}
.fd-be:active,.fd-ib2:active{transform:scale(.96)}
.fd-ib2{width:36px;height:36px;display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;padding:0;border-radius:10px;border:1px solid var(--btn);background:transparent;color:var(--text2);transition:background-color .2s,color .2s,border-color .2s,transform .3s var(--spring)}
.fd-ib2:hover,.fd-ib2.fd-on{background:var(--card2);border-color:var(--b2);color:var(--text)}
.fd-ib2.fd-ghost{border-color:transparent}
.fd-st{display:flex;align-items:center;flex-wrap:wrap;gap:6px 16px;font-size:12.5px;color:var(--text2)}
.fd-stx{display:inline-flex;align-items:center;gap:7px;white-space:nowrap}
.fd-stx i{width:7px;height:7px;border-radius:50%;background:currentColor}
.fd-stx.fd-ok{color:var(--accent2)}
.fd-stx.fd-draft{color:var(--draft)}
.fd-stx.fd-mono{font-size:11.5px}
.fd-mscrim{position:fixed;inset:0;z-index:55;border:0;padding:0;background:transparent;cursor:default}
.fd-menu{position:absolute;right:0;top:calc(100% + 8px);z-index:60;width:264px;padding:6px;border-radius:14px;background:var(--pop);border:1px solid var(--pop-b);box-shadow:0 30px 60px -18px var(--shadow);transform-origin:top right;animation:fd-mpop .24s var(--spring) backwards;text-align:left}
@keyframes fd-mpop{from{opacity:0;transform:translateY(-6px) scale(.96)}}
.fd-mh{padding:8px 10px 4px;font-size:11.5px;font-weight:700;color:var(--muted)}
.fd-mi{display:flex;align-items:center;gap:11px;width:100%;height:38px;padding:0 10px;border:0;border-radius:9px;background:transparent;color:var(--text);font-size:13px;font-weight:600;text-align:left;white-space:nowrap}
.fd-mi:hover{background:var(--card2)}
.fd-mi svg{color:var(--text2)}
.fd-mi:hover svg{color:var(--accent2)}
.fd-mi .fd-mx{margin-left:auto;font:500 11px/1.5 'JetBrains Mono',monospace;color:var(--muted)}
.fd-mi.fd-dis{color:var(--faint);cursor:not-allowed}
.fd-mi.fd-dis:hover{background:transparent}
.fd-mi.fd-dis svg{color:var(--faint)}
.fd-msep{height:1px;margin:6px 4px;background:var(--hair)}
.fd-db{position:relative;z-index:1;flex:1 1 auto;min-height:0;overflow:auto;padding:22px 28px 48px}
.fd-db::-webkit-scrollbar{width:10px}
.fd-db::-webkit-scrollbar-thumb{background:var(--btn);border-radius:10px;border:3px solid transparent;background-clip:content-box}
.fd-tpa{animation:fd-tpa .45s var(--ease) backwards}
.fd-tpb{animation:fd-tpb .45s var(--ease) backwards}
@keyframes fd-tpa{from{opacity:0;transform:translateY(8px)}}
@keyframes fd-tpb{from{opacity:0;transform:translateY(8px)}}
.fd-sec+.fd-sec{margin-top:30px;padding-top:26px;border-top:1px solid var(--hair)}
.fd-sh{display:flex;align-items:center;gap:10px;margin-bottom:14px}
.fd-sh h3{font-size:14px;font-weight:700;letter-spacing:-.005em}
.fd-ct{min-width:22px;height:20px;padding:0 7px;display:inline-flex;align-items:center;justify-content:center;border-radius:999px;background:var(--card2);font:600 11px/1 'JetBrains Mono',monospace;color:var(--text2)}
.fd-lk{display:inline-flex;align-items:center;gap:6px;margin-left:auto;height:30px;padding:0 10px;border:0;border-radius:8px;background:transparent;color:var(--accent2);font-size:12.5px;font-weight:700}
.fd-lk:hover{background:var(--tint)}
.fd-lk.fd-dis{color:var(--faint);cursor:default}
.fd-lk.fd-dis:hover{background:transparent}
.fd-ab-d{font-size:17px;font-weight:500;line-height:1.55;letter-spacing:-.01em}
.fd-ab-d.fd-clamp{display:-webkit-box;-webkit-line-clamp:3;-webkit-box-orient:vertical;overflow:hidden}
.fd-ab-more{margin-top:6px;padding:0;border:0;background:transparent;color:var(--accent2);font-size:12.5px;font-weight:700}
.fd-ab-p{margin-top:12px;font-size:13.5px;line-height:1.6;color:var(--text2)}
.fd-facts{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));margin-top:20px}
.fd-facts>div{min-width:0;padding:2px 16px}
.fd-facts>div:first-child{padding-left:0}
.fd-facts>div+div{border-left:1px solid var(--hair)}
.fd-facts b{display:block;font-size:26px;font-weight:300;letter-spacing:-.03em;line-height:1.15;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-facts>div>span{font-size:12px;font-weight:600;color:var(--muted)}
.fd-dg+.fd-dg{margin-top:20px}
.fd-dg-t{margin-bottom:4px;font-size:12px;font-weight:700;color:var(--muted)}
.fd-dl{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));column-gap:28px}
.fd-dl>div{min-width:0;padding:10px 0;border-bottom:1px solid var(--hair)}
.fd-dl dt{font-size:12px;color:var(--muted)}
.fd-dl dd{display:flex;align-items:center;gap:6px;min-width:0;margin-top:3px;font-size:13.5px;font-weight:600}
.fd-dl dd>span{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.fd-dl dd.fd-mono{font-size:12.5px;font-weight:500}
.fd-dl dd.fd-unset{font-weight:500;font-style:italic;color:var(--faint)}
.fd-cp{width:24px;height:24px;flex-shrink:0;display:inline-flex;align-items:center;justify-content:center;padding:0;border:0;border-radius:6px;background:transparent;color:var(--muted);opacity:0;transition:opacity .2s,background-color .2s,color .2s}
.fd-dl>div:hover .fd-cp,.fd-cp:focus-visible{opacity:1}
.fd-cp:hover{background:var(--card2);color:var(--accent2)}
.fd-cmb{display:flex;align-items:center;flex-wrap:wrap;gap:12px}
.fd-seg2{position:relative;display:grid;grid-template-columns:repeat(2,minmax(0,1fr));width:250px;padding:3px;border-radius:11px;background:var(--surface);border:1px solid var(--hair)}
.fd-seg2 .fd-ind{position:absolute;top:3px;bottom:3px;left:3px;width:calc(50% - 3px);border-radius:8px;background:var(--card2);border:1px solid var(--btn);transition:transform .45s var(--spring)}
.fd-seg2 button{position:relative;z-index:1;height:30px;display:flex;align-items:center;justify-content:center;gap:7px;border:0;background:transparent;color:var(--muted);font-size:12.5px;font-weight:700;transition:color .2s}
.fd-seg2 button.fd-on{color:var(--text)}
.fd-seg2 em{font:600 10.5px/1.5 'JetBrains Mono',monospace;font-style:normal;color:var(--muted)}
.fd-fl{flex:1;min-width:170px;display:flex;align-items:center;gap:8px;height:36px;padding:0 12px;border-radius:10px;border:1px solid var(--hair);background:var(--surface);color:var(--muted);transition:border-color .2s,box-shadow .2s}
.fd-fl:focus-within{border-color:var(--accent);box-shadow:0 0 0 3px var(--tint2)}
.fd-fl input{flex:1;min-width:0;height:100%;border:0;outline:0;background:transparent;color:var(--text);font:inherit;font-size:13px}
.fd-fl input::placeholder{color:var(--faint)}
.fd-cl{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));column-gap:28px;margin-top:12px}
.fd-ci{display:flex;align-items:center;gap:10px;min-width:0;height:42px;border-bottom:1px solid var(--hair);font-size:13px;font-weight:600}
.fd-ci svg{color:var(--muted)}
.fd-ci>span{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.fd-ci .fd-mono{font-size:12px;font-weight:500}
.fd-cla .fd-ci{animation:fd-cia .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*22ms)}
.fd-clb .fd-ci{animation:fd-cib .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*22ms)}
@keyframes fd-cia{from{opacity:0;transform:translateY(6px)}}
@keyframes fd-cib{from{opacity:0;transform:translateY(6px)}}
.fd-more{display:inline-flex;align-items:center;gap:8px;margin-top:14px;height:34px;padding:0 12px;border-radius:10px;border:1px dashed var(--btn);background:transparent;color:var(--text2);font-size:12.5px;font-weight:700}
.fd-more:hover{border-color:var(--accent);color:var(--accent2)}
.fd-rule{padding:18px;border-radius:16px;background:var(--surface);border:1px solid var(--hair)}
.fd-rule-s{font-size:14px;color:var(--text2)}
.fd-rule-s b{font-weight:700;color:var(--text)}
.fd-vals{display:flex;flex-wrap:wrap;gap:8px;margin-top:14px}
.fd-val{display:flex;align-items:center;gap:10px;padding:6px 14px 6px 6px;border-radius:12px;background:var(--card);border:1px solid var(--border)}
.fd-val b{min-width:34px;height:28px;padding:0 6px;display:flex;align-items:center;justify-content:center;border-radius:8px;background:var(--tint);color:var(--accent2);font:600 12px/1 'JetBrains Mono',monospace}
.fd-val>span{font-size:13px;font-weight:600}
.fd-pr{display:flex;align-items:center;gap:12px;width:100%;padding:12px 0;border:0;border-bottom:1px solid var(--hair);background:transparent;color:var(--text);text-align:left;font-size:13.5px;font-weight:600}
.fd-pr svg{color:var(--muted)}
.fd-pr .fd-mono{font-size:11.5px;font-weight:500;color:var(--muted)}
.fd-pill{display:inline-flex;align-items:center;height:22px;padding:0 8px;border-radius:6px;font-size:11px;font-weight:700}
.fd-pill.fd-ok{background:var(--tint);color:var(--accent2)}
.fd-pill.fd-draft{background:var(--dtint);color:var(--draft)}
.fd-pill.fd-del{background:var(--card2);color:var(--muted)}
.fd-pill.fd-live{background:var(--accent);color:var(--on)}
.fd-tl-x{flex-shrink:0;height:30px;padding:0 10px;border:0;border-radius:8px;background:transparent;color:var(--accent2);font-size:12px;font-weight:700;white-space:nowrap;opacity:0;transition:opacity .2s,background-color .2s}
.fd-tl-x:focus-visible{opacity:1}
.fd-tl-x:hover{background:var(--tint)}
.fd-dx{display:grid;grid-template-rows:0fr;opacity:0;transition:grid-template-rows .5s var(--ease),opacity .35s}
.fd-dx.fd-open{grid-template-rows:1fr;opacity:1}
.fd-dx>div{min-height:0;overflow:hidden}
.fd-btn{display:inline-flex;align-items:center;gap:8px;height:36px;padding:0 14px;border-radius:10px;border:1px solid transparent;font-size:13px;font-weight:700;white-space:nowrap;transition:background-color .2s,transform .3s var(--spring)}
.fd-btn:active{transform:scale(.97)}
.fd-btn-p{background:var(--accent);color:var(--on)}
.fd-btn-p:hover{background:var(--accent2)}
.fd-btn-g{background:transparent;color:var(--text2)}
.fd-btn-g:hover{background:var(--raise);color:var(--text)}
.fd-cs-wrap{position:absolute;inset:0;z-index:40;display:flex;justify-content:flex-end}
.fd-cs-scrim{position:absolute;inset:0;border:0;padding:0;background:var(--scrim);cursor:default;animation:fd-fade .3s}
.fd-cs{position:relative;width:100%;height:100%;display:flex;flex-direction:column;background:var(--cs-bg);border-left:1px solid var(--border);animation:fd-csin .5s var(--ease) backwards}
@keyframes fd-csin{from{opacity:0;transform:translateX(56px)}}
.fd-cs-h{display:flex;align-items:center;gap:12px;padding:20px 24px 0}
.fd-cs-h h3{margin-top:2px;font-size:19px;font-weight:700;letter-spacing:-.015em}
.fd-cs-pk{display:grid;grid-template-columns:minmax(0,1fr) 40px minmax(0,1fr);align-items:center;gap:12px;margin:18px 24px 0;padding:14px 16px;border-radius:16px;background:var(--surface);border:1px solid var(--hair)}
.fd-cs-lbl{display:flex;align-items:center;gap:6px;margin-bottom:8px;font-size:12px;font-weight:700;color:var(--muted)}
.fd-cs-lbl i{width:7px;height:7px;border-radius:50%}
.fd-cs-lbl.fd-from i{background:var(--err)}
.fd-cs-lbl.fd-to i{background:var(--accent)}
.fd-vp{display:flex;flex-wrap:wrap;gap:6px}
.fd-vpb{display:inline-flex;flex-direction:column;align-items:flex-start;min-width:62px;padding:6px 10px;border-radius:10px;border:1px solid var(--hair);background:transparent;color:var(--text2);font-size:13.5px;font-weight:700;line-height:1.2;transition:border-color .2s,background-color .2s}
.fd-vpb small{font-size:10.5px;font-weight:600;color:var(--muted)}
.fd-vpb:hover{border-color:var(--btn);color:var(--text)}
.fd-vpb.fd-on-from{border-color:var(--err);background:var(--etint);color:var(--text)}
.fd-vpb.fd-on-to{border-color:var(--accent);background:var(--tint);color:var(--text)}
.fd-vpb.fd-off{opacity:.35;pointer-events:none}
.fd-swap{width:40px;height:40px;display:flex;align-items:center;justify-content:center;padding:0;border-radius:50%;border:1px solid var(--btn);background:var(--card);color:var(--text2);transition:transform .5s var(--spring),color .2s}
.fd-swap:hover{transform:rotate(180deg);color:var(--accent2)}
.fd-cs-sum{display:flex;align-items:center;gap:10px;margin:18px 24px 0;font-size:13px;color:var(--text2)}
.fd-cs-sum b{color:var(--text)}
.fd-cs-sum .fd-lg{margin-left:auto;display:flex;gap:12px;font-size:11.5px;color:var(--muted)}
.fd-cs-sum .fd-lg>span{display:inline-flex;align-items:center;gap:5px}
.fd-cs-sum .fd-lg i{width:8px;height:8px;border-radius:2px}
.fd-cs-b{flex:1;min-height:0;overflow:auto;padding:6px 24px 32px}
.fd-chg{padding:16px 0;border-bottom:1px solid var(--hair)}
.fd-csa .fd-chg{animation:fd-up8 .45s var(--ease) backwards;animation-delay:calc(var(--i,0)*70ms + 120ms)}
.fd-csb .fd-chg{animation:fd-chb .45s var(--ease) backwards;animation-delay:calc(var(--i,0)*70ms + 120ms)}
@keyframes fd-chb{from{opacity:0;transform:translateY(8px)}}
.fd-chg-k{margin-bottom:10px;font-size:12px;font-weight:700;color:var(--muted)}
.fd-chg-s{display:flex;align-items:center;flex-wrap:wrap;gap:10px;font-size:13.5px;font-weight:600}
.fd-chg-s svg{color:var(--muted)}
.fd-old{padding:5px 10px;border-radius:8px;background:var(--etint);color:var(--err);text-decoration:line-through}
.fd-new{padding:5px 10px;border-radius:8px;background:var(--tint);color:var(--accent2)}
.fd-tks{display:flex;flex-wrap:wrap;gap:6px}
.fd-tk{display:inline-flex;align-items:center;height:28px;padding:0 10px;border-radius:8px;background:var(--card);border:1px solid var(--border);color:var(--text2);font:600 12px/1 'JetBrains Mono',monospace}
.fd-tk.fd-add{background:var(--tint);border-color:var(--accent);color:var(--accent2)}
.fd-tk.fd-rem{background:var(--etint);border-color:var(--err);color:var(--err);text-decoration:line-through}
.fd-cs-none{padding:28px 0;font-size:13.5px;color:var(--muted)}
.fd-cs-same{display:flex;align-items:center;gap:8px;margin-top:14px;height:32px;padding:0 10px;border:0;border-radius:8px;background:transparent;color:var(--text2);font-size:12.5px;font-weight:700}
.fd-cs-same:hover{background:var(--raise);color:var(--text)}
.fd-cs-same svg{transition:transform .4s var(--ease)}
.fd-cs-same.fd-open svg{transform:rotate(180deg)}
.fd-sm{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));column-gap:28px}
.fd-sm>div{min-width:0;padding:10px 0;border-bottom:1px solid var(--hair)}
.fd-sm dt{font-size:12px;color:var(--muted)}
.fd-sm dd{margin-top:3px;font-size:13px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.fd-lc-s{position:relative;display:flex;flex-direction:column;align-items:center;gap:2px;padding:0 6px;text-align:center;min-width:0}
.fd-lc-s b{font-size:12.5px;font-weight:700;color:var(--muted);white-space:nowrap}
.fd-lc-s.fd-is-done b,.fd-lc-s.fd-is-now b{color:var(--text)}
.fd-lc-s small{max-width:100%;font-size:11.5px;color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-hero{padding:4px 0 30px}
.fd-hero-top{display:flex;align-items:flex-end;gap:20px}
.fd-hero-w{font-size:68px;font-weight:300;letter-spacing:-.05em;line-height:.9;color:var(--accent2);text-shadow:0 0 44px rgba(45,212,191,.35)}
.fd-hero-sub{display:flex;flex-direction:column;padding-bottom:6px;font-size:13px;color:var(--text2)}
.fd-hero-sub b{font-size:15px;font-weight:700;color:var(--text)}
.fd-sg{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:6px;margin-top:26px}
.fd-sg .fd-lc-s{align-items:flex-start;padding:0;text-align:left}
.fd-sg-bar{position:relative;width:100%;height:8px;margin-bottom:10px;border-radius:4px;background:var(--rail);overflow:hidden}
.fd-sg-bar::after{content:"";position:absolute;inset:0;background:linear-gradient(90deg,var(--accent),var(--accent2));transform-origin:left}
.fd-lca .fd-sg-bar::after{animation:fd-sga .5s var(--ease) backwards;animation-delay:calc(var(--k)*190ms + 150ms)}
.fd-lcb .fd-sg-bar::after{animation:fd-sgb .5s var(--ease) backwards;animation-delay:calc(var(--k)*190ms + 150ms)}
@keyframes fd-sga{from{transform:scaleX(0)}}
@keyframes fd-sgb{from{transform:scaleX(0)}}
.fd-sg .fd-is-now .fd-sg-bar{box-shadow:0 0 18px var(--glow)}
.fd-sg .fd-has-draft .fd-sg-bar::before{content:"";position:absolute;inset:0;z-index:1;background:repeating-linear-gradient(-45deg,#F6C177 0 5px,rgba(246,193,119,.25) 5px 10px);background-size:28px 8px;animation:fd-stripe 1.4s linear infinite}
@keyframes fd-stripe{to{background-position:28px 0}}
.fd-hero-dr{display:flex;align-items:center;gap:10px;margin-top:18px;padding:8px 8px 8px 14px;border-radius:12px;background:var(--dtint);color:var(--draft);font-size:13px}
.fd-hero-dr i{width:7px;height:7px;flex-shrink:0;border-radius:50%;background:var(--draft)}
.fd-hero-dr .fd-mono{font-size:12px}
.fd-hero-dr .fd-lk{color:var(--draft)}
.fd-sy{position:relative;margin-top:4px}
.fd-sy::before{content:"";position:absolute;left:73px;top:14px;bottom:14px;width:2px;border-radius:2px;background:var(--rail)}
.fd-ev{position:relative;display:grid;grid-template-columns:58px 32px minmax(0,1fr) auto;align-items:start;padding:11px 0;border-radius:12px;animation:fd-up8 .5s var(--ease) backwards;animation-delay:calc(var(--i,0)*90ms + 120ms)}
.fd-ev:hover{background:var(--raise)}
.fd-sy-d{padding:2px 8px 0 0;font-size:11.5px;color:var(--muted);text-align:right}
.fd-sy-dot{position:relative;z-index:1;justify-self:center;width:12px;height:12px;margin-top:4px;border-radius:50%;background:var(--dot);border:2px solid var(--btn)}
.fd-ev.fd-is-live .fd-sy-dot{background:var(--accent);border-color:var(--accent);box-shadow:0 0 0 5px var(--tint2)}
.fd-ev.fd-is-draft .fd-sy-dot{background:var(--dtint);border-color:var(--draft)}
.fd-ev.fd-is-start .fd-sy-dot{background:var(--text2);border-color:var(--text2)}
.fd-sy-t{display:flex;align-items:center;flex-wrap:wrap;gap:8px;font-size:14px;font-weight:600}
.fd-ev.fd-is-del .fd-sy-t{color:var(--muted)}
.fd-sy-by{margin-top:2px;font-size:11.5px;color:var(--muted)}
.fd-ev .fd-tl-x{margin-right:6px}
.fd-ev:hover .fd-tl-x{opacity:1}
.fd-ptb{position:relative;z-index:2;flex-shrink:0;display:grid;grid-template-columns:repeat(var(--n),minmax(0,1fr));margin:16px 28px 0;padding:4px;border-radius:12px;background:var(--surface);border:1px solid var(--hair)}
.fd-ptb .fd-ind{position:absolute;top:4px;bottom:4px;left:4px;width:calc((100% - 8px) / var(--n));border-radius:9px;background:var(--card2);border:1px solid var(--btn);transition:transform .45s var(--spring)}
.fd-ptb button{position:relative;z-index:1;height:34px;display:flex;align-items:center;justify-content:center;gap:7px;border:0;background:transparent;color:var(--muted);font-size:13px;font-weight:700;transition:color .2s}
.fd-ptb button:hover,.fd-ptb button.fd-on{color:var(--text)}
.fd-ptb em{font:600 10.5px/1 'JetBrains Mono',monospace;font-style:normal;color:var(--muted)}
.fd-dw-sheet{top:26px;bottom:26px;left:0;right:0;width:min(1000px,calc(100% - 48px));max-width:none;margin:0 auto;border-radius:22px;border:1px solid var(--pop-b);background:#122230;box-shadow:0 70px 140px -40px rgba(0,0,0,.95)}
.fd-dw-sheet.fd-dwa{animation-name:fd-sha}
.fd-dw-sheet.fd-dwb{animation-name:fd-shb}
@keyframes fd-sha{from{opacity:0;transform:translateY(18px) scale(.98)}}
@keyframes fd-shb{from{opacity:0;transform:translateY(18px) scale(.98)}}
.fd-dw-sheet .fd-dh{padding:20px 24px 18px;border-bottom:1px solid var(--hair)}
.fd-dw-sheet .fd-dh-top{align-items:center}
.fd-dw-sheet .fd-st{margin-top:6px}
.fd-dw-sheet .fd-nm{font-size:22px}
.fd-sh-body{flex:1 1 auto;min-height:0;display:grid;grid-template-columns:240px minmax(0,1fr)}
.fd-sh-nav{display:flex;flex-direction:column;gap:2px;min-height:0;padding:16px 12px;border-right:1px solid var(--hair);background:rgba(15,26,34,.45);border-radius:0 0 0 22px}
.fd-sh-nav>button{display:flex;align-items:center;gap:11px;height:40px;padding:0 12px;border:0;border-radius:10px;background:transparent;color:var(--text2);font-size:13px;font-weight:700;text-align:left;transition:background-color .2s,color .2s}
.fd-sh-nav>button:hover{background:var(--raise);color:var(--text)}
.fd-sh-nav>button.fd-on{background:var(--card2);color:var(--text);box-shadow:inset 2px 0 0 var(--accent)}
.fd-sh-nav>button svg{color:var(--muted)}
.fd-sh-nav>button.fd-on svg{color:var(--accent2)}
.fd-sh-nav em{margin-left:auto;font:600 10.5px/1 'JetBrains Mono',monospace;font-style:normal;color:var(--muted)}
.fd-sh-main{position:relative;min-width:0;min-height:0;display:flex;flex-direction:column}
.fd-dw-sheet .fd-db{padding:26px 32px 40px}
.fd-dw-sheet .fd-cs-wrap{border-radius:0 0 22px 0;overflow:hidden}
.fd-sh-scrim{position:fixed;inset:0;z-index:90;border:0;padding:0;background:rgba(5,10,14,.64);-webkit-backdrop-filter:blur(5px);backdrop-filter:blur(5px);cursor:default;animation:fd-fade .35s}
.fd-dw-sheet.fd-fx{position:fixed;top:44px;bottom:44px;left:0;right:0;z-index:100;width:min(1040px,calc(100% - 64px));margin:0 auto;outline:none}
.fd-dw-sheet .fd-acts{gap:4px}
.fd-dw-sheet .fd-acts .fd-be{margin-right:2px}
.fd-dw-sheet .fd-tpa,.fd-dw-sheet .fd-tpb{padding-top:0}
.fd-sy.fd-one::before{display:none}
.fd-sy-note{margin:6px 0 0 90px;font-size:12.5px;color:var(--muted)}
.fd-fx-scrim{position:fixed;inset:0;z-index:90;border:0;padding:0;background:rgba(5,10,14,.58);-webkit-backdrop-filter:blur(2px);backdrop-filter:blur(2px);cursor:default;animation:fd-fade .35s}
.fd-dw-final{position:fixed;top:0;right:0;bottom:0;z-index:100;width:var(--dw);max-width:100%;border-left:1px solid var(--border);transition:width .6s var(--ease),border-color .3s;outline:none}
.fd-wrap{width:100%;max-width:1320px;margin:0 auto}
.fd-tbar{position:relative;z-index:8;flex-shrink:0;display:flex;align-items:center;gap:8px;height:54px;padding:0 16px 0 22px;border-bottom:1px solid var(--hair);background:rgba(11,20,27,.6)}
.fd-nb{display:inline-flex;align-items:center;gap:6px;height:32px;padding:0 12px 0 9px;border-radius:9px;border:1px solid var(--btn);background:transparent;color:var(--text);font-size:12.5px;font-weight:700;transition:background-color .2s,color .2s,border-color .2s}
.fd-nb:hover{background:var(--card2)}
.fd-nb.fd-dis{color:var(--faint);border-color:var(--hair);cursor:default}
.fd-nb.fd-dis:hover{background:transparent}
.fd-pos{margin-left:6px;font-size:12px;color:var(--muted)}
.fd-esc{height:26px;padding:0 8px;border-radius:7px;border:1px solid var(--btn);background:var(--surface);color:var(--text2);font:600 11px/1 'JetBrains Mono',monospace}
.fd-esc:hover{color:var(--text);border-color:var(--b2)}
.fd-tbar .fd-ib2{width:34px;height:34px}
.fd-dw-final .fd-dh{padding:22px 32px 18px}
.fd-dw-final .fd-db{padding:30px 32px 64px}
.fd-dw-final .fd-hero{padding-top:0}
.fd-tstrip{position:sticky;top:0;z-index:5;margin:0 -32px;padding:12px 32px;background:rgba(16,29,38,.94);-webkit-backdrop-filter:blur(12px);backdrop-filter:blur(12px)}
.fd-dw-final .fd-ptb{margin:0;border-radius:14px;background:rgba(255,255,255,.035);border-color:rgba(255,255,255,.07)}
.fd-dw-final .fd-ptb .fd-ind{border-radius:10px;background:rgba(255,255,255,.08);border-color:rgba(255,255,255,.12);box-shadow:inset 0 1px 0 rgba(255,255,255,.06)}
.fd-dw-final .fd-ptb button{height:38px;font-size:13.5px}
.fd-dw-final .fd-tpa,.fd-dw-final .fd-tpb{padding-top:20px}
.fd-toast{position:fixed;z-index:120}
.fd-dw-final .fd-hero-w{text-shadow:0 0 28px rgba(45,212,191,.22)}
.fd-dw-final .fd-tstrip{top:-30px}
@media (max-width:900px){.fd-dw-final{width:100%}}
.fd-btn.fd-bs{height:30px;padding:0 12px;border-radius:9px;font-size:12.5px}
.fd-btn-g.fd-bd{border-color:var(--btn)}
.fd-tsep{width:1px;height:20px;margin:0 4px;background:var(--hair)}
.fd-nb kbd{height:18px;min-width:18px;margin-left:2px;font-size:10px}
.fd-nb.fd-on{background:var(--card2);border-color:var(--b2)}
.fd-ptb button.fd-chg::after{content:"";width:6px;height:6px;margin-left:-1px;border-radius:50%;background:var(--draft);box-shadow:0 0 0 3px var(--dtint)}
.fd-dw-sheet>.fd-cs-wrap,.fd-dw-sheet>.fd-dlg-wrap,.fd-dw-sheet>.fd-pal-wrap,.fd-dw-sheet>.fd-stk-wrap{border-radius:22px;overflow:hidden}
.fd-dw-sheet>.fd-cs-wrap .fd-cs{width:min(640px,100%);border-radius:0 22px 22px 0}
.fd-dw-sheet .fd-tsep{margin:0 2px}
.fd-segn{position:relative;display:grid;grid-template-columns:repeat(var(--n),minmax(0,1fr));padding:3px;border-radius:11px;background:var(--surface);border:1px solid var(--hair)}
.fd-segn .fd-ind{position:absolute;top:3px;bottom:3px;left:3px;width:calc((100% - 6px) / var(--n));border-radius:8px;background:var(--card2);border:1px solid var(--btn);transition:transform .45s var(--spring)}
.fd-segn button{position:relative;z-index:1;height:30px;padding:0 14px;display:flex;align-items:center;justify-content:center;gap:7px;border:0;background:transparent;color:var(--muted);font-size:12.5px;font-weight:700;white-space:nowrap;transition:color .2s}
.fd-segn button:hover,.fd-segn button.fd-on{color:var(--text)}
.fd-segn em{font:600 10.5px/1.5 'JetBrains Mono',monospace;font-style:normal;color:var(--muted)}
.fd-dlg-wrap{position:absolute;inset:0;z-index:70;display:flex;align-items:center;justify-content:center;padding:24px}
.fd-dlg-scrim{position:absolute;inset:0;border:0;padding:0;background:rgba(5,10,14,.62);-webkit-backdrop-filter:blur(3px);backdrop-filter:blur(3px);cursor:default;animation:fd-fade .3s}
.fd-dlg{position:relative;width:min(500px,100%);max-height:100%;overflow:auto;padding:24px;border-radius:20px;background:#162A37;border:1px solid var(--pop-b);box-shadow:0 50px 100px -30px rgba(0,0,0,.95);animation:fd-dlgin .4s var(--spring) backwards}
@keyframes fd-dlgin{from{opacity:0;transform:translateY(12px) scale(.97)}}
.fd-uf{display:grid;grid-template-columns:minmax(0,1fr) 30px minmax(0,1fr) 30px minmax(0,1fr);align-items:stretch}
.fd-uf-n{display:flex;flex-direction:column;justify-content:center;min-width:0;padding:14px;border-radius:14px;border:1px solid var(--hair);background:var(--surface);color:var(--text);text-align:left;font:inherit;animation:fd-up8 .45s var(--ease) backwards;transition:border-color .2s,background-color .2s}
.fd-uf>:nth-child(3){animation-delay:.1s}
.fd-uf>:nth-child(5){animation-delay:.2s}
button.fd-uf-n:hover{border-color:var(--btn);background:var(--card)}
.fd-uf-n small{font-size:11.5px;font-weight:700;color:var(--muted)}
.fd-uf-n b{margin-top:4px;font-size:14px;font-weight:700;line-height:1.3;overflow-wrap:anywhere}
.fd-uf-n>span{margin-top:3px;font-size:11.5px;color:var(--muted)}
.fd-uf-n.fd-me{border-color:var(--accent);background:var(--tint);box-shadow:0 0 34px -14px var(--glow)}
.fd-uf-n.fd-me small{color:var(--accent2)}
.fd-uf-a{position:relative;align-self:center;height:2px;margin:0 3px;border-radius:2px;background:var(--btn);overflow:visible}
.fd-uf-a::after{content:"";position:absolute;right:-2px;top:-4px;border:5px solid transparent;border-left:6px solid var(--btn);border-right:0}
.fd-uf-a::before{content:"";position:absolute;top:-2px;left:0;width:6px;height:6px;border-radius:50%;background:var(--accent2);box-shadow:0 0 8px var(--glow);animation:fd-flow 2.2s var(--ease) infinite}
.fd-uf>:nth-child(4)::before{animation-delay:.5s}
@keyframes fd-flow{0%{left:0;opacity:0}20%{opacity:1}80%{opacity:1}100%{left:calc(100% - 6px);opacity:0}}
.fd-uf-imp{display:flex;gap:10px;margin-top:16px;padding:12px 14px;border-radius:12px;background:var(--dtint);color:var(--draft);font-size:13px;font-weight:600;line-height:1.5}
.fd-uf-imp svg{margin-top:1px}
.fd-ug{margin-top:26px}
.fd-pr-k{font-size:12px;font-weight:500;color:var(--muted)}
.fd-pr-n{font-size:12px;font-weight:600;color:var(--text2)}
.fd-pr-go{color:var(--muted)}
.fd-ug .fd-pr:hover,.fd-stk-used .fd-pr:hover{color:var(--accent2)}
.fd-dw-final>.fd-dh,.fd-dw-final>.fd-db,.fd-dw-final>.fd-tbar{transition:transform .55s var(--ease),filter .55s var(--ease)}
.fd-dw-final.fd-has-stk>.fd-dh,.fd-dw-final.fd-has-stk>.fd-db,.fd-dw-final.fd-has-stk>.fd-tbar{transform:scale(.965) translateX(-10px);filter:brightness(.7)}
.fd-stk-wrap{position:absolute;inset:0;z-index:45;display:flex}
.fd-stk-peek{width:64px;flex-shrink:0;display:flex;justify-content:center;border:0;padding:0;background:transparent;color:var(--text2);cursor:pointer;animation:fd-fade .3s}
.fd-stk-peek>span{display:flex;align-items:center;gap:8px;margin-top:140px;height:max-content;padding:12px 7px;border-radius:999px;background:rgba(11,20,27,.75);border:1px solid var(--hair);writing-mode:vertical-rl;transform:rotate(180deg);font-size:12px;font-weight:700;white-space:nowrap;transition:color .2s,border-color .2s}
.fd-stk-peek>span svg{transform:rotate(-90deg)}
.fd-stk-peek:hover>span{color:var(--text);border-color:var(--btn)}
.fd-stk{position:relative;flex:1;min-width:0;display:flex;flex-direction:column;background:linear-gradient(180deg,#16293A,#112230 70%);border-left:1px solid var(--pop-b);border-radius:20px 0 0 20px;box-shadow:-40px 0 80px -30px rgba(0,0,0,.95)}
.fd-stk.fd-ska{animation:fd-ska .55s var(--ease) backwards}
.fd-stk.fd-skb{animation:fd-skb .55s var(--ease) backwards}
@keyframes fd-ska{from{transform:translateX(70px);opacity:0}}
@keyframes fd-skb{from{transform:translateX(70px);opacity:0}}
.fd-stk-h{display:flex;align-items:center;gap:10px;flex-shrink:0;padding:14px 18px 14px 16px;border-bottom:1px solid var(--hair)}
.fd-crumbs{flex:1;min-width:0;display:flex;align-items:center;gap:4px;font-size:12.5px}
.fd-crumbs button{min-width:0;max-width:220px;padding:5px 7px;border:0;border-radius:7px;background:transparent;color:var(--muted);font-weight:700;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-crumbs button:hover{background:var(--raise);color:var(--text)}
.fd-crumbs svg{color:var(--faint)}
.fd-crumbs b{min-width:0;padding-left:3px;font-weight:700;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-stk-h .fd-lk{margin-left:0}
.fd-stk-b{flex:1;min-height:0;overflow:auto;padding:26px 28px 40px}
.fd-stk-id{display:flex;align-items:flex-start;gap:14px}
.fd-stk-id .fd-nm{margin-top:2px;font-size:22px}
.fd-stk-id .fd-st{margin-top:8px}
.fd-stk-bars{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:5px;margin:22px 0 18px}
.fd-stk-bars i{height:6px;border-radius:3px;background:linear-gradient(90deg,var(--accent),var(--accent2));transform-origin:left;animation:fd-sga .45s var(--ease) backwards}
.fd-stk-bars i:nth-child(1){animation-delay:.25s}
.fd-stk-bars i:nth-child(2){animation-delay:.37s}
.fd-stk-bars i:nth-child(3){animation-delay:.49s}
.fd-stk-bars i:nth-child(4){animation-delay:.61s}
.fd-stk .fd-rule{margin-top:18px}
.fd-stk-dl{margin-top:18px}
.fd-stk-used{margin-top:24px}
.fd-shp{position:absolute;top:60px;right:16px;z-index:62;width:400px;max-width:calc(100% - 32px);padding:18px;border-radius:18px;background:var(--pop);border:1px solid var(--pop-b);box-shadow:0 40px 80px -24px rgba(0,0,0,.95);transform-origin:top right;animation:fd-mpop .28s var(--spring) backwards}
.fd-dw-sheet .fd-shp{top:84px;right:24px}
.fd-shp-t{font-size:14.5px;font-weight:700}
.fd-shp-s{margin-top:16px;padding-top:14px;border-top:1px solid var(--hair)}
.fd-shp-s.fd-first{margin-top:12px;padding-top:0;border-top:0}
.fd-shp-r{display:flex;align-items:center}
.fd-shp-r b,.fd-shp-b{font-size:12.5px;font-weight:700}
.fd-shp-r .fd-lk{height:26px}
.fd-shp-sum{margin:8px 0 0;padding:10px 12px;border-radius:10px;background:var(--surface);font:500 11.5px/1.65 'JetBrains Mono',monospace;color:var(--text2);white-space:pre-wrap;overflow-wrap:anywhere}
.fd-shp-x{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:6px;margin-top:8px}
.fd-shp-x button{display:flex;flex-direction:column;align-items:flex-start;gap:1px;padding:10px 11px;border-radius:11px;border:1px solid var(--hair);background:transparent;color:var(--text);text-align:left;transition:border-color .2s,background-color .2s}
.fd-shp-x button:hover{border-color:var(--accent);background:var(--tint)}
.fd-shp-x b{font-size:13px;font-weight:800}
.fd-shp-x small{font-size:11px;color:var(--muted)}
.fd-shp-raw{display:flex;align-items:center;gap:10px;width:100%;height:42px;margin-top:14px;padding:0 12px;border-radius:11px;border:1px dashed var(--btn);background:transparent;color:var(--text2);font-size:12.5px;font-weight:700;transition:border-color .2s,color .2s}
.fd-shp-raw:hover{border-color:var(--accent);color:var(--accent2)}
.fd-raw .fd-cs-h .fd-btn{margin-right:2px}
.fd-code{flex:1;min-height:0;overflow:auto;padding:14px 0 28px;background:rgba(8,15,20,.5);font:500 12px/1.8 'JetBrains Mono',monospace}
.fd-cline{display:flex;padding-right:24px;animation:fd-fade .4s backwards}
.fd-cline:hover{background:rgba(255,255,255,.03)}
.fd-ln{width:52px;flex-shrink:0;padding-right:16px;text-align:right;color:#4E6776;user-select:none}
.fd-lc{min-width:0;white-space:pre;color:var(--text2)}
.fd-jk{color:#93C5FD}
.fd-js{color:#5EEAD4}
.fd-jn{color:#F6C177}
.fd-jb{color:#C9B6FF}
.fd-jp{color:#6F8896}
.fd-kb{display:inline-flex;align-items:center;gap:8px;height:32px;padding:0 6px 0 10px;border-radius:9px;border:1px solid var(--btn);background:transparent;color:var(--text2);font-size:12.5px;font-weight:700;transition:background-color .2s,color .2s}
.fd-kb:hover{background:var(--card2);color:var(--text)}
.fd-pal-wrap{position:absolute;inset:0;z-index:70;display:flex;align-items:flex-start;justify-content:center;padding:84px 24px 24px}
.fd-pal-scrim{position:absolute;inset:0;border:0;padding:0;background:rgba(5,10,14,.58);-webkit-backdrop-filter:blur(4px);backdrop-filter:blur(4px);cursor:default;animation:fd-fade .25s}
.fd-pal{position:relative;width:min(560px,100%);max-height:100%;display:flex;flex-direction:column;border-radius:18px;background:var(--pop);border:1px solid var(--pop-b);box-shadow:0 50px 100px -30px rgba(0,0,0,.95);overflow:hidden;animation:fd-dlgin .32s var(--spring) backwards}
.fd-pal-in{display:flex;align-items:center;gap:12px;flex-shrink:0;height:58px;padding:0 14px 0 18px;border-bottom:1px solid var(--hair);color:var(--muted)}
.fd-pal-in input{flex:1;min-width:0;height:100%;border:0;outline:0;background:transparent;color:var(--text);font:inherit;font-size:15.5px}
.fd-pal-in input::placeholder{color:var(--faint)}
.fd-pal-l{flex:1;min-height:0;overflow:auto;padding:6px}
.fd-pal-g{padding:10px 10px 4px;font-size:11.5px;font-weight:700;color:var(--muted)}
.fd-pi{display:flex;align-items:center;gap:12px;width:100%;height:40px;padding:0 10px;border:0;border-radius:10px;background:transparent;color:var(--text);font-size:13.5px;font-weight:600;text-align:left}
.fd-pi>.fd-grow{white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-pi svg{color:var(--muted)}
.fd-pi.fd-on{background:var(--card2);box-shadow:inset 2px 0 0 var(--accent)}
.fd-pi.fd-on svg{color:var(--accent2)}
.fd-pi.fd-on kbd{border-color:var(--b2);color:var(--text)}
.fd-pal-none{padding:20px 10px;font-size:13px;color:var(--muted)}
.fd-pal-f{display:flex;align-items:center;gap:14px;flex-shrink:0;height:46px;padding:0 8px 0 16px;border-top:1px solid var(--hair);font-size:11.5px;color:var(--muted)}
.fd-pal-f>span{display:inline-flex;align-items:center;gap:4px}
.fd-pal-f kbd{margin-right:3px}
.fd-pal-f .fd-lk{gap:8px;font-size:12px}
.fd-dlg.fd-keys{width:min(640px,100%)}
.fd-kh{display:flex;align-items:center;justify-content:space-between}
.fd-kh h3{margin:0}
.fd-kgs{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:24px;margin-top:16px}
.fd-kr{display:flex;align-items:center;justify-content:space-between;gap:8px;height:36px;border-bottom:1px solid var(--hair);font-size:12.5px;color:var(--text2)}
.fd-kn{margin-top:18px;font-size:12px;color:var(--faint)}
.fd-il-top{display:flex;align-items:center;gap:12px;margin-bottom:14px}
.fd-il-top .fd-segn{width:min(380px,100%)}
.fd-il .fd-val{position:relative;transition:border-color .3s,background-color .3s}
.fd-il .fd-val.fd-add{border:1px dashed var(--accent);background:var(--tint)}
.fd-il .fd-val.fd-add>b{background:var(--accent);color:var(--on)}
.fd-il .fd-val.fd-rem{border-color:rgba(244,163,163,.4);background:var(--etint)}
.fd-il .fd-val.fd-rem>b,.fd-il .fd-val.fd-rem>span{text-decoration:line-through;color:var(--err)}
.fd-il .fd-val em{margin-left:2px;padding:2px 7px;border-radius:6px;font-style:normal;font-size:10.5px;font-weight:800;background:rgba(45,212,191,.18);color:var(--accent2)}
.fd-il .fd-val.fd-rem em{background:rgba(244,163,163,.14);color:var(--err)}
.fd-il .fd-cla .fd-val{animation:fd-cia .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*70ms)}
.fd-il .fd-clb .fd-val{animation:fd-cib .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*70ms)}
.fd-il-lead{display:flex;align-items:center;gap:8px;margin-top:16px;font-size:12.5px;font-weight:600;color:var(--muted)}
.fd-il-lead i{width:7px;height:7px;border-radius:50%;background:var(--muted)}
.fd-il-lead.fd-dif{color:var(--draft)}
.fd-il-lead.fd-dif i{background:var(--draft)}
.fd-il-by{margin-top:10px;font-size:12px;color:var(--faint)}
.fd-il-by .fd-mono{font-size:11.5px}
@media (max-width:900px){.fd-uf{grid-template-columns:minmax(0,1fr)}.fd-uf-a{width:2px;height:18px;justify-self:center;margin:3px 0}.fd-uf-a::after,.fd-uf-a::before{display:none}.fd-kgs{grid-template-columns:minmax(0,1fr)}}
.fd-toast.fd-in-dw{left:calc(100% - min(720px,100%) / 2)}
.fd-pos{white-space:nowrap}
.fd-kb.fd-ic{padding:0 6px 0 9px;gap:6px;height:36px}
.fd-dw-sheet .fd-acts .fd-nb{height:36px}
.fd-dw-sheet.fd-fx{width:min(1120px,calc(100% - 64px))}
.fd-dw-sheet .fd-dh{padding:20px 24px 18px}
.fd-kid{margin-left:10px;padding-left:10px;border-left:1px solid var(--btn);font-size:11.5px;font-weight:500;color:var(--muted)}
.fd-xh{position:relative;z-index:1;flex-shrink:0;display:grid;grid-template-columns:auto minmax(0,1fr);align-items:end;column-gap:52px;row-gap:16px;padding:22px 28px 22px;border-bottom:1px solid var(--hair);background:linear-gradient(180deg,rgba(45,212,191,.05),rgba(45,212,191,0) 85%)}
.fd-xh .fd-hero-top{gap:16px}
.fd-xh .fd-hero-w{font-size:62px;text-shadow:0 0 28px rgba(45,212,191,.22)}
.fd-xh .fd-hero-sub{padding-bottom:4px}
.fd-xh .fd-sg{margin-top:0;padding-bottom:2px}
.fd-xh .fd-hero-dr{grid-column:1/-1;margin-top:0}
.fd-dw-sheet .fd-sh-body{grid-template-columns:220px minmax(0,1fr)}
.fd-dw-sheet .fd-sh-nav{padding:16px 12px 14px}
.fd-qf{margin-top:auto;padding:14px 10px 0;border-top:1px solid var(--hair)}
.fd-qf-h{margin-bottom:4px;font-size:12px;font-weight:700;color:var(--muted)}
.fd-qf-r{padding:8px 0}
.fd-qf-r>span{display:block;font-size:11.5px;color:var(--muted)}
.fd-qf-v{display:flex;align-items:center;gap:4px;min-width:0;margin-top:2px}
.fd-qf-v b{min-width:0;font-size:13px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-qf-v b.fd-mono{font-size:12px;font-weight:500}
.fd-qf-r:hover .fd-cp{opacity:1}
.fd-dw-sheet .fd-db{padding:26px 36px 40px}
@media (max-width:900px){.fd-xh{grid-template-columns:minmax(0,1fr)}.fd-dw-sheet .fd-sh-body{grid-template-columns:minmax(0,1fr)}}
.fd-btn.fd-dis{opacity:.42;cursor:not-allowed}
.fd-btn.fd-dis:active{transform:none}
.fd-mg svg{color:var(--accent2)}
.fd-dh .fd-kind .fd-kid{white-space:nowrap}
.fd-hero-w.fd-dr{color:var(--draft);text-shadow:0 0 40px rgba(246,193,119,.28)}
.fd-dw-final .fd-hero-w.fd-dr{text-shadow:0 0 28px rgba(246,193,119,.22)}
.fd-sg .fd-is-todo .fd-sg-bar::after{display:none}
.fd-sg .fd-is-todo>b{color:var(--muted)}
.fd-sg .fd-is-now.fd-has-draft .fd-sg-bar::after{display:none}
.fd-sg .fd-is-now.fd-has-draft .fd-sg-bar{box-shadow:0 0 18px rgba(246,193,119,.28)}
.fd-sg .fd-is-now.fd-has-draft>b{color:var(--draft)}
.fd-ab-none{display:flex;align-items:center;gap:10px;font-size:15px;font-style:italic;color:var(--faint)}
.fd-ab-none .fd-lk{margin-left:0;font-style:normal}
.fd-empty{padding:18px 0 6px;font-size:13px;color:var(--muted)}
.fd-dash-e{display:flex;align-items:center;gap:10px;padding:14px 16px;border-radius:12px;border:1px dashed var(--btn);font-size:13px;color:var(--muted)}
.fd-dash-e svg{color:var(--faint)}
.fd-sh-n{margin-left:auto;font-size:12px;color:var(--muted)}
.fd-pr-t{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.fd-uf-imp.fd-ok{background:var(--tint);color:var(--accent2)}
.fd-dw-final .fd-dh .fd-nm{font-size:22px}
.fd-pmx{border-radius:14px;border:1px solid var(--hair);background:var(--surface);overflow:hidden}
.fd-pmx-h,.fd-pmx-r{display:grid;grid-template-columns:minmax(0,1fr) repeat(var(--c),96px);align-items:center;padding:0 16px}
.fd-pmx-h{height:40px;border-bottom:1px solid var(--hair);background:rgba(255,255,255,.02);font-size:12px;font-weight:700;color:var(--muted)}
.fd-pmx-h>span+span{text-align:center}
.fd-pmx-r{min-height:46px;border-bottom:1px solid var(--hair);transition:background-color .2s;animation:fd-up8 .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*35ms + 80ms)}
.fd-pmx-r:last-child{border-bottom:0}
.fd-pmx-r:hover{background:var(--raise)}
.fd-pmx-r>b{min-width:0;font-size:13.5px;font-weight:600;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.fd-pmx-c{display:flex;justify-content:center}
.fd-pm-on{width:24px;height:24px;display:flex;align-items:center;justify-content:center;border-radius:7px;background:var(--tint2);border:1px solid rgba(45,212,191,.45);color:var(--accent2)}
.fd-pm-off{width:5px;height:5px;border-radius:50%;background:var(--btn)}
.fd-pr .fd-pill{margin-right:4px}
.fd-acc-sum{display:flex;align-items:center;gap:12px;padding:14px 16px;border-radius:14px;background:var(--tint);border:1px solid var(--tint2);font-size:14px;color:var(--text2)}
.fd-acc-sum svg{color:var(--accent2)}
.fd-acc-sum b{color:var(--text);font-weight:700}
.fd-ag{margin-top:22px}
.fd-ag-h{margin-bottom:6px}
.fd-app-chip{display:inline-flex;align-items:center;height:24px;padding:0 9px;border-radius:7px;background:var(--card2);border:1px solid var(--btn);font-size:11.5px;font-weight:600;color:var(--text2)}
.fd-ent-row{display:flex;align-items:center;gap:6px;border-bottom:1px solid var(--hair)}
.fd-ent-row .fd-pr{border-bottom:0}
.fd-ent-row .fd-mono{font-size:12px;color:var(--muted)}
.fd-rm{width:32px;height:32px;color:var(--muted)}
.fd-rm:hover{color:var(--err);background:var(--etint);border-color:transparent}
.fd-stk .fd-facts{margin-top:18px}
.fd-stk-bars.fd-dr i:first-child{background:repeating-linear-gradient(-45deg,#F6C177 0 5px,rgba(246,193,119,.25) 5px 10px)}
.fd-stk-bars.fd-dr i+i{background:var(--rail);animation:none}
.fd-cf-card{padding:18px 20px;border-radius:16px;background:var(--card);border:1px solid var(--border)}
.fd-cf-h{display:flex;align-items:center;gap:14px}
.fd-cf-ic{width:36px;height:36px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:10px;background:var(--tint2);color:var(--accent2)}
.fd-cf-h h3{display:flex;align-items:center;gap:8px;font-size:14px;font-weight:700}
.fd-cf-h p{margin-top:2px;font-size:12.5px;color:var(--muted)}
.fd-cf-l{margin-top:14px}
.fd-cf-i{display:flex;align-items:center;gap:12px;padding:10px 4px 10px 14px;border-radius:12px;background:var(--surface);border:1px solid var(--hair);animation:fd-up8 .35s var(--ease) backwards}
.fd-cf-i+.fd-cf-i{margin-top:6px}
.fd-cf-i b{display:block;font-size:13.5px;font-weight:700}
.fd-cf-i small{display:block;margin-top:1px;font-size:12px;color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.fd-cf-i small .fd-mono{font-size:11.5px}
.fd-cf-o{font-size:11.5px;color:var(--muted)}
.fd-cf-e{padding:12px 14px;border-radius:12px;border:1px dashed var(--btn);font-size:12.5px;color:var(--muted)}
.fd-pk{display:flex;flex-direction:column}
.fd-pk-h{display:flex;align-items:center;gap:10px;flex-shrink:0;padding:22px 20px 16px 28px}
.fd-pk-h .fd-nm{font-size:22px}
.fd-pk-tools{display:flex;align-items:center;gap:10px;flex-shrink:0;padding:0 28px 14px;border-bottom:1px solid var(--hair)}
.fd-pk-tools .fd-fl{flex:1}
.fd-pk-seg{width:220px}
.fd-pk-l{flex:1;min-height:0;overflow:auto;padding:14px 20px 24px}
.fd-pki{border-radius:14px;border:1px solid transparent;transition:background-color .25s,border-color .25s;animation:fd-up8 .4s var(--ease) backwards;animation-delay:calc(var(--i,0)*45ms + 60ms)}
.fd-pki+.fd-pki{margin-top:4px}
.fd-pki:hover{background:var(--raise)}
.fd-pki.fd-open{background:var(--card);border-color:var(--border)}
.fd-pki.fd-on{border-color:rgba(45,212,191,.45)}
.fd-pki.fd-on:not(.fd-open){background:var(--tint)}
.fd-pki-r{display:flex;align-items:center;gap:4px;padding:4px 6px 4px 10px}
.fd-ckb{width:22px;height:22px;flex-shrink:0;display:flex;align-items:center;justify-content:center;margin-right:6px;padding:0;border-radius:6px;border:1.5px solid var(--b2);background:transparent;color:var(--on);transition:background-color .2s,border-color .2s}
.fd-ckb svg{opacity:0;transform:scale(.5);transition:opacity .2s,transform .3s var(--spring)}
.fd-ckb:hover{border-color:var(--accent)}
.fd-pki.fd-on .fd-ckb{background:var(--accent);border-color:var(--accent)}
.fd-pki.fd-on .fd-ckb svg{opacity:1;transform:none}
.fd-pki-m{flex:1;min-width:0;display:flex;align-items:center;gap:12px;padding:10px 8px;border:0;border-radius:10px;background:transparent;color:var(--text);text-align:left}
.fd-pki-m b{display:block;font-size:14px;font-weight:700}
.fd-pki-m small{display:flex;align-items:center;gap:8px;min-width:0;margin-top:2px;font-size:12px;color:var(--muted);white-space:nowrap;overflow:hidden}
.fd-pki-m small>i{width:3px;height:3px;flex-shrink:0;border-radius:50%;background:var(--faint)}
.fd-pki-m small .fd-mono{font-size:11.5px;overflow:hidden;text-overflow:ellipsis}
.fd-pki-id{font-size:12px;color:var(--muted)}
.fd-pki-ch{color:var(--muted);transition:transform .45s var(--ease)}
.fd-pki.fd-open .fd-pki-ch{transform:rotate(180deg);color:var(--accent2)}
.fd-pkd{padding:4px 18px 18px 48px}
.fd-kvs{display:flex;flex-wrap:wrap;gap:6px}
.fd-kv{display:inline-flex;align-items:center;gap:7px;height:28px;padding:0 10px;border-radius:8px;background:var(--surface);border:1px solid var(--hair);font-size:12px}
.fd-kv small{font-size:11px;color:var(--muted)}
.fd-kv b{font-weight:600}
.fd-kv b.fd-mono{font-size:11.5px;font-weight:500}
.fd-kv.fd-ok{color:var(--accent2);font-weight:700}
.fd-kv.fd-ok i{width:6px;height:6px;border-radius:50%;background:var(--accent)}
.fd-pkd-note{display:flex;align-items:center;gap:8px;margin-top:10px;font-size:12.5px;font-weight:600;color:var(--draft)}
.fd-pkd-note i{width:6px;height:6px;border-radius:50%;background:var(--draft)}
.fd-pkd-h{margin:16px 0 8px;font-size:12px;font-weight:700;color:var(--muted)}
.fd-ctt{border-radius:12px;border:1px solid var(--hair);overflow:hidden}
.fd-ctt-h,.fd-ctt-r{display:grid;grid-template-columns:minmax(0,1.25fr) 52px minmax(0,1.75fr) 40px;align-items:center;gap:10px;padding:0 14px}
.fd-ctt-h{height:34px;background:rgba(255,255,255,.025);border-bottom:1px solid var(--hair);font-size:11.5px;font-weight:700;color:var(--muted)}
.fd-ctt-r{min-height:48px;padding-top:8px;padding-bottom:8px}
.fd-ctt-r+.fd-ctt-r{border-top:1px solid var(--hair)}
.fd-ctt-r .fd-f{min-width:0;font-size:12px;font-weight:600;overflow-wrap:anywhere}
.fd-op{display:inline-flex;align-items:center;height:22px;padding:0 7px;border-radius:6px;background:rgba(147,197,253,.12);color:var(--blue);font-size:11px;font-style:normal;font-weight:600}
.fd-vs{display:flex;flex-wrap:wrap;gap:5px}
.fd-vc{display:inline-flex;align-items:center;gap:6px;height:26px;padding:0 9px 0 4px;border-radius:7px;background:var(--surface);border:1px solid var(--border);font-size:12px;font-weight:600}
.fd-vc b{min-width:24px;height:18px;padding:0 4px;display:flex;align-items:center;justify-content:center;border-radius:5px;background:var(--tint);color:var(--accent2);font-size:10.5px;font-weight:600}
.fd-lg{font-size:11.5px;color:var(--muted)}
.fd-pkd-rel{display:flex;flex-wrap:wrap;gap:6px}
.fd-rc{display:inline-flex;align-items:center;gap:7px;height:30px;padding:0 10px;border-radius:9px;background:var(--surface);border:1px solid var(--hair);font-size:12.5px;font-weight:600}
.fd-rc svg{color:var(--muted)}
.fd-rc .fd-mono{font-size:11.5px;font-weight:500;color:var(--muted)}
.fd-pk-f{display:flex;align-items:center;gap:8px;flex-shrink:0;padding:14px 20px 14px 28px;border-top:1px solid var(--hair);background:rgba(11,20,27,.6)}
.fd-pk-sel{display:flex;flex-wrap:wrap;align-items:center;gap:6px;min-width:0}
.fd-sc2{display:inline-flex;align-items:center;gap:4px;height:28px;padding:0 4px 0 10px;border-radius:999px;background:var(--tint);border:1px solid var(--tint2);color:var(--accent2);font-size:12px;font-weight:700;white-space:nowrap;animation:fd-mpop .25s var(--spring) backwards}
.fd-sc2 button{width:20px;height:20px;display:flex;align-items:center;justify-content:center;padding:0;border:0;border-radius:50%;background:transparent;color:var(--accent2)}
.fd-sc2 button:hover{background:var(--tint2)}
.fd-sc2.fd-more2{padding:0 10px}
.fd-pk-hint{font-size:12.5px;color:var(--muted)}
.fd-toast.fd-in-pk{left:calc(100% - min(620px,100%) / 2)}
@media (max-width:900px){.fd-pmx-h,.fd-pmx-r{grid-template-columns:minmax(0,1fr) repeat(var(--c),64px)}.fd-ctt-h{display:none}.fd-ctt-r{grid-template-columns:minmax(0,1fr) auto}.fd-ctt-r .fd-vs{grid-column:1/-1}.fd-pkd{padding-left:18px}}
.fd-sg .fd-has-pend .fd-sg-bar::before{content:"";position:absolute;inset:0;z-index:1;background:repeating-linear-gradient(-45deg,#C9B6FF 0 5px,rgba(201,182,255,.22) 5px 10px);background-size:28px 8px;animation:fd-stripe 1.4s linear infinite}
.fd-sg .fd-is-now.fd-has-pend .fd-sg-bar::after{display:none}
.fd-sg .fd-has-pend>b{color:var(--pend)}
.fd-hero-w.fd-pd{color:var(--pend);text-shadow:0 0 40px rgba(201,182,255,.25)}
.fd-hero-w.fd-off{color:var(--muted);text-shadow:none}
.fd-hero-dr.fd-pend{background:rgba(201,182,255,.1);color:var(--pend)}
.fd-hero-dr.fd-pend i{background:var(--pend)}
.fd-hero-dr.fd-pend .fd-lk{color:var(--pend)}
.fd-pill.fd-pend{background:rgba(201,182,255,.14);color:var(--pend)}
.fd-ev.fd-is-pend .fd-sy-dot{background:rgba(201,182,255,.2);border-color:var(--pend)}
.fd-stx.fd-pend{color:var(--pend)}
.fd-kv.fd-inact{color:var(--muted);font-weight:700}
.fd-kv.fd-inact i{width:6px;height:6px;border-radius:50%;background:var(--faint)}
.fd-skel{display:block;height:12px;border-radius:6px;background:linear-gradient(90deg,var(--card2) 0%,#24404F 50%,var(--card2) 100%);background-size:200% 100%;animation:fd-shim 1.4s linear infinite}
@keyframes fd-shim{to{background-position:-200% 0}}
.fd-errbox{display:flex;align-items:center;gap:10px;padding:14px 16px;border-radius:12px;background:var(--etint);border:1px solid rgba(244,163,163,.3);color:var(--err);font-size:13px}
.fd-errbox .fd-lk{margin-left:auto;color:var(--err)}
.fd-skels{display:flex;flex-direction:column;gap:12px;padding:6px 0}
.fd-skels+.fd-skels{margin-top:28px}
```

---

# Appendix B — Markup

Translate this to your framework using the table in section 1. Class names, element types, ARIA attributes and the order of children are part of the design: keep them. Text in plain words (for example "Where it’s used") is the exact wording.

`{name}` values are explained in the section of the spec named in each heading. Values that only drive animation:
- `--i` = the item's index in its list (stagger delay).
- `--k` = the bar index 0–3 (lifecycle fill order).
- `--n` = the number of segments in a segmented control.
- `{tabX}`, `{segX}`, `{inlX}`, `{pkSegX}` = `{index of the selected segment} * 100%`.
- `{fd-xxa|fd-xxb}` = the replay pairs from section 2.

Class hint shorthand used below: `{[fd-on]}` means "add `fd-on` when it applies, otherwise nothing".

## B.1 Root wrapper, toast, loading and error

Everything the drawer adds to the page lives in one root, ideally portalled to `document.body`:

```html
<div class="fd-root">
  <!-- side panel (B.2) OR expanded sheet (B.20), never both -->
  <!-- picker (B.21), when open -->
  <div class="fd-toast {[fd-show]} {[fd-in-dw|fd-in-pk]}" role="status" aria-live="polite">
    <Icon name="check" size="16" bold/>{toastText}
  </div>
</div>
```

- The toast element is always rendered; add `fd-show` for 2.4 s. Use `fd-in-dw` while the side panel is open, `fd-in-pk` while the picker is open, neither in the expanded view.

**Skeleton** (section 19), used in place of the hero and the tab content while a record, a linked record or picker details load:

```html
<div class="fd-skels" aria-busy="true" aria-label="Loading">
  <span class="fd-skel" style="width: 60%"></span>
  <span class="fd-skel" style="width: 90%"></span>
  <span class="fd-skel" style="width: 40%"></span>
</div>
```

Use two lines (90%, 60%) inside a picker row's details. Stack two `fd-skels` groups for the hero + tab area.

**Error box** (section 19):

```html
<div class="fd-errbox" role="alert">
  <Icon name="info" size="16"/><span>Couldn’t load this {type}.</span>
  <button type="button" class="fd-lk" onClick={retry}>Retry</button>
</div>
```

## B.2 Side panel shell

Sections 4 and 16. The bracketed names are the parts below, in this exact order.

```html
<!-- if side panel is open -->
  <button type="button" class="fd-fx-scrim" aria-label="Close details" onClick={close}></button>
  <aside class="{fd-dw fd-dw-final fd-dwa|fd-dwb [fd-has-stk]}" style="--dw: 720px" tabindex="-1" ref={dwRef} onKeyDown={onKey} aria-label="Record details">
    [B.3 Toolbar]
    [B.4 Header and ••• menu]
    <div class="fd-db" ref={bodyRef}>
      <div class="fd-wrap">
        [B.5 Lifecycle hero]
        <div data-sec="tabs"></div>
        <div class="fd-tstrip">
          [B.6 Tabs]
        </div>
        <div class="{fd-tpa|fd-tpb}">
          <!-- if tab is Overview -->
            <div class="fd-ov">[B.7 Overview]</div>
          <!-- /if -->
          <!-- if tab is the main tab -->
            [one of B.8 Components / B.9 Access rule / B.10 Permissions / B.11 Applications / B.12 Access]
          <!-- /if -->
          <!-- if tab is History -->[B.13 History]<!-- /if -->
          <!-- if tab is Used by -->[B.15 Used by]<!-- /if -->
        </div>
      </div>
    </div>
    [B.14 Compare sheet]
    [B.17 Share popover]
    [B.18 Raw JSON sheet]
    [B.16 Linked record panel]
    [B.19 Quick actions + shortcuts dialog]
  </aside>
<!-- /if -->
```

`fd-has-stk` is on the aside while a linked record panel is open (section 11).

## B.3 Toolbar

Section 4.1. `{pos}` = "2 of 4". `{expLabel}` = "Expand" (side panel) / "Back to side panel" (sheet). `{expD}` = the `expand` icon in the side panel, `collapse` in the sheet. `fd-dis` on Prev/Next at the ends; `fd-on` on Share while its popover is open.

```html
<div class="fd-tbar">
  <div class="fd-wrap" style="display: flex; align-items: center; gap: 8px">
    <button type="button" class="{fd-nb [fd-dis]}" onClick={prev} aria-disabled={prevDis} title="Previous record (K)">
      <Icon name="chevron-up" size="15" bold/>Prev
    </button>
    <button type="button" class="{fd-nb [fd-dis]}" onClick={next} aria-disabled={nextDis} title="Next record (J)">
      <Icon name="chevron-down" size="15" bold/>Next
    </button>
    <span class="fd-pos">{pos}</span><span class="fd-grow"></span>
    <button type="button" class="fd-kb" onClick={palGo} title="Quick actions">
      <Icon name="search" size="15" bold/>Actions<kbd>/</kbd>
    </button>
    <button type="button" class="{fd-nb [fd-on]}" onClick={shareTog} aria-expanded={shareExp}>
      <Icon name="share" size="15" bold/>Share
    </button>
    <span class="fd-tsep"></span>
    <button type="button" class="fd-ib2 fd-ghost" onClick={toggleExp} aria-label={expLabel} title="{expLabel} (F)">
      <Icon name="{expD}" size="17"/>
    </button>
    <button type="button" class="fd-esc" onClick={close} aria-label="Close (Esc)">Esc</button>
    <button type="button" class="fd-ib2 fd-ghost" onClick={close} aria-label="Close">
      <Icon name="close" size="18"/>
    </button>
  </div>
</div>
```

## B.4 Header and ••• menu

Sections 4.2 and 12. `{r.idk} {r.idv}` = "ID 164" / "ITAM 51824" / "App FMCES". `{r.avatar}` is true for accounts (person icon), otherwise `{r.ini}` shows the initials. `fd-on` on ••• while the menu is open. Menu items (`mitems`) are headings (`m.head`), separators (`m.sepr`) or items (`m.item`, icon `m.d`, optional right text `m.x`).

```html
<header class="fd-dh">
  <div class="fd-wrap">
    <div class="fd-dh-top">
      <span class="fd-mg" aria-hidden="true">
      <!-- if r.avatar -->
        <Icon name="user" size="20"/>
      <!-- /if -->
      <!-- if r.noAv -->
        {r.ini}
      <!-- /if -->
      </span>
      <div class="fd-grow">
        <p class="fd-kind">{r.kind}<span class="fd-kid fd-mono">{r.idk} {r.idv}</span></p>
        <h2 class="fd-nm">{r.name}</h2>
      </div>
      <div class="fd-acts">
        <button type="button" class="fd-be" onClick={editGo}>
          <Icon name="pencil" size="15"/>{r.edit}
        </button>
        <button type="button" class="{fd-ib2 [fd-on]}" onClick={toggleMenu} aria-haspopup="menu" aria-expanded={menuExp} aria-label="More actions">
          <Icon name="dots" size="18" class="fd-dots"/>
        </button>
        <!-- if menuOpen -->
          <button type="button" class="fd-mscrim" aria-label="Close menu" onClick={closeMenu}>
          </button>
          <div class="fd-menu" role="menu" aria-label="More actions">
            <!-- for each m in mitems -->
              <!-- if m.head -->
                <p class="fd-mh">{m.ht}</p>
              <!-- /if -->
              <!-- if m.sepr -->
                <div class="fd-msep">
                </div>
              <!-- /if -->
              <!-- if m.item -->
                <button type="button" class="{fd-mi [fd-dis]}" role="menuitem" onClick={m.f}>
                  <Icon name="{m.d}" size="16"/>{m.t}
                  <!-- if m.hasX -->
                    <span class="fd-mx">{m.x}</span>
                  <!-- /if -->
                </button>
              <!-- /if -->
            <!-- /for -->
          </div>
        <!-- /if -->
      </div>
    </div>
  </div>
</header>
```

## B.5 Lifecycle hero

Section 5. `steps` = the 4 bars (Draft, Pending approval, Approved, Active) with `st.t` label, `st.s` caption, `st.k` 0–3. The hero word gets `fd-dr` (draft), `fd-pd` (pending) or `fd-off` (inactive account). The note gets `fd-pend` when it is about a pending version. `noteA` / `noteBy` / `noteB` = the note sentence split around the user id; `noteLk` = the link text.

```html
<section class="fd-hero {fd-lca|fd-lcb}" data-sec="life" aria-label="Lifecycle">
  <div class="fd-hero-top">
    <p class="{fd-hero-w [fd-dr|fd-pd|fd-off]}">{stage}</p>
    <div class="fd-hero-sub">
      <b>{heroB}</b><span>{heroS}</span>
    </div>
  </div>
  <!-- if hasBars -->
    <div class="fd-sg">
      <!-- for each st in steps -->
        <div class="{fd-lc-s fd-is-done|fd-is-now|fd-is-todo [fd-has-draft|fd-has-pend]}" style="--k: {st.k}">
          <span class="fd-sg-bar"></span><b>{st.t}</b><small>{st.s}</small>
        </div>
      <!-- /for -->
    </div>
  <!-- /if -->
  <!-- if noteOn -->
    <p class="fd-hero-dr {[fd-pend]}">
      <i></i><span class="fd-grow">{noteA}<span class="fd-mono">{noteBy}</span>{noteB}</span>
      <button type="button" class="fd-lk" onClick={noteGo}>{noteLk}</button>
    </p>
  <!-- /if -->
</section>
```

## B.6 Tabs

Section 6. `{tabN}` = number of tabs. `t.n` = the count, shown only when `t.hasN`.

```html
<nav class="fd-ptb" style="--n: {tabN}" role="tablist" aria-label="Sections">
  <span class="fd-ind" style="transform: translateX({tabX})"></span>
  <!-- for each t in tabsE -->
    <button type="button" role="tab" class="{[fd-on] [fd-chg]}" aria-selected={t.sel} onClick={t.pick}>
      {t.t}
      <!-- if t.hasN -->
        <em>{t.n}</em>
      <!-- /if -->
    </button>
  <!-- /for -->
</nav>
```

## B.7 Overview

Section 7. Wrapped in `<div class="fd-ov">` (B.2). `r.desc` description, `r.plain` summary sentence (section 17), `facts` = 3 × `{v, k}`, `groups` = detail groups with `g.t` title and rows `d.k` / `d.v`. `fd-clamp` while a long description is collapsed; `descBtn` = "Read more" / "Show less".

```html
<section class="fd-sec">
  <div class="fd-sh">
    <h3>About</h3>
  </div>
  <!-- if hasDesc -->
    <p class="{fd-ab-d [fd-clamp]}">{r.desc}</p>
    <!-- if longDesc -->
      <button type="button" class="fd-ab-more" onClick={toggleDesc}>{descBtn}</button>
    <!-- /if -->
  <!-- /if -->
  <!-- if noDesc -->
    <p class="fd-ab-none">
      No description yet
      <button type="button" class="fd-lk" onClick={editGo}>Add one</button>
    </p>
  <!-- /if -->
  <p class="fd-ab-p">{r.plain}</p>
  <div class="fd-facts">
    <!-- for each f in facts -->
      <div>
        <b>{f.v}</b><span>{f.k}</span>
      </div>
    <!-- /for -->
  </div>
</section>
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Details</h3>
  </div>
  <!-- for each g in groups -->
    <div class="fd-dg">
      <p class="fd-dg-t">{g.t}</p>
      <dl class="fd-dl">
        <!-- for each d in g.rows -->
          <div>
            <dt>{d.k}</dt>
            <dd class="{[fd-mono|fd-unset]}">
              <span>{d.v}</span>
              <!-- if d.copy -->
                <button type="button" class="fd-cp" onClick={d.cp} aria-label="Copy value">
                  <Icon name="copy" size="13"/>
                </button>
              <!-- /if -->
            </dd>
          </div>
        <!-- /for -->
      </dl>
    </div>
  <!-- /for -->
</section>
```

## B.8 Application → Components

Section 8.1. `nF` / `nA` = feature / action counts. `fph` = "Filter 10 features" / "Filter 52 actions". Icon per row: `box` for features, `zap` for actions. `compEmpty` = the empty-state text. `more` / `moreText` only when paging exists.

```html
<section class="fd-sec">
  <div class="fd-cmb">
    <div class="fd-seg2" role="tablist" aria-label="Component type">
      <span class="fd-ind" style="transform: translateX({segX})"></span>
      <button type="button" class="{[fd-on]}" onClick={pickSeg0}>Features<em>{nF}</em></button>
      <button type="button" class="{[fd-on]}" onClick={pickSeg1}>Actions<em>{nA}</em></button>
    </div>
    <label class="fd-fl">
      <Icon name="search" size="15"/>
      <input type="search" value={q} onChange={onq} placeholder={fph} aria-label={fph}>
    </label>
  </div>
  <ul class="{fd-cl fd-cla|fd-clb}">
    <!-- for each c in comp -->
      <li class="fd-ci" style="--i: {index}">
        <Icon name="{c.d}" size="15"/><span class="{[fd-mono]}">{c.t}</span>
      </li>
    <!-- /for -->
  </ul>
  <!-- if noComp -->
    <p class="fd-empty">{compEmpty}</p>
  <!-- /if -->
  <!-- if more -->
    <button type="button" class="fd-more" onClick={moreGo}>{moreText}</button>
  <!-- /if -->
</section>
```

## B.9 Data profile / Data policy → Access rule

Section 8.2. `inlOff` = normal view (no live + differing draft); `inlOn` = draft changes shown in place. `inlSeg` = Live V1 · Draft V4 · Changes. `v.tag` = "Removed" / "New in V4". `fd-dif` on the lead line in Changes mode. "Profiles used" renders for policies only.

```html
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Access rule</h3>
    <span class="fd-ct">{ruleN}</span>
  </div>
  <!-- if inlOff -->
    <!-- for each c in rule -->
      <div class="fd-rule">
        <p class="fd-rule-s">Rows where <b>{c.k}</b> is any of</p>
        <div class="fd-vals">
          <!-- for each v in c.vals -->
            <span class="fd-val"><b>{v.c}</b><span>{v.n}</span></span>
          <!-- /for -->
        </div>
      </div>
    <!-- /for -->
  <!-- /if -->
  <!-- if inlOn -->
    <div class="fd-il">
      <div class="fd-il-top">
        <div class="fd-segn" style="--n: 3" role="tablist" aria-label="Show version">
          <span class="fd-ind" style="transform: translateX({inlX})"></span>
          <!-- for each x in inlSeg -->
            <button type="button" class="{[fd-on]}" onClick={x.pick}>{x.t}</button>
          <!-- /for -->
        </div>
        <button type="button" class="fd-lk" onClick={draftCmp}>
          <Icon name="compare" size="15"/>Full compare
        </button>
      </div>
      <div class="fd-rule">
        <p class="fd-rule-s">Rows where <b>{inlK}</b> is any of</p>
        <div class="{fd-vals fd-cla|fd-clb}">
          <!-- for each v in inlVals -->
            <span class="{fd-val [fd-add|fd-rem]}" style="--i: {index}"><b>{v.c}</b><span>{v.n}</span>
            <!-- if v.hasTag -->
              <em>{v.tag}</em>
            <!-- /if -->
            </span>
          <!-- /for -->
        </div>
        <p class="{fd-il-lead [fd-dif]}">
          <i></i>{inlLead}
        </p>
      </div>
      <p class="fd-il-by">Draft by <span class="fd-mono">{inlBy}</span>, {inlDate}. Changes in the draft appear here as they’re made.</p>
    </div>
  <!-- /if -->
</section>
<!-- if hasProfiles -->
  <section class="fd-sec">
    <div class="fd-sh">
      <h3>Profiles used</h3>
      <span class="fd-ct">{nProfiles}</span>
    </div>
    <!-- for each p in profiles -->
      <button type="button" class="fd-pr" onClick={p.go}>
        <Icon name="layers" size="16"/>{p.t}<span class="fd-mono">{p.id}</span><span class="fd-grow"></span>
        <!-- if p.link -->
          <Icon name="chevron-right" size="15" class="fd-pr-go"/>
        <!-- /if -->
      </button>
    <!-- /for -->
  </section>
<!-- /if -->
```

## B.10 Entitlement → Permissions

Section 8.3. `{pmC}` = number of action columns; `pmCols` = the action column titles; each row has one cell per column (`x.on` granted / `x.off` not). `pmN` = "9 features", `pmP` = "15 permissions". When data policies / profiles exist, render them as `fd-pr` rows (same markup as the "Profiles used" rows in B.9: `layers` icon for profiles, `shield` for policies) instead of the empty line.

```html
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Permissions</h3>
    <span class="fd-ct">{pmN}</span><span class="fd-sh-n">{pmP}</span>
  </div>
  <div class="fd-pmx" style="--c: {pmC}" role="table" aria-label="Permissions">
    <div class="fd-pmx-h" role="row">
      <span role="columnheader">Feature</span>
      <!-- for each c in pmCols -->
        <span role="columnheader">{c.t}</span>
      <!-- /for -->
    </div>
    <!-- for each p in pmRows -->
      <div class="fd-pmx-r" role="row" style="--i: {p.i}">
        <b role="cell">{p.t}</b>
        <!-- for each x in p.cells -->
          <span class="fd-pmx-c" role="cell" aria-label={x.a}>
          <!-- if x.on -->
            <i class="fd-pm-on"><Icon name="check" size="13" bold/></i>
          <!-- /if -->
          <!-- if x.off -->
            <i class="fd-pm-off"></i>
          <!-- /if -->
          </span>
        <!-- /for -->
      </div>
    <!-- /for -->
  </div>
</section>
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Data policies</h3>
    <span class="fd-ct">{nDpol}</span>
  </div>
  <!-- if noDpol -->
    <p class="fd-dash-e">
      <Icon name="shield" size="16"/>No data policies assigned
    </p>
  <!-- /if -->
</section>
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Data profiles</h3>
    <span class="fd-ct">{nDprof}</span>
  </div>
  <!-- if noDprof -->
    <p class="fd-dash-e">
      <Icon name="layers" size="16"/>No data profiles configured
    </p>
  <!-- /if -->
</section>
```

## B.11 ITAM record → Applications

Section 8.4.

```html
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Applications</h3>
    <span class="fd-ct">{nApps}</span>
  </div>
  <!-- for each a in apps -->
    <button type="button" class="fd-pr" onClick={a.go}>
      <Icon name="grid" size="16"/>{a.t}<span class="fd-mono">{a.id}</span><span class="fd-grow"></span><span class="{fd-pill fd-ok|fd-draft|fd-del}">{a.st}</span><Icon name="chevron-right" size="15" class="fd-pr-go"/>
    </button>
  <!-- /for -->
</section>
```

## B.12 Account → Access

Section 8.5. `accE` = "1 entitlement", `accA` = "1 application".

```html
<section class="fd-sec">
  <p class="fd-acc-sum">
    <Icon name="key" size="17"/><span><b>{accE}</b> in <b>{accA}</b></span>
  </p>
  <!-- for each g in access -->
    <div class="fd-ag">
      <p class="fd-ag-h">
        <span class="fd-app-chip fd-mono">{g.app}</span>
      </p>
      <!-- for each x in g.items -->
        <div class="fd-ent-row">
          <button type="button" class="fd-pr" onClick={x.go}>
            <Icon name="key" size="16"/>{x.t}<span class="fd-grow"></span><span class="fd-mono">{x.id}</span>
          </button>
          <button type="button" class="fd-ib2 fd-ghost fd-rm" onClick={x.rm} aria-label="Remove {x.t}">
            <Icon name="close" size="15"/>
          </button>
        </div>
      <!-- /for -->
    </div>
  <!-- /for -->
</section>
```

## B.13 History (version story)

Section 9.1. `events` newest first. `e.hasX` / `e.xt` = the hover shortcut "Compare with V1". `fd-one` on the list when there is a single version.

```html
<section class="fd-sec">
  <div class="fd-sh">
    <h3>Version history</h3>
    <span class="fd-ct">{nVers}</span>
    <button type="button" class="{fd-lk [fd-dis]}" onClick={cmpGo}>
      <Icon name="compare" size="15"/>Compare
    </button>
  </div>
  <ol class="{fd-sy [fd-one]}">
    <!-- for each e in events -->
      <li class="{fd-ev fd-is-draft|fd-is-pend|fd-is-del|fd-is-ok [fd-is-live]|fd-is-start}" style="--i: {e.i}">
        <span class="fd-sy-d fd-mono">{e.date}</span><span class="fd-sy-dot"></span>
        <div>
          <p class="fd-sy-t">
            {e.t}
            <!-- if e.hasTag -->
              <span class="{fd-pill fd-draft|fd-pend|fd-del|fd-live}">{e.tag}</span>
            <!-- /if -->
          </p>
          <p class="fd-sy-by fd-mono">{e.by}</p>
        </div>
        <!-- if e.hasX -->
          <button type="button" class="fd-tl-x" onClick={e.x}>{e.xt}</button>
        <!-- /if -->
      </li>
    <!-- /for -->
  </ol>
  <!-- if single -->
    <p class="fd-sy-note">Only one version so far. Compare becomes available after the next change.</p>
  <!-- /if -->
</section>
```

## B.14 Compare sheet

Section 9.2. Rendered inside the aside (side panel) or inside `fd-sh-main` (expanded view). `changes`: `c.scalar` rows show `c.a → c.b`; `c.list` rows show tokens (`x.t` = "+ JP", "− HK" or "SG"). `sameText` = "Show 4 unchanged fields" / "Hide unchanged fields".

```html
<!-- if cmpOpen -->
  <div class="fd-cs-wrap">
    <button type="button" class="fd-cs-scrim" aria-label="Close compare" onClick={closeCmp}>
    </button>
    <section class="fd-cs" role="dialog" aria-label="Compare versions">
      <div class="fd-cs-h">
        <button type="button" class="fd-ib2" onClick={closeCmp} aria-label="Back">
          <Icon name="arrow-left" size="17"/>
        </button>
        <div class="fd-grow">
          <p class="fd-kind">Compare versions</p>
          <h3>{r.name}</h3>
        </div>
        <button type="button" class="fd-ib2 fd-ghost" onClick={closeCmp} aria-label="Close">
          <Icon name="close" size="18"/>
        </button>
      </div>
      <div class="fd-cs-pk">
        <div>
          <span class="fd-cs-lbl fd-from"><i></i>From</span>
          <div class="fd-vp">
            <!-- for each p in fromPills -->
              <button type="button" class="{fd-vpb [fd-on-from|fd-on-to] [fd-off]}" onClick={p.pick}>{p.v}<small>{p.st}</small></button>
            <!-- /for -->
          </div>
        </div>
        <button type="button" class="fd-swap" onClick={swap} aria-label="Swap versions">
          <Icon name="swap" size="16"/>
        </button>
        <div>
          <span class="fd-cs-lbl fd-to"><i></i>To</span>
          <div class="fd-vp">
            <!-- for each p in toPills -->
              <button type="button" class="{fd-vpb [fd-on-from|fd-on-to] [fd-off]}" onClick={p.pick}>{p.v}<small>{p.st}</small></button>
            <!-- /for -->
          </div>
        </div>
      </div>
      <p class="fd-cs-sum">
        <span><b>{nChanges} {sumText}</b> from {cmpFrom} to {cmpTo}, {nSame} unchanged</span><span class="fd-lg"><span><i style="background: #2DD4BF"></i>Added</span><span><i style="background: #F4A3A3"></i>Removed</span></span>
      </p>
      <div class="{fd-cs-b fd-csa|fd-csb}">
        <!-- for each c in changes -->
          <div class="fd-chg" style="--i: {c.i}">
            <p class="fd-chg-k">{c.k}</p>
            <!-- if c.scalar -->
              <div class="fd-chg-s">
                <span class="fd-old">{c.a}</span><Icon name="arrow-right" size="15"/><span class="fd-new">{c.b}</span>
              </div>
            <!-- /if -->
            <!-- if c.list -->
              <div class="fd-tks">
                <!-- for each x in c.toks -->
                  <span class="{fd-tk [fd-add|fd-rem]}">{x.t}</span>
                <!-- /for -->
              </div>
            <!-- /if -->
          </div>
        <!-- /for -->
        <!-- if noChanges -->
          <p class="fd-cs-none">These two versions are identical.</p>
        <!-- /if -->
        <!-- if hasSame -->
          <button type="button" class="{fd-cs-same [fd-open]}" onClick={toggleSame}>
            <Icon name="chevron-down" size="15" bold/>{sameText}
          </button>
          <div class="{fd-dx [fd-open]}">
            <div>
              <dl class="fd-sm">
                <!-- for each x in same -->
                  <div>
                    <dt>{x.k}</dt>
                    <dd>{x.v}</dd>
                  </div>
                <!-- /for -->
              </dl>
            </div>
          </div>
        <!-- /if -->
      </div>
    </section>
  </div>
<!-- /if -->
```

## B.15 Used by

Section 10. `uUpL` = "Built on" or "Created by"; `uUpT` / `uUpK` = first upstream name / kind; `uMeK` = "This policy" etc.; `uDownT` / `uDownK` = "2 entitlements" / "Entitlement". `fd-ok` on the impact line when nothing depends on the record. `x.n` = extra text on the right (e.g. "View, Download").

```html
<section class="fd-sec fd-used">
  <div class="fd-sh">
    <h3>Where it’s used</h3>
  </div>
  <div class="fd-uf">
    <button type="button" class="fd-uf-n" onClick={uUpGo}>
      <small>{uUpL}</small><b class="{[fd-mono]}">{uUpT}</b><span>{uUpK}</span>
    </button>
    <span class="fd-uf-a"></span>
    <div class="fd-uf-n fd-me">
      <small>{uMeK}</small><b>{r.name}</b><span>{r.kind}</span>
    </div>
    <span class="fd-uf-a"></span>
    <button type="button" class="fd-uf-n" onClick={uDownGo}>
      <small>Used by</small><b>{uDownT}</b><span>{uDownK}</span>
    </button>
  </div>
  <p class="{fd-uf-imp [fd-ok]}">
    <Icon name="info" size="16"/><span>{uImpact}</span>
  </p>
  <!-- if uHasUp -->
    <div class="fd-ug">
      <p class="fd-dg-t">{uUpL}</p>
      <!-- for each x in uUp -->
        <button type="button" class="fd-pr" onClick={x.go}>
          <Icon name="layers" size="16"/><span class="{[fd-mono]}">{x.t}</span><span class="fd-pr-k">{x.k}</span>
          <!-- if x.hasId -->
            <span class="fd-mono">{x.id}</span>
          <!-- /if -->
          <span class="fd-grow"></span>
          <!-- if x.link -->
            <Icon name="chevron-right" size="15" class="fd-pr-go"/>
          <!-- /if -->
        </button>
      <!-- /for -->
    </div>
  <!-- /if -->
  <div class="fd-ug" data-sec="udown">
    <p class="fd-dg-t">Used by</p>
    <!-- if uHasDown -->
      <!-- for each x in uDown -->
        <button type="button" class="fd-pr" onClick={x.go}>
          <Icon name="key" size="16"/><span class="fd-pr-t">{x.t}</span><span class="fd-pr-k">{x.k}</span>
          <!-- if x.hasId -->
            <span class="fd-mono">{x.id}</span>
          <!-- /if -->
          <span class="fd-grow"></span>
          <!-- if x.hasN -->
            <span class="fd-pr-n">{x.n}</span>
          <!-- /if -->
          <!-- if x.link -->
            <Icon name="chevron-right" size="15" class="fd-pr-go"/>
          <!-- /if -->
        </button>
      <!-- /for -->
    <!-- /if -->
    <!-- if uNone -->
      <p class="fd-dash-e">
        <Icon name="info" size="16"/>Nothing uses this yet
      </p>
    <!-- /if -->
  </div>
</section>
```

## B.16 Linked record panel

Section 11. `crumbFrom` = the parent record name. `st.*` = the linked record: `st.stat` status line, `st.facts` 3 facts, `st.dl` the first 4 detail rows, `st.hasRule` / `st.ruleK` / `st.vals` for profiles and policies. `fd-dr` on the bars when the linked record is draft-only.

```html
<!-- if stOpen -->
  <div class="fd-stk-wrap">
    <button type="button" class="fd-stk-peek" onClick={stBack} aria-label="Back to {crumbFrom}">
      <span><Icon name="arrow-left" size="14"/>{crumbFrom}</span>
    </button>
    <section class="{fd-stk fd-ska|fd-skb}" role="dialog" aria-label={st.name}>
      <div class="fd-stk-h">
        <button type="button" class="fd-ib2" onClick={stBack} aria-label="Back">
          <Icon name="arrow-left" size="17"/>
        </button>
        <nav class="fd-crumbs" aria-label="Breadcrumb">
          <button type="button" onClick={stBack}>{crumbFrom}</button>
          <Icon name="chevron-right" size="14"/><b>{st.name}</b>
        </nav>
        <button type="button" class="fd-lk" onClick={stFull}>Open full record<Icon name="external" size="14"/></button>
      </div>
      <div class="fd-stk-b">
        <div class="fd-stk-id">
          <span class="fd-mg" aria-hidden="true">
          <!-- if st.avatar -->
            <Icon name="user" size="20"/>
          <!-- /if -->
          <!-- if st.noAv -->
            {st.ini}
          <!-- /if -->
          </span>
          <div class="fd-grow">
            <p class="fd-kind">{st.kind}<span class="fd-kid fd-mono">{st.id}</span></p>
            <h3 class="fd-nm">{st.name}</h3>
            <p class="fd-st">
              <span class="{fd-stx fd-ok|fd-draft|fd-pend}"><i></i>{st.stat}</span>
            </p>
          </div>
        </div>
        <div class="{fd-stk-bars [fd-dr]}" aria-hidden="true">
          <i></i><i></i><i></i><i></i>
        </div>
        <p class="fd-ab-p">{st.plain}</p>
        <div class="fd-facts">
          <!-- for each f in st.facts -->
            <div>
              <b>{f.v}</b><span>{f.k}</span>
            </div>
          <!-- /for -->
        </div>
        <!-- if st.hasRule -->
          <div class="fd-rule">
            <p class="fd-rule-s">Rows where <b>{st.ruleK}</b> is any of</p>
            <div class="fd-vals">
              <!-- for each v in st.vals -->
                <span class="fd-val"><b>{v.c}</b><span>{v.n}</span></span>
              <!-- /for -->
            </div>
          </div>
        <!-- /if -->
        <dl class="fd-dl fd-stk-dl">
          <!-- for each d in st.dl -->
            <div>
              <dt>{d.k}</dt>
              <dd class="{[fd-mono|fd-unset]}">
                <span>{d.v}</span>
              </dd>
            </div>
          <!-- /for -->
        </dl>
        <div class="fd-stk-used">
          <p class="fd-dg-t">Linked to</p>
          <button type="button" class="fd-pr" onClick={stBack}>
            <Icon name="layers" size="16"/>{crumbFrom}<span class="fd-pr-k">You came from here</span>
          </button>
        </div>
      </div>
    </section>
  </div>
<!-- /if -->
```

## B.17 Share popover

Section 14. `shSum` = the summary text. `exps` = PDF / CSV / JSON tiles (`x.t` title, `x.d` caption). Hide the PDF tile if we decide so in Step 0.

```html
<!-- if shareOpen -->
  <button type="button" class="fd-mscrim" aria-label="Close share" onClick={shareClose}>
  </button>
  <section class="fd-shp" role="dialog" aria-label="Share">
    <p class="fd-shp-t">Share {r.name}</p>
    <div class="fd-shp-s fd-first">
      <div class="fd-shp-r">
        <b>Summary to paste</b>
        <button type="button" class="fd-lk" onClick={copySum}>
          <Icon name="copy" size="14"/>Copy
        </button>
      </div>
      <pre class="fd-shp-sum">{shSum}</pre>
    </div>
    <div class="fd-shp-s">
      <b class="fd-shp-b">Download</b>
      <div class="fd-shp-x">
        <!-- for each x in exps -->
          <button type="button" onClick={x.go}>
            <b>{x.t}</b><small>{x.d}</small>
          </button>
        <!-- /for -->
      </div>
    </div>
    <button type="button" class="fd-shp-raw" onClick={rawGo}>
      <Icon name="code" size="16"/>View raw JSON<span class="fd-grow"></span><Icon name="arrow-right" size="15"/>
    </button>
  </section>
<!-- /if -->
```

## B.18 Raw JSON sheet

Section 14. `rawLines` = one entry per pretty-printed line: `l.n` line number, `l.segs` coloured tokens (`g.t` text + one class from the token table).

```html
<!-- if rawOpen -->
  <div class="fd-cs-wrap">
    <button type="button" class="fd-cs-scrim" aria-label="Close" onClick={rawClose}>
    </button>
    <section class="fd-cs fd-raw" role="dialog" aria-label="Raw JSON">
      <div class="fd-cs-h">
        <button type="button" class="fd-ib2" onClick={rawClose} aria-label="Back">
          <Icon name="arrow-left" size="17"/>
        </button>
        <div class="fd-grow">
          <p class="fd-kind">Raw JSON · {rawN} lines</p>
          <h3>{r.name}</h3>
        </div>
        <button type="button" class="fd-btn fd-btn-g fd-bd fd-bs" onClick={copyRaw}>
          <Icon name="copy" size="14"/>Copy JSON
        </button>
        <button type="button" class="fd-ib2 fd-ghost" onClick={rawClose} aria-label="Close">
          <Icon name="close" size="18"/>
        </button>
      </div>
      <div class="fd-code">
        <!-- for each l in rawLines -->
          <div class="fd-cline">
            <span class="fd-ln">{l.n}</span><span class="fd-lc">
            <!-- for each g in l.segs -->
              <span class="{fd-jk|fd-js|fd-jn|fd-jb|fd-jp|fd-jw}">{g.t}</span>
            <!-- /for -->
            </span>
          </div>
        <!-- /for -->
      </div>
    </section>
  </div>
<!-- /if -->
```

## B.19 Quick actions palette and shortcuts dialog

Section 13. `palItems` = group headings (`x.head`) and commands (`x.item`, icon `x.d`, key cap `x.k`). `fd-on` marks the highlighted command. `helpGroups` = Move around / Act / Tabs with `k.t` label and `k.k` key.

```html
<!-- if palOpen -->
  <div class="fd-pal-wrap">
    <button type="button" class="fd-pal-scrim" aria-label="Close quick actions" onClick={palClose}>
    </button>
    <section class="fd-pal" role="dialog" aria-label="Quick actions">
      <label class="fd-pal-in">
        <Icon name="search" size="18"/>
        <input type="text" ref={palRef} value={palQ} onChange={onPalQ} onKeyDown={palKey} placeholder="Type a command…" aria-label="Type a command">
        <kbd>Esc</kbd>
      </label>
      <div class="fd-pal-l">
        <!-- for each x in palItems -->
          <!-- if x.head -->
            <p class="fd-pal-g">{x.ht}</p>
          <!-- /if -->
          <!-- if x.item -->
            <button type="button" class="{fd-pi [fd-on]}" onClick={x.run} onMouseMove={x.hov}>
              <Icon name="{x.d}" size="16"/><span class="fd-grow">{x.t}</span>
              <!-- if x.hasK -->
                <kbd>{x.k}</kbd>
              <!-- /if -->
            </button>
          <!-- /if -->
        <!-- /for -->
        <!-- if palNone -->
          <p class="fd-pal-none">No command matches “{palQ}”.</p>
        <!-- /if -->
      </div>
      <div class="fd-pal-f">
        <span><kbd>↑</kbd><kbd>↓</kbd>Move</span><span><kbd>↵</kbd>Run</span><span class="fd-grow"></span>
        <button type="button" class="fd-lk" onClick={helpGo}>
          <kbd>?</kbd>All shortcuts
        </button>
      </div>
    </section>
  </div>
<!-- /if -->
<!-- if helpOpen -->
  <div class="fd-dlg-wrap">
    <button type="button" class="fd-dlg-scrim" aria-label="Close" onClick={helpClose}>
    </button>
    <section class="fd-dlg fd-keys" role="dialog" aria-label="Keyboard shortcuts">
      <div class="fd-kh">
        <h3>Keyboard shortcuts</h3>
        <button type="button" class="fd-ib2 fd-ghost" onClick={helpClose} aria-label="Close">
          <Icon name="close" size="18"/>
        </button>
      </div>
      <div class="fd-kgs">
        <!-- for each g in helpGroups -->
          <div class="fd-kg">
            <p class="fd-dg-t">{g.t}</p>
            <!-- for each k in g.keys -->
              <div class="fd-kr">
                <span>{k.t}</span><kbd>{k.k}</kbd>
              </div>
            <!-- /for -->
          </div>
        <!-- /for -->
      </div>
      <p class="fd-kn">Shortcuts work while the drawer is open and you’re not typing in a field.</p>
    </section>
  </div>
<!-- /if -->
```

## B.20 Expanded sheet shell

Section 15. Same record, same tab content and same overlays as the side panel, in a different frame. Note where Compare sits.

```html
<!-- if expanded view is open -->
  <button type="button" class="fd-sh-scrim" aria-label="Close details" onClick={close}></button>
  <aside class="{fd-dw fd-dw-sheet fd-fx fd-dwa|fd-dwb}" tabindex="-1" ref={dwRef} onKeyDown={onKey} aria-label="Record details">
    [B.20a Sheet header]
    [B.20b Hero band]
    <div class="fd-sh-body">
      [B.20c Left menu]
      <div class="fd-sh-main">
        <div class="fd-db" ref={bodyRef}>
          <div class="{fd-tpa|fd-tpb}">
            [the same tab content as in B.2]
          </div>
        </div>
        [B.14 Compare sheet]
      </div>
    </div>
    [B.17 Share popover]
    [B.18 Raw JSON sheet]
    [B.16 Linked record panel]
    [B.19 Quick actions + shortcuts dialog]
  </aside>
<!-- /if -->
```

### B.20a Sheet header

Same identity block as B.4. In this view `{expD}` is the `collapse` icon and `{expLabel}` is "Back to side panel".

```html
<header class="fd-dh">
  <div class="fd-dh-top">
    <span class="fd-mg" aria-hidden="true">
    <!-- if r.avatar -->
      <Icon name="user" size="20"/>
    <!-- /if -->
    <!-- if r.noAv -->
      {r.ini}
    <!-- /if -->
    </span>
    <div class="fd-grow">
      <p class="fd-kind">{r.kind}<span class="fd-kid fd-mono">{r.idk} {r.idv}</span></p>
      <h2 class="fd-nm">{r.name}</h2>
    </div>
    <div class="fd-acts">
      <button type="button" class="fd-kb fd-ic" onClick={palGo} aria-label="Quick actions" title="Quick actions (/)">
        <Icon name="search" size="15" bold/><kbd>/</kbd>
      </button>
      <button type="button" class="{fd-nb [fd-on]}" onClick={shareTog} aria-expanded={shareExp}>
        <Icon name="share" size="15" bold/>Share
      </button>
      <span class="fd-tsep"></span>
      <button type="button" class="fd-be" onClick={editGo}>
        <Icon name="pencil" size="15"/>{r.edit}
      </button>
      <button type="button" class="{fd-ib2 [fd-on]}" onClick={toggleMenu} aria-haspopup="menu" aria-expanded={menuExp} aria-label="More actions">
        <Icon name="dots" size="18" class="fd-dots"/>
      </button>
      <button type="button" class="fd-ib2 fd-ghost" onClick={toggleExp} aria-label={expLabel} title="{expLabel} (F)">
        <Icon name="{expD}" size="17"/>
      </button>
      <button type="button" class="fd-ib2 fd-ghost" onClick={close} aria-label="Close">
        <Icon name="close" size="18"/>
      </button>
      <!-- if menuOpen -->
        <button type="button" class="fd-mscrim" aria-label="Close menu" onClick={closeMenu}>
        </button>
        <div class="fd-menu" role="menu" aria-label="More actions">
          <!-- for each m in mitems -->
            <!-- if m.head -->
              <p class="fd-mh">{m.ht}</p>
            <!-- /if -->
            <!-- if m.sepr -->
              <div class="fd-msep">
              </div>
            <!-- /if -->
            <!-- if m.item -->
              <button type="button" class="{fd-mi [fd-dis]}" role="menuitem" onClick={m.f}>
                <Icon name="{m.d}" size="16"/>{m.t}
                <!-- if m.hasX -->
                  <span class="fd-mx">{m.x}</span>
                <!-- /if -->
              </button>
            <!-- /if -->
          <!-- /for -->
        </div>
      <!-- /if -->
    </div>
  </div>
</header>
```

### B.20b Hero band

Same values as B.5, laid out horizontally by the CSS.

```html
<section class="fd-xh {fd-lca|fd-lcb}" aria-label="Lifecycle">
  <div class="fd-hero-top">
    <p class="{fd-hero-w [fd-dr|fd-pd|fd-off]}">{stage}</p>
    <div class="fd-hero-sub">
      <b>{heroB}</b><span>{heroS}</span>
    </div>
  </div>
  <!-- if hasBars -->
    <div class="fd-sg">
      <!-- for each st in steps -->
        <div class="{fd-lc-s fd-is-done|fd-is-now|fd-is-todo [fd-has-draft|fd-has-pend]}" style="--k: {st.k}">
          <span class="fd-sg-bar"></span><b>{st.t}</b><small>{st.s}</small>
        </div>
      <!-- /for -->
    </div>
  <!-- /if -->
  <!-- if noteOn -->
    <p class="fd-hero-dr {[fd-pend]}">
      <i></i><span class="fd-grow">{noteA}<span class="fd-mono">{noteBy}</span>{noteB}</span>
      <button type="button" class="fd-lk" onClick={noteGo}>{noteLk}</button>
    </p>
  <!-- /if -->
</section>
```

### B.20c Left menu with "At a glance"

`t.d` = the tab icon: Overview `info`, Components `box`, Access rule `filter`, Permissions `key`, Applications `grid`, Access `key`, History `history`, Used by `nodes`. `qf` = the 3 "At a glance" rows from section 15 (`f.copy` only on the ID row).

```html
<nav class="fd-sh-nav" role="tablist" aria-label="Sections">
  <!-- for each t in tabsE -->
    <button type="button" role="tab" class="{[fd-on] [fd-chg]}" aria-selected={t.sel} onClick={t.pick}>
      <Icon name="{t.d}" size="16"/>{t.t}
      <!-- if t.hasN -->
        <em>{t.n}</em>
      <!-- /if -->
    </button>
  <!-- /for -->
  <div class="fd-qf">
    <p class="fd-qf-h">At a glance</p>
    <!-- for each f in qf -->
      <div class="fd-qf-r">
        <span>{f.k}</span>
        <div class="fd-qf-v">
          <b class="{[fd-mono]}">{f.v}</b>
          <!-- if f.copy -->
            <button type="button" class="fd-cp" onClick={f.cp} aria-label="Copy">
              <Icon name="copy" size="13"/>
            </button>
          <!-- /if -->
        </div>
      </div>
    <!-- /for -->
  </div>
</nav>
```

## B.21 Picker

Section 18. `{pkContext}` = "New entitlement" or "Edit entitlement". `{pkTitle}` = "Select data policies" / "Select data profiles". Row classes: `fd-on` when ticked, `fd-open` when its details are open. `x.sum` = first condition + " +1" when there are more. `x.status` = "Active" (`fd-ok`) or "Inactive" (`fd-inact`), from data. `x.hasNote` / `x.note` = the amber draft / pending line [IF DATA EXISTS]. `x.conds` = conditions (`c.f` field, `c.op` raw operator, `c.vals` chips with `v.c` code + `v.n` name, `c.logic` AND / OR / "—"). `x.relK` = "Applies to profiles" / "Used by policies". `pkSelChips` = the first 3 ticked items, `pkMoreT` = "+N". `pkAddT` = "Add 2 policies" etc.

```html
<!-- if pkOpen -->
  <button type="button" class="fd-fx-scrim" aria-label="Close" onClick={pkClose}>
  </button>
  <aside class="{fd-dw fd-dw-final fd-pk fd-dwa|fd-dwb}" style="--dw: 620px" tabindex="-1" ref={dwRef} onKeyDown={onKey} aria-label={pkTitle}>
    <header class="fd-pk-h">
      <div class="fd-grow">
        <p class="fd-kind">{pkContext}</p>
        <h2 class="fd-nm">{pkTitle}</h2>
      </div>
      <button type="button" class="fd-esc" onClick={pkClose} aria-label="Close (Esc)">Esc</button>
      <button type="button" class="fd-ib2 fd-ghost" onClick={pkClose} aria-label="Close">
        <Icon name="close" size="18"/>
      </button>
    </header>
    <div class="fd-pk-tools">
      <label class="fd-fl">
        <Icon name="search" size="15"/>
        <input type="search" value={pq} onChange={onPq} placeholder="Search by name, ID or owner" aria-label="Search by name, ID or owner">
      </label>
      <div class="fd-segn fd-pk-seg" style="--n: 2" role="tablist" aria-label="Show">
        <span class="fd-ind" style="transform: translateX({pkSegX})"></span>
        <button type="button" class="{[fd-on]}" onClick={pkAll}>All<em>{pkAllN}</em></button>
        <button type="button" class="{[fd-on]}" onClick={pkSel}>Selected<em>{pkSelN}</em></button>
      </div>
    </div>
    <div class="fd-pk-l">
      <!-- for each x in pkItems -->
        <div class="{fd-pki [fd-on] [fd-open]}" style="--i: {x.i}">
          <div class="fd-pki-r">
            <button type="button" class="fd-ckb" role="checkbox" aria-checked={x.ck} aria-label="Select {x.name}" onClick={x.togSel}>
              <Icon name="check" size="13" bold/>
            </button>
            <button type="button" class="fd-pki-m" aria-expanded={x.ex} onClick={x.togExp}>
              <span class="fd-grow"><b>{x.name}</b><small><span class="fd-mono">{x.owner}</span><i></i><span class="fd-mono">{x.sum}</span></small></span><span class="fd-pki-id fd-mono">#{x.id}</span><Icon name="chevron-down" size="16" bold class="fd-pki-ch"/>
            </button>
          </div>
          <div class="{fd-dx [fd-open]}">
            <div>
              <div class="fd-pkd">
                <div class="fd-kvs">
                  <span class="fd-kv"><small>ID</small><b class="fd-mono">{x.id}</b></span><span class="fd-kv"><small>Owner</small><b class="fd-mono">{x.owner}</b></span>
                  <!-- if x.hasMod -->
                    <span class="fd-kv"><small>Modified</small><b>{x.mod}</b></span>
                  <!-- /if -->
                  <span class="{fd-kv fd-ok|fd-inact}"><i></i>{x.status}</span>
                </div>
                <!-- if x.hasNote -->
                  <p class="fd-pkd-note">
                    <i></i>{x.note}
                  </p>
                <!-- /if -->
                <p class="fd-pkd-h">Conditions</p>
                <div class="fd-ctt" role="table" aria-label="Conditions">
                  <div class="fd-ctt-h" role="row">
                    <span>Field</span><span>Operator</span><span>Values</span><span>Logic</span>
                  </div>
                  <!-- for each c in x.conds -->
                    <div class="fd-ctt-r" role="row">
                      <span class="fd-mono fd-f">{c.f}</span><span><i class="fd-op fd-mono">{c.op}</i></span><span class="fd-vs">
                      <!-- for each v in c.vals -->
                        <span class="fd-vc"><b class="fd-mono">{v.c}</b>{v.n}</span>
                      <!-- /for -->
                      </span><span class="fd-lg fd-mono">{c.logic}</span>
                    </div>
                  <!-- /for -->
                </div>
                <!-- if x.hasRel -->
                  <p class="fd-pkd-h">{x.relK}</p>
                  <div class="fd-pkd-rel">
                    <!-- for each y in x.rel -->
                      <span class="fd-rc"><Icon name="layers" size="14"/>{y.t}<span class="fd-mono">{y.id}</span></span>
                    <!-- /for -->
                  </div>
                <!-- /if -->
              </div>
            </div>
          </div>
        </div>
      <!-- /for -->
      <!-- if pkNone -->
        <p class="fd-empty">{pkNoneT}</p>
      <!-- /if -->
    </div>
    <footer class="fd-pk-f">
      <div class="fd-grow fd-pk-sel">
        <!-- if pkHasSel -->
          <!-- for each c in pkSelChips -->
            <span class="fd-sc2">{c.t}
            <button type="button" onClick={c.rm} aria-label="Unselect {c.t}">
              <Icon name="close" size="12" bold/>
            </button>
            </span>
          <!-- /for -->
          <!-- if pkMore -->
            <span class="fd-sc2 fd-more2">{pkMoreT}</span>
          <!-- /if -->
        <!-- /if -->
        <!-- if pkNoSel -->
          <span class="fd-pk-hint">Tick the ones this entitlement should use</span>
        <!-- /if -->
      </div>
      <button type="button" class="fd-btn fd-btn-g" onClick={pkClose}>Cancel</button>
      <button type="button" class="{fd-btn fd-btn-p [fd-dis]}" onClick={pkAdd}>{pkAddT}</button>
    </footer>
  </aside>
<!-- /if -->
```

## B.22 New entitlement sections

Section 18.6. These two cards replace only the Data policies and Data profiles sections of the New / Edit entitlement form, inside an `fd-root` wrapper. `x.sum` = "Booking_Location IN CN, IN".

```html
<section class="fd-cf-card">
  <div class="fd-cf-h">
    <span class="fd-cf-ic"><Icon name="shield" size="17"/></span>
    <div class="fd-grow">
      <h3>Data policies <span class="fd-ct">{nPol}</span></h3>
      <p>Limit which rows people with this entitlement can see.</p>
    </div>
    <button type="button" class="fd-be" onClick={openPol}>
      <Icon name="plus" size="15" bold/>Add data policies
    </button>
  </div>
  <div class="fd-cf-l">
    <!-- if hasPol -->
      <!-- for each x in crPols -->
        <div class="fd-cf-i">
          <span class="fd-grow"><b>{x.name}</b><small><span class="fd-mono">#{x.id}</span> · {x.sum}</small></span><span class="fd-cf-o fd-mono">{x.owner}</span>
          <button type="button" class="fd-ib2 fd-ghost fd-rm" onClick={x.rm} aria-label="Remove {x.name}">
            <Icon name="close" size="15"/>
          </button>
        </div>
      <!-- /for -->
    <!-- /if -->
    <!-- if noPol -->
      <p class="fd-cf-e">No data policies yet.</p>
    <!-- /if -->
  </div>
</section>
<section class="fd-cf-card">
  <div class="fd-cf-h">
    <span class="fd-cf-ic"><Icon name="layers" size="17"/></span>
    <div class="fd-grow">
      <h3>Data profiles <span class="fd-ct">{nProf}</span></h3>
      <p>Pick the client populations this entitlement covers.</p>
    </div>
    <button type="button" class="fd-be" onClick={openProf}>
      <Icon name="plus" size="15" bold/>Add data profiles
    </button>
  </div>
  <div class="fd-cf-l">
    <!-- if hasProf -->
      <!-- for each x in crProfs -->
        <div class="fd-cf-i">
          <span class="fd-grow"><b>{x.name}</b><small><span class="fd-mono">#{x.id}</span> · {x.sum}</small></span><span class="fd-cf-o fd-mono">{x.owner}</span>
          <button type="button" class="fd-ib2 fd-ghost fd-rm" onClick={x.rm} aria-label="Remove {x.name}">
            <Icon name="close" size="15"/>
          </button>
        </div>
      <!-- /for -->
    <!-- /if -->
    <!-- if noProf -->
      <p class="fd-cf-e">No data profiles yet.</p>
    <!-- /if -->
  </div>
</section>
```

---

# Appendix C — Icons

One `<path>` each, drawn with the Icon component from section 1 (stroke only, `fill: none`, round caps and joins come from the CSS). If the app already has an icon library, you may use it **only** if the shapes match these closely; otherwise use these paths.

| Name | Path `d` (24×24 viewBox) | Used for |
|---|---|---|
| `pencil` | `M4 20h4L19 9l-4-4L4 16zM13.5 6.5l4 4` | Edit button, palette "Edit" |
| `dots` | `M5 12h.01M12 12h.01M19 12h.01` | ••• menu button |
| `close` | `M6 6l12 12M18 6 6 18` | Close, remove ×, chip × |
| `check` | `M5 12.5l4.5 4.5L19 7.5` | Permission cells, tick boxes, toast |
| `arrow-right` | `M5 12h14M13 6l6 6-6 6` | Compare value arrow, "View raw JSON" |
| `arrow-left` | `M19 12H5M11 6l-6 6 6 6` | Back buttons, peek strip |
| `chevron-down` | `M6 9l6 6 6-6` | Next record, expanders, picker row chevron |
| `swap` | `M7 4 3 8l4 4M3 8h14M17 20l4-4-4-4M21 16H7` | Compare swap |
| `search` | `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4` | Actions button, filters, palette |
| `copy` | `M9 9h10v10H9zM5 15V5h10` | Copy buttons, "Copy name" |
| `compare` | `M7 7h13M16 3l4 4-4 4M17 17H4M8 13l-4 4 4 4` | Compare links and commands |
| `layers` | `M12 3 3 7.5l9 4.5 9-4.5zM3 12l9 4.5 9-4.5M3 16.5 12 21l9-4.5` | Profiles / policies rows, Used by "Built on", linked commands |
| `info` | `M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zM12 11v6M12 7.5v.01` | Overview tab, impact line, empty Used by, error box |
| `history` | `M3 12a9 9 0 1 0 3-6.7L3 8M3 3v5h5M12 7v5l3 2` | History tab, "Version history" |
| `box` | `M12 3 4 7v10l8 4 8-4V7zM4 7l8 4 8-4M12 11v10` | Features, Components tab |
| `chevron-up` | `M6 15l6-6 6 6` | Previous record |
| `shield` | `M12 3 4 6v6c0 5 3.5 8 8 9 4.5-1 8-4 8-9V6z` | Data policies (empty line, page card) |
| `share` | `M12 15V3M7 8l5-5 5 5M5 13v7h14v-7` | Share button |
| `code` | `M8 7l-5 5 5 5M16 7l5 5-5 5M14 4l-4 16` | Raw JSON |
| `external` | `M14 4h6v6M20 4l-9 9M18 14v6H4V6h6` | "Open full record" |
| `chevron-right` | `M9 6l6 6-6 6` | Openable rows, breadcrumb |
| `key` | `M15 4a5 5 0 1 0 0 10 5 5 0 0 0 0-10zM11.5 12.5 3 21M6 18l2.5 2.5M8.5 15.5 11 18` | Entitlements, Permissions / Access tabs, Used by rows |
| `user` | `M12 12a4 4 0 1 0 0-8 4 4 0 0 0 0 8zM4 21a8 8 0 0 1 16 0` | Account avatar |
| `grid` | `M4 4h6v6H4zM14 4h6v6h-6zM4 14h6v6H4zM14 14h6v6h-6z` | Applications |
| `plus` | `M12 5v14M5 12h14` | "Add data policies / profiles" |
| `zap` | `M13 3 5 14h6l-1 7 8-11h-6z` | Actions (component type) |
| `filter` | `M4 5h16l-6 8v6l-4-2v-4z` | Access rule tab |
| `nodes` | `M6 8.5a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5zM6 20.5a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5zM18 14.5a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5zM8.3 7.2l7.4 3.6M8.3 16.8l7.4-3.6` | Used by tab |
| `hash` | `M5 9h14M5 15h14M10 3 8 21M16 3l-2 18` | "Copy ID" |
| `doc` | `M7 3h7l5 5v13H7zM14 3v5h5M10 13h6M10 17h6` | "Copy all details" / "Copy summary" |
| `expand` | `M15 3h6v6M21 3l-7 7M9 21H3v-6M3 21l7-7` | Expand to centered view |
| `collapse` | `M4 14h6v6M10 14l-7 7M20 10h-6V4M14 10l7-7` | Back to side panel |

Palette command icons (section 13): Edit / Continue draft `pencil`, Compare versions `compare`, Version history `history`, Copy summary `doc`, Copy ID `hash`, View raw JSON `code`, Linked items `layers`, Go to a tab = that tab's icon (B.20c), Next `chevron-down`, Previous `chevron-up`, Expand `expand` / Back to side panel `collapse`, Close `close`.

---

**End of the prompt.** Start with the Step 0 report. Don't write code until I reply to it.
