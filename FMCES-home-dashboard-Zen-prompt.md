# FMCES: rebuild the Home dashboard — "Zen" layout

Rebuild the **Home** page of FMCES. FMCES is an internal "Markets Operations One" app with a dark navy theme and a teal accent.

The new Home is calm and typographic:
- a large thin clock and a greeting;
- one status line;
- a "Jump to anything" search box;
- six large module names. Hovering a name opens a preview panel on the right;
- a slow ticker of recently visited pages along the bottom.

There are no images. Everything you need is in this prompt. Use the numbers given here exactly — never adjust them by eye.

## 0. Rules

**Keep unchanged:**
- Routes and navigation targets.
- API calls and payloads.
- Authentication and permissions.
- Any ids, names or data-testids that tests use.

**Do not touch:**
- The header (logo, New tile, theme, clock, feedback, avatar).
- The workspace tabs.
- The floating dock.
- Every page other than Home.

Do not add endpoints and do not change backend code.

**Remove** everything on Home today: the old welcome card, roles card, pending-approval cards and recent-visits list. The new layout replaces all of it.

**[IF DATA EXISTS]** means: build it only if the app already has this data, or an existing API returns it. Otherwise follow the fallback written next to it. Never show placeholder or invented values.

**CSS:**
- Copy Appendix A **verbatim** into a new global stylesheet, `home-zen.css`.
- Load it only on Home, or globally — it is fully scoped under `.hz`, and every class starts with `hz-`.
- Do not translate it to Tailwind, CSS modules or inline styles. Do not rename classes.
- The only inline styles allowed are the `style="--i:N"` stagger indexes shown in Appendix B.

**Fonts:**
- The design needs **Manrope weight 300** (the thin clock and module names).
- Make sure the font link includes 300. Replace the existing link if it lacks it:
  `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Manrope:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap">`

**Page height:** set `--hz-top` on `.hz` to the real combined height of the header and tabs. In the reference that is 56px + 40px = 96px, which is the default. The page then fills the rest of the screen.

If the content container adds padding around routes, remove it for Home only. Otherwise the background glow and the bottom ticker will not reach the edges.

### Step 0 — map the current code first

Put this in your first message or the PR description:

1. **Home.** The Home route component, and which element scrolls (the main content container).
2. **Data available today.** For each item, say where it comes from:
   - user display name and initials;
   - region;
   - roles (application + role name);
   - pending-approval counts per type: Policies, Profiles, Applications, Features, Actions;
   - recent visits (page + date-time).
3. **[IF DATA EXISTS] data.** For each item, say whether it exists:
   - the current user's drafts (name, type, id, version, last edited, and a condition summary if available);
   - the entitlement count in FMCES;
   - a global or record search API;
   - a Bank ID user lookup.
4. **Routes.** The routes for:
   - the 6 modules — App Management, Data Profiles, Data Policies, Entitlement Management, User Entitlements, Reports;
   - Documents;
   - the 8 create pages — ITAM, Application, Features, Actions, Data profile, Data policy, Entitlement, Account;
   - the pending-approvals page, if there is one;
   - opening a draft for editing.

## 1. Colour scheme

Use only these colours. They are defined as variables on `.hz` in Appendix A.

| Token | Hex | Used for |
|---|---|---|
| `--bg` | `#0F1A22` | Page background (dark navy) |
| `--surface` | `#111E27` | Key caps, deepest fills |
| `--card` | `#152733` | Card base |
| `--card2` | `#1B3140` | Raised fills: icon tiles, hover fill |
| `--border` | `#23394A` | Card borders |
| `--hair` | `#1E3240` | Hairline dividers |
| `--btn` | `#2B4556` | Input underline, key-cap borders, ticker dots |
| `--text` | `#E6EEF2` | Main text |
| `--text2` | `#A7B8C4` | Secondary text, module names at rest |
| `--muted` | `#86A0AE` | Labels, meta text, index numbers |
| `--faint` | `#6F8896` | Input placeholder |
| `--accent` | `#2DD4BF` | Teal: underlines, status dot, primary button |
| `--accent2` | `#5EEAD4` | Light teal: hover text, kicker labels, seconds, icons |
| `--on` | `#062A26` | Text on the teal button |
| `--blue` | `#60A5FA` | Second background glow only |
| `--draft` | `#F6C177` | Amber: anything that is a draft |
| `--pend` | `#C9B6FF` | Lavender: anything waiting for approval |
| Clock digits | `#EEF4F7` | Slightly brighter than text |

**Effects** (already in the CSS):
- **Background glow:** two blurred circles drift slowly behind everything.
  - Teal `#2DD4BF` at 16% opacity, top-left.
  - Blue `#60A5FA` at 12% opacity, bottom-right.
  - Both use `blur(60px)` and loop every 26s / 32s.
- **Preview panel:**
  - Background: a translucent gradient, `rgba(27,49,64,.78)` → `rgba(17,30,39,.72)`, with `backdrop-filter: blur(14px)`.
  - Corner radius 24px.
  - A 1px edge highlight from teal (top-left) to blue (bottom-right).
- **Selected search result:** `rgba(45,212,191,.06)` fill with a 2px teal bar on the left.
- **Ticker bar:** `rgba(11,20,27,.55)` with a hairline top border.

## 2. Type and motion

**Fonts:**
- Manrope for all text.
- JetBrains Mono (`.hz-mono`) for the date line, index numbers 01–06, tags, times, group labels, application codes and role names that are CODE_CASE.

| Element | Size / weight |
|---|---|
| Clock (HH:MM) | `clamp(84px, 8.4vw, 128px)` / 300, letter-spacing −0.055em |
| Seconds | 16px / 500, light teal, top-right of the minutes |
| Greeting | 30px / 300; the first name is 700 |
| Status line | 14px |
| Jump input | 22px / 300 |
| Module names | 30px / 300 |
| Result titles | 19px / 400 (CODE_CASE names: 16px mono) |
| Panel title | 26px / 700 |
| Panel kicker and section labels | 10.5px / 800, uppercase, letter-spacing .12–.14em |

**Motion.** All of it is already in the CSS — do not add JS animation. Easing is `cubic-bezier(.22,1,.36,1)`.

- **Page load:** each block rises 14px and fades in, staggered 90ms by `--i`. Module names stagger 60ms after a 300ms delay.
- **Clock:** the colon blinks every 2s. The seconds tick.
- **Jump input:** on focus, a teal underline sweeps in from the left (0.7s) and the ↑ ↓ ↵ key hints fade in.
- **Module hover:**
  - The other modules dim to 36%.
  - The hovered one moves 8px right and its teal underline draws in (0.5s).
  - Its index number turns teal and an arrow slides in.
- **Preview panel:** when its content changes, it re-enters (fade, 8px rise and 3px blur clearing, 0.55s). The large faint watermark changes too.
- **Ticker:** loops every 46s and pauses on hover.
- `prefers-reduced-motion` turns all of this off; the CSS handles it.

## 3. Layout

```
┌ main content (scrolls) ────────────────────────────────────────────────────┐
│ ░ teal glow                                                                │
│  FMCES · Wednesday 14 October · Region IN            ┌ preview panel ────┐ │
│  17:08 ⁴²   ← thin clock, seconds in teal            │ OVERVIEW       SH │ │
│  Good evening, Shashwat.                             │ Your workspace    │ │
│  ● Nothing needs you right now. 2 drafts are …       │ 2 │ 0 │ 68        │ │
│                                                      │ YOUR ACCESS · 4   │ │
│  🔍 Jump to anything…                     ↑ ↓ ↵      │ FMCES  CES App…   │ │
│  ─────────────────────────────────────────────       │ …                 │ │
│  01  App Management     8 Oct                        └───────────────────┘ │
│  02  Data Profiles      1 draft                          (sticky)          │
│  03  Data Policies      1 draft                                            │
│  04  Entitlements       68                                                 │
│  05  User Entitlements  8 Oct                                  ░ blue glow │
│  06  Reports                                                               │
├────────────────────────────────────────────────────────────────────────────┤
│ RECENTLY VISITED   App Management 8 Oct · 06:43 PM  •  Data Profiles …  ⟶  │
└────────────────────────────────────────────────────────────────────────────┘
```

**Grid:** `minmax(0,1fr)` | `minmax(320px,400px)`, with a 56px gap.
- Padding: 34px top, 44px right, 24px left.
- The ticker spans both columns and is pinned to the bottom of the page height.

**Responsive** (already in the CSS):

| Width | Layout |
|---|---|
| ≤1100px | Right column 320px, 36px gap, module names 26px |
| ≤860px | One column; the panel moves below the list and is not sticky |
| ≤760px | 16px side padding, clock 76px, module names 22px, result descriptions hidden |

## 4. Content and data

Appendix B shows every state filled with sample values. Replace the sample values with real data as follows.

**Meta line:** `FMCES · {weekday day month} · Region {region}`.
- Example: `FMCES · Wednesday 14 October · Region IN` (en-GB date format).
- Leave out "· Region …" if there is no region.

**Clock:**
- `HH` : `MM` in 24h, then seconds in `.hz-sec`.
- Update every second, and clear the timer on unmount.
- Set `aria-label="Local time HH:MM"`.

**Greeting:**
- Before 12:00 → "Good morning", before 17:00 → "Good afternoon", otherwise "Good evening".
- Then `, <b>{first name}</b>.`
- With no name: "Good evening."

**Status line (`.hz-calm`)** — the first sentence depends on the waiting count:

| Waiting = sum of pending counts | Dot | First sentence |
|---|---|---|
| 0 | `.hz-pulse` (teal) | `Nothing needs you right now.` |
| > 0 | `.hz-pulse is-pend` (lavender) | `{n} request(s) waiting for your approval.` + ` <a>Review →</a>` to the pending-approvals page (only if that route exists) |

Then add an `<em>` second sentence:
- [IF DATA EXISTS] drafts > 0 → `{n} drafts are where you left them.` For one draft: `1 draft is where you left it.`
- Otherwise, if there is a recent visit → `Last here {8 Oct}, {06:43 PM}.`
- Otherwise leave out the `<em>`.

**Module list:** always these 6, in this order. Index numbers are `01`–`06`.

The tag (`.hz-mod-x`) uses the first rule that applies, top to bottom:
1. Pending approvals for this module's types are above 0 → `{n} waiting` with `is-pend`.
   - Policies → Data Policies; Profiles → Data Profiles; Applications + Features + Actions → App Management.
2. [IF DATA EXISTS] drafts of this type are above 0 → `{n} draft(s)` with `is-draft`.
3. Entitlements only, [IF DATA EXISTS] → the entitlement count, e.g. `68`.
4. Last visit date of this module → `8 Oct`.
5. Otherwise no tag.

| # | Label | Opens | Description in the panel |
|---|---|---|---|
| 01 | App Management | App Management | Register ITAMs and applications, then define their features and actions. |
| 02 | Data Profiles | Data Profiles | Define which rows of a data source a profile covers. |
| 03 | Data Policies | Data Policies | Combine profiles and conditions into policies that limit data. |
| 04 | Entitlements | Entitlement Management | What users can do in each feature, and on which data. |
| 05 | User Entitlements | User Entitlements | Give users entitlements and manage their accounts. |
| 06 | Reports | Reports | Access and audit reports across FMCES. |

**Module panel (PANEL 2):**
- Kicker: `Module 0N`.
- Title and description from the table above.
- Watermark (`.hz-big`): `0N`.

The panel has **two key/value rows**. Take the first two that apply for that module:
- [IF DATA EXISTS] `Draft open` → `{latest draft name} · V{n}` with `class="is-draft"` (profiles and policies only).
- `Waiting for you` → the pending count for that module's types, or `Nothing` when it is 0.
- `In FMCES` → `{n} entitlements` (Entitlements only, [IF DATA EXISTS]).
- `Assigned to you` → `{n} roles` (User Entitlements only).
- `Last visit` → `8 Oct · 06:43 PM`, or `Not visited lately`.

**Quick actions** use `.hz-qb`:
- Icon tile: `+` for create actions, the pen for continuing a draft, the search icon for find actions.
- Bold title, then a `<small>` sub-line.

| Module | Quick actions (title — sub-line → target) |
|---|---|
| App Management | New application — Under an ITAM → create Application · New ITAM — Register an ITAM entry → create ITAM · New features — Add one or more → create Features |
| Data Profiles | [IF DATA EXISTS] Continue {latest profile draft} — {condition summary, or "Draft V{n} · edited {date}"} → edit that draft · New data profile — Data source and conditions → create Data profile |
| Data Policies | [IF DATA EXISTS] Continue {latest policy draft} — {condition summary, or "Draft V{n} · edited {date}"} → edit that draft · New data policy — Application, profiles, conditions → create Data policy |
| Entitlements | New entitlement — Name, data permissions, features → create Entitlement · Find an entitlement — Search by name → focus the Jump input |
| User Entitlements | New account — Give a user an entitlement → create Account · Find a user — Search by Bank ID → focus the Jump input |
| Reports | No quick actions: leave out the "Quick actions" label and list |

The footer has two buttons:
- `Open {module label}` (primary) → the module route.
- `Overview` (ghost) → back to PANEL 1.

**Overview panel (PANEL 1):**
- Kicker: `Overview`. Title: `Your workspace`.
- Description: `FMCES · Region {region}. Hover a module on the left to peek inside without leaving Home.`
- Watermark: the user's initials (e.g. `SH`).
- **Three numbers (`.hz-nums`).** Use the first three available, in this order:
  - [IF DATA EXISTS] `Drafts open` — add `is-draft` when above 0.
  - `Waiting for you` — add `is-pend` when above 0.
  - [IF DATA EXISTS] `Entitlements`.
  - `Roles`.
- **Your access:** label `Your access · {n} roles`, then one `.hz-role` per role:
  - The application code goes in `.hz-role-app` (mono).
  - The role name goes in `.hz-role-name`. Add `hz-mono` when the name is CODE_CASE (all caps with underscores).
  - Set `--i` to the row index.

**Jump to anything — result sources.** Show at most 6 results, ordered Record → User → Page → Create:
- **Page:** the 6 modules + Documents.
  - Sub-lines: Applications, features and actions · Profiles and their conditions · Policies, profiles and conditions · Entitlements and permissions · Accounts and assigned roles · Access and audit reports · Documents.
- **Create:** `New ITAM`, `New application`, `New features`, `New actions`, `New data profile`, `New data policy`, `New entitlement`, `New account`.
  - Sub-lines: Register an ITAM entry · Under an ITAM · Add one or more features · Add one or more actions · Data source and conditions · Application, profiles, conditions · Name, data permissions, features · Give a user an entitlement.
- **Record:** [IF DATA EXISTS]
  - With a search API: call it at 2+ characters, debounced 200ms, keeping the existing request shape.
  - Otherwise: match against the drafts and any records already loaded.
  - Sub-line: `{Type} · #{id}` plus ` · Draft V{n}` for drafts.
  - Use `hz-res-t hz-mono` for CODE_CASE names.
- **User:** [IF DATA EXISTS — a Bank ID lookup or filter] when the query is 5+ digits, show `User` · `{digits}` · `Open in User Entitlements`.
- **Matching:** case-insensitive substring over group + title + sub-line.
- **Group labels:** `Record`, `User`, `Page`, `Create`.

**Recently visited ticker:**
- Use the existing recent visits, newest first, at most 8.
- Each item is the page name plus `{8 Oct · 06:43 PM}` (12-hour time, two-digit hour).
- Render the list **twice in a row** inside `.hz-mq`. On the second copy, set `aria-hidden="true"` and `tabindex="-1"`.
- With fewer than 3 visits: render it once and add `is-static` to `.hz-mq`.
- With no visits: leave out the whole `<footer class="hz-marq">`.
- Clicking an item navigates to that page.

## 5. Behaviour

**State:**
- `now`: a Date that ticks every 1s.
- `query`: a string.
- `hi`: the highlighted result index.
- `peek`: −1 for the overview, or 0–5 for a module.
- `listAnim` and `peekAnim`: each is `'a'` or `'b'`.

**Replaying animations.** Use class swaps, not re-mounting or JS animation:
- When the left list switches between modules and results (query becomes empty ↔ non-empty), flip `listAnim`. The list element's class is `hz-list hz-{listAnim}`. The two classes run identical keyframes under different names, so swapping them replays the entrance.
- When `peek` changes, flip `peekAnim`. The panel content's class is `hz-pv hz-{peekAnim}`.

**Jump input** — keys handled only in the input's own keydown, with no global listeners:
- Typing sets `query` and resets `hi` to 0.
- ↓ / ↑ move `hi` and wrap around. Call `preventDefault`.
- Enter opens the highlighted result, then clears `query`.
- Esc clears `query`.

The results list:
- Hovering a result sets `hi`; clicking opens it.
- The highlighted row gets `is-on` and `aria-selected="true"`.
- With no matches, render only `<p class="hz-none">Nothing matches “{query}”.</p>`.
- "Opens" means: navigate to the route, start the create page, or open the record/draft the way the app does today.

**Modules:**
- `mouseenter` or `focus` on a module sets `peek` to its index — only when it differs, so the animation does not replay.
- `peek` does **not** reset on mouse leave, so the user can move into the panel and click its actions.
- The peeked module gets `is-on` (teal underline and index stay visible).
- Clicking a module navigates to it.
- The panel's `Overview` button sets `peek = −1`.
- "Find …" quick actions focus the Jump input.

**Accessibility:**
- The results container has `role="listbox"` and `id="hz-results"`, and each row has `role="option"`.
- The input has `aria-controls="hz-results"`.
- The right column has `aria-live="polite"`.
- Every interactive item is a real `<button>` or link with a visible focus ring; the CSS provides it.

## 6. Checks before you finish

1. Compare colours against §1. No other colours appear (no default blues, greys or whites).
2. The clock and module names render **thin** (Manrope 300 is loaded). The seconds are light teal and the colon blinks.
3. The greeting changes with the hour. The status line follows the §4 rules with 0 and with >0 pending approvals.
4. Focusing Jump draws the teal underline and shows ↑ ↓ ↵. Typing `pol` swaps the module list for results. Arrow keys, Enter and Esc work.
5. Hovering a module:
   - dims the others;
   - draws its underline and slides its arrow in;
   - makes the panel re-enter with the new content and watermark number.

   Moving the mouse into the panel keeps it, and `Overview` returns to PANEL 1.
6. The ticker loops seamlessly, pauses on hover and is hidden when there are no visits. The background glow drifts slowly behind the content.
7. Layout checks:
   - At 1440×900, everything fits without scrolling, with the ticker on the bottom edge.
   - At 1024×768, the right column is 320px.
   - At 390×844, there is one column, and the module names stay on one line.
8. With reduced motion turned on, nothing animates.
9. The header, tabs and dock are unchanged. There are no console errors, and the clock timer is cleared on unmount.

---

## Appendix A — `home-zen.css` (copy verbatim)

```css
/* FMCES Home — "Zen". Copy verbatim. Everything is prefixed hz- and scoped under .hz */
.hz{--bg:#0F1A22;--surface:#111E27;--card:#152733;--card2:#1B3140;--border:#23394A;--hair:#1E3240;--btn:#2B4556;--text:#E6EEF2;--text2:#A7B8C4;--muted:#86A0AE;--faint:#6F8896;--accent:#2DD4BF;--accent2:#5EEAD4;--on:#062A26;--blue:#60A5FA;--draft:#F6C177;--pend:#C9B6FF;--ease:cubic-bezier(.22,1,.36,1);--spring:cubic-bezier(.34,1.36,.64,1);--hz-top:96px;--hz-pl:24px;--hz-pr:44px;position:relative;min-height:calc(100vh - var(--hz-top));background:var(--bg);color:var(--text);font-family:'Manrope',system-ui,sans-serif;font-size:13px;line-height:1.45;-webkit-font-smoothing:antialiased;color-scheme:dark}
:where(.hz) *,:where(.hz) *::before,:where(.hz) *::after{box-sizing:border-box}
:where(.hz) button{font:inherit;cursor:pointer}
:where(.hz) :is(h1,h2,p){margin:0}
:where(.hz) a{text-decoration:none}
:where(.hz) :focus-visible{outline:2px solid #5EEAD4;outline-offset:2px}
.hz-mono{font-family:'JetBrains Mono',ui-monospace,monospace}
.hz-ico{display:block;flex-shrink:0;fill:none;stroke:currentColor;stroke-width:1.75;stroke-linecap:round;stroke-linejoin:round}
.hz-ico.bold{stroke-width:2.25}
@keyframes hz-rise{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:none}}
.hz-rise{animation:hz-rise .9s var(--ease) backwards;animation-delay:calc(var(--i,0)*90ms)}
@keyframes hz-breathe{0%,100%{box-shadow:0 0 0 0 rgba(45,212,191,.55)}60%{box-shadow:0 0 0 8px rgba(45,212,191,0)}}
@keyframes hz-breathe-p{0%,100%{box-shadow:0 0 0 0 rgba(201,182,255,.55)}60%{box-shadow:0 0 0 8px rgba(201,182,255,0)}}
.hz-pulse{width:8px;height:8px;border-radius:50%;background:var(--accent);animation:hz-breathe 2.4s ease-in-out infinite;flex-shrink:0}
.hz-pulse.is-pend{background:var(--pend);animation-name:hz-breathe-p}
.hz-kbd{display:inline-flex;align-items:center;justify-content:center;min-width:24px;height:24px;padding:0 6px;border-radius:6px;border:1px solid var(--btn);background:var(--surface);color:var(--text2);font:600 11px/1.5 'JetBrains Mono',monospace}

/* aurora background */
.hz-aur{position:absolute;inset:0;overflow:hidden;pointer-events:none;z-index:0}
.hz-aur i{position:absolute;border-radius:50%;filter:blur(60px)}
.hz-b1{width:620px;height:620px;left:-140px;top:-220px;background:radial-gradient(circle,#2DD4BF,transparent 65%);opacity:.16;animation:hz-drift1 26s ease-in-out infinite alternate}
.hz-b2{width:560px;height:560px;right:-120px;bottom:-200px;background:radial-gradient(circle,#60A5FA,transparent 65%);opacity:.12;animation:hz-drift2 32s ease-in-out infinite alternate}
@keyframes hz-drift1{to{transform:translate(240px,140px) scale(1.15)}}
@keyframes hz-drift2{to{transform:translate(-260px,-120px) scale(.9)}}

/* page grid */
.hz-grid{position:relative;z-index:1;min-height:calc(100vh - var(--hz-top));display:grid;grid-template-columns:minmax(0,1fr) minmax(320px,400px);grid-template-rows:1fr auto;column-gap:56px;padding:34px var(--hz-pr) 0 var(--hz-pl)}
.hz-left{min-width:0;display:flex;flex-direction:column;padding-bottom:28px}
.hz-meta{font-size:11.5px;color:var(--muted);letter-spacing:.04em}

/* clock + greeting */
.hz-clock{display:flex;align-items:flex-start;margin-top:10px;font-size:clamp(84px,8.4vw,128px);font-weight:300;line-height:.92;letter-spacing:-.055em;font-variant-numeric:tabular-nums;color:#EEF4F7}
.hz-col{margin:0 .02em;animation:hz-blink 2s steps(1) infinite}
@keyframes hz-blink{50%{opacity:.22}}
.hz-sec{margin:.2em 0 0 .3em;font-size:16px;font-weight:500;letter-spacing:0;color:var(--accent2)}
.hz-hello{margin-top:18px;font-size:30px;font-weight:300;letter-spacing:-.02em;color:var(--text2)}
.hz-hello b{font-weight:700;color:var(--text)}
.hz-calm{display:flex;align-items:center;gap:10px;margin-top:10px;font-size:14px;color:var(--text2)}
.hz-calm em{font-style:normal;color:var(--muted)}
.hz-calm a{color:var(--accent2);font-weight:700}
.hz-calm a:hover{color:#99F6E4}

/* jump to anything */
.hz-jump{position:relative;display:flex;align-items:center;gap:14px;height:52px;margin-top:34px;border-bottom:1px solid var(--btn);color:var(--muted)}
.hz-jump::after{content:"";position:absolute;left:0;right:0;bottom:-1px;height:1px;background:linear-gradient(90deg,var(--accent),var(--accent2) 60%,transparent);transform:scaleX(0);transform-origin:left;transition:transform .7s var(--ease)}
.hz-jump:focus-within::after{transform:scaleX(1)}
.hz-jump:focus-within{color:var(--accent2)}
.hz-jump input{flex:1;min-width:0;height:100%;padding:0;border:0;outline:0;background:transparent;color:var(--text);font:inherit;font-size:22px;font-weight:300;letter-spacing:-.01em}
.hz-jump input::placeholder{color:var(--faint)}
.hz-hint{display:flex;align-items:center;gap:6px;opacity:0;transition:opacity .3s}
.hz-jump:focus-within .hz-hint{opacity:1}

/* list area: replay the entrance by switching hz-a <-> hz-b */
.hz-list{margin-top:14px}
.hz-list.hz-a{animation:hz-la .5s var(--ease) backwards}
.hz-list.hz-b{animation:hz-lb .5s var(--ease) backwards}
@keyframes hz-la{from{opacity:0;transform:translateY(6px)}}
@keyframes hz-lb{from{opacity:0;transform:translateY(6px)}}

/* module list */
.hz-mod{position:relative;display:flex;align-items:baseline;gap:18px;width:100%;padding:7px 0;border:0;background:transparent;color:var(--text2);text-align:left;transition:opacity .35s,color .25s,transform .45s var(--ease);animation:hz-rise .8s var(--ease) backwards;animation-delay:calc(var(--i,0)*60ms + 300ms)}
.hz-mod-n{width:24px;font-size:11.5px;color:var(--muted);transition:color .25s}
.hz-mod-t{position:relative;white-space:nowrap;font-size:30px;font-weight:300;letter-spacing:-.025em;line-height:1.2}
.hz-mod-t::after{content:"";position:absolute;left:0;right:0;bottom:-2px;height:2px;border-radius:2px;background:var(--accent);transform:scaleX(0);transform-origin:left;transition:transform .5s var(--ease)}
.hz-mod-x{font-size:11px;font-weight:700;letter-spacing:.04em;color:var(--muted)}
.hz-mod-x.is-draft{color:var(--draft)}
.hz-mod-x.is-pend{color:var(--pend)}
.hz-mod-go{align-self:center;margin-left:-4px;color:var(--accent2);opacity:0;transform:translateX(-10px);transition:opacity .3s,transform .45s var(--ease)}
.hz-mods:hover .hz-mod{opacity:.36}
.hz-mods .hz-mod:hover,.hz-mods .hz-mod:focus-visible{opacity:1;color:var(--text);transform:translateX(8px)}
.hz-mod:hover .hz-mod-t::after,.hz-mod:focus-visible .hz-mod-t::after,.hz-mod.is-on .hz-mod-t::after{transform:scaleX(1)}
.hz-mod:hover .hz-mod-n,.hz-mod.is-on .hz-mod-n{color:var(--accent2)}
.hz-mod:hover .hz-mod-go{opacity:1;transform:none}
.hz-mod.is-on{color:var(--text)}

/* search results */
.hz-res{display:flex;align-items:center;gap:16px;width:100%;height:46px;padding:0 14px 0 0;border:0;border-radius:10px;background:transparent;color:var(--text2);text-align:left;transition:color .2s,transform .4s var(--ease),background-color .2s}
.hz-res-g{width:64px;flex-shrink:0;padding-left:14px;font-size:10.5px;font-weight:600;letter-spacing:.06em;text-transform:uppercase;color:var(--muted)}
.hz-res-t{font-size:19px;font-weight:400;letter-spacing:-.01em;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.hz-res-t.hz-mono{font-size:16px}
.hz-res-m{flex:1;min-width:0;font-size:12.5px;color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.hz-res-e{opacity:0;transition:opacity .2s}
.hz-res.is-on{color:var(--text);background:rgba(45,212,191,.06);transform:translateX(6px);box-shadow:inset 2px 0 0 var(--accent)}
.hz-res.is-on .hz-res-g{color:var(--accent2)}
.hz-res.is-on .hz-res-e{opacity:1}
.hz-none{padding:18px 0;font-size:15px;font-weight:300;color:var(--muted)}

/* peek panel */
.hz-right{min-width:0;padding-bottom:28px}
.hz-peek{position:sticky;top:10px;min-height:470px;margin-top:22px;padding:26px 26px 24px;border-radius:24px;border:1px solid rgba(43,69,86,.85);background:linear-gradient(160deg,rgba(27,49,64,.78),rgba(17,30,39,.72));-webkit-backdrop-filter:blur(14px);backdrop-filter:blur(14px);overflow:hidden;box-shadow:0 40px 80px -50px rgba(0,0,0,.9);animation:hz-rise .9s var(--ease) backwards;animation-delay:.35s}
.hz-peek::before{content:"";position:absolute;inset:-1px;border-radius:24px;padding:1px;background:linear-gradient(140deg,rgba(94,234,212,.55),transparent 35%,transparent 70%,rgba(96,165,250,.35));-webkit-mask:linear-gradient(#000 0 0) content-box,linear-gradient(#000 0 0);-webkit-mask-composite:xor;mask-composite:exclude;pointer-events:none}
.hz-big{position:absolute;right:14px;top:-6px;font-size:132px;font-weight:300;letter-spacing:-.06em;line-height:1;color:rgba(230,238,242,.04);pointer-events:none}
.hz-pv{position:relative}
.hz-pv.hz-a{animation:hz-pa .55s var(--ease) backwards}
.hz-pv.hz-b{animation:hz-pb .55s var(--ease) backwards}
@keyframes hz-pa{from{opacity:0;transform:translateY(8px);filter:blur(3px)}}
@keyframes hz-pb{from{opacity:0;transform:translateY(8px);filter:blur(3px)}}
.hz-kick{font-size:10.5px;font-weight:800;letter-spacing:.14em;text-transform:uppercase;color:var(--accent2)}
.hz-pv h2{margin-top:8px;font-size:26px;font-weight:700;letter-spacing:-.02em}
.hz-desc{margin-top:8px;font-size:13.5px;line-height:1.55;color:var(--text2)}
.hz-nums{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));margin-top:22px;border-top:1px solid var(--hair);border-bottom:1px solid var(--hair)}
.hz-nums div{display:flex;flex-direction:column;gap:2px;padding:14px 0 12px}
.hz-nums div+div{padding-left:16px;border-left:1px solid var(--hair)}
.hz-nums b{font-size:28px;font-weight:300;letter-spacing:-.03em;line-height:1.1}
.hz-nums b.is-draft{color:var(--draft)}
.hz-nums b.is-pend{color:var(--pend)}
.hz-nums span{font-size:11px;font-weight:600;color:var(--muted)}
.hz-sect{margin-top:20px;font-size:10.5px;font-weight:800;letter-spacing:.12em;text-transform:uppercase;color:var(--muted)}
.hz-roles{display:flex;flex-direction:column;margin-top:8px}
.hz-role{display:flex;align-items:center;gap:12px;padding:9px 0;border-bottom:1px solid rgba(30,50,64,.7);font-size:13px;font-weight:600;animation:hz-rise .6s var(--ease) backwards;animation-delay:calc(var(--i,0)*60ms + 150ms)}
.hz-role-app{width:46px;flex-shrink:0;font-size:10.5px;font-weight:600;color:var(--accent2)}
.hz-role-name{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.hz-role-name.hz-mono{font-size:11.5px}
.hz-kv{margin-top:20px;border-top:1px solid var(--hair)}
.hz-kv div{display:flex;justify-content:space-between;gap:16px;padding:11px 0;border-bottom:1px solid var(--hair);font-size:12.5px}
.hz-kv span{flex-shrink:0;white-space:nowrap;color:var(--muted)}
.hz-kv b{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;text-align:right;font-weight:600}
.hz-kv b.is-draft{color:var(--draft)}
.hz-qa{display:flex;flex-direction:column;gap:6px;margin-top:10px}
.hz-qb{display:flex;align-items:center;gap:12px;width:100%;padding:10px 12px;border-radius:12px;border:1px solid var(--hair);background:rgba(17,30,39,.6);color:var(--text);font-size:13px;font-weight:700;text-align:left;transition:border-color .2s,background-color .2s,transform .4s var(--ease);animation:hz-rise .55s var(--ease) backwards;animation-delay:calc(var(--i,0)*60ms + 180ms)}
.hz-qb:hover{border-color:#33566A;background:var(--card2);transform:translateX(3px)}
.hz-qb-ico{width:28px;height:28px;flex-shrink:0;display:flex;align-items:center;justify-content:center;border-radius:8px;border:1px solid var(--btn);background:var(--card2);color:var(--accent2)}
.hz-qb small{display:block;font-size:11.5px;font-weight:500;color:var(--muted)}
.hz-pfoot{display:flex;flex-wrap:wrap;align-items:center;gap:10px;margin-top:20px}
.hz-btn{display:inline-flex;align-items:center;gap:8px;height:38px;padding:0 14px;border-radius:10px;border:1px solid transparent;font-size:13px;font-weight:700;white-space:nowrap;transition:background-color .2s,border-color .2s,transform .3s var(--spring)}
.hz-btn:active{transform:scale(.97)}
.hz-btn-p{background:var(--accent);color:var(--on)}
.hz-btn-p:hover{background:var(--accent2)}
.hz-btn-g{background:transparent;color:var(--text2)}
.hz-btn-g:hover{background:var(--card2);color:var(--text)}

/* recently visited marquee */
.hz-marq{grid-column:1 / -1;display:flex;align-items:center;gap:18px;height:54px;margin:0 calc(var(--hz-pr) * -1) 0 calc(var(--hz-pl) * -1);padding:0 var(--hz-pr) 0 var(--hz-pl);border-top:1px solid var(--hair);background:rgba(11,20,27,.55)}
.hz-marq-lbl{flex-shrink:0;font-size:10.5px;font-weight:800;letter-spacing:.14em;text-transform:uppercase;color:var(--muted)}
.hz-track{flex:1;min-width:0;overflow:hidden;-webkit-mask-image:linear-gradient(90deg,transparent,#000 8%,#000 92%,transparent);mask-image:linear-gradient(90deg,transparent,#000 8%,#000 92%,transparent)}
.hz-mq{display:flex;width:max-content;animation:hz-marq 46s linear infinite}
.hz-mq.is-static{animation:none}
.hz-marq:hover .hz-mq{animation-play-state:paused}
@keyframes hz-marq{to{transform:translateX(-50%)}}
.hz-mit{display:flex;align-items:center;gap:10px;padding:0 22px;border:0;background:transparent;color:var(--text2);font-size:13px;font-weight:600;white-space:nowrap;transition:color .2s}
.hz-mit:hover{color:var(--accent2)}
.hz-mit .hz-mono{font-size:11px;color:var(--muted)}
.hz-mit::after{content:"";width:4px;height:4px;margin-left:12px;border-radius:50%;background:var(--btn)}

/* responsive */
@media (max-width:1100px){.hz{--hz-pr:28px}.hz-grid{grid-template-columns:minmax(0,1fr) 320px;column-gap:36px}.hz-mod-t{font-size:26px}.hz-hello{font-size:26px}}
@media (max-width:860px){.hz-grid{grid-template-columns:minmax(0,1fr)}.hz-peek{position:static;min-height:0;margin-top:0}}
@media (max-width:760px){.hz{--hz-pl:16px;--hz-pr:16px}.hz-grid{padding-top:20px}.hz-clock{font-size:76px}.hz-hello{font-size:22px}.hz-mod-t{font-size:22px}.hz-mod{gap:12px}.hz-res-m{display:none}}
@media (prefers-reduced-motion:reduce){.hz *,.hz *::before,.hz *::after{animation-duration:.01ms!important;animation-iteration-count:1!important;transition-duration:.01ms!important}}
```

## Appendix B — markup (all states, sample values)

The structure and class names are exact. Convert this to your framework's template syntax and keep every class and attribute.

- **STATE A and STATE B are alternatives.** Render the module list when `query` is empty, and the results when it is not.
- **PANEL 1 and PANEL 2 are alternatives** inside `.hz-peek`.
- **Sample values:** the text below shows sample values; replace them with the data from §4.
- **Classes that change with state:** `is-on`, `is-draft`, `is-pend`, `hz-a`/`hz-b`.

```html
<div class="hz">
  <div class="hz-aur" aria-hidden="true"><i class="hz-b1"></i><i class="hz-b2"></i></div>
  <div class="hz-grid">

    <section class="hz-left">
      <p class="hz-meta hz-mono hz-rise" style="--i:0">FMCES · Wednesday 14 October · Region IN</p>
      <p class="hz-clock hz-rise" style="--i:1" aria-label="Local time 17:08"><span>17</span><span class="hz-col">:</span><span>08</span><span class="hz-sec hz-mono">42</span></p>
      <h1 class="hz-hello hz-rise" style="--i:2">Good evening, <b>Shashwat</b>.</h1>
      <p class="hz-calm hz-rise" style="--i:3"><span class="hz-pulse"></span><span>Nothing needs you right now. <em>2 drafts are where you left them.</em></span></p>

      <label class="hz-jump hz-rise" style="--i:4">
        <svg class="hz-ico" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4"/></svg>
        <input type="text" placeholder="Jump to anything…" aria-label="Jump to a record, page or action" aria-controls="hz-results" autocomplete="off">
        <span class="hz-hint" aria-hidden="true"><span class="hz-kbd">↑</span><span class="hz-kbd">↓</span><span class="hz-kbd">↵</span></span>
      </label>

      <!-- STATE A — query is empty: the module list -->
      <nav class="hz-list hz-mods hz-a" aria-label="Modules">
        <button type="button" class="hz-mod" style="--i:0"><span class="hz-mod-n hz-mono">01</span><span class="hz-mod-t">App Management</span><span class="hz-mod-x hz-mono">8 Oct</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
        <button type="button" class="hz-mod" style="--i:1"><span class="hz-mod-n hz-mono">02</span><span class="hz-mod-t">Data Profiles</span><span class="hz-mod-x hz-mono is-draft">1 draft</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
        <button type="button" class="hz-mod is-on" style="--i:2"><span class="hz-mod-n hz-mono">03</span><span class="hz-mod-t">Data Policies</span><span class="hz-mod-x hz-mono is-draft">1 draft</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
        <button type="button" class="hz-mod" style="--i:3"><span class="hz-mod-n hz-mono">04</span><span class="hz-mod-t">Entitlements</span><span class="hz-mod-x hz-mono">68</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
        <button type="button" class="hz-mod" style="--i:4"><span class="hz-mod-n hz-mono">05</span><span class="hz-mod-t">User Entitlements</span><span class="hz-mod-x hz-mono">8 Oct</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
        <button type="button" class="hz-mod" style="--i:5"><span class="hz-mod-n hz-mono">06</span><span class="hz-mod-t">Reports</span><svg class="hz-ico bold hz-mod-go" width="20" height="20" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
      </nav>

      <!-- STATE B — query is not empty: results (replaces the module list) -->
      <div class="hz-list hz-b" id="hz-results" role="listbox" aria-label="Results">
        <button type="button" class="hz-res is-on" role="option" aria-selected="true"><span class="hz-res-g hz-mono">Record</span><span class="hz-res-t">Policy Versioning Testing</span><span class="hz-res-m">Data policy · #164 · Draft V4</span><span class="hz-res-e hz-kbd" aria-hidden="true">↵</span></button>
        <button type="button" class="hz-res" role="option" aria-selected="false"><span class="hz-res-g hz-mono">Record</span><span class="hz-res-t hz-mono">RATAN_ENTITLEMENT_RULE</span><span class="hz-res-m">Application · ITAM 51358 · 5 features</span><span class="hz-res-e hz-kbd" aria-hidden="true">↵</span></button>
        <button type="button" class="hz-res" role="option" aria-selected="false"><span class="hz-res-g hz-mono">Page</span><span class="hz-res-t">Data Policies</span><span class="hz-res-m">Policies, profiles and conditions</span><span class="hz-res-e hz-kbd" aria-hidden="true">↵</span></button>
        <button type="button" class="hz-res" role="option" aria-selected="false"><span class="hz-res-g hz-mono">Create</span><span class="hz-res-t">New data policy</span><span class="hz-res-m">Application, profiles, conditions</span><span class="hz-res-e hz-kbd" aria-hidden="true">↵</span></button>
        <!-- when nothing matches, render only this: -->
        <p class="hz-none">Nothing matches “xyz”.</p>
      </div>
    </section>

    <aside class="hz-right" aria-live="polite">
      <div class="hz-peek">
        <span class="hz-big hz-mono" aria-hidden="true">SH</span>

        <!-- PANEL 1 — Overview (peek = none) -->
        <div class="hz-pv hz-a">
          <p class="hz-kick">Overview</p>
          <h2>Your workspace</h2>
          <p class="hz-desc">FMCES · Region IN. Hover a module on the left to peek inside without leaving Home.</p>
          <div class="hz-nums">
            <div><b class="is-draft">2</b><span>Drafts open</span></div>
            <div><b>0</b><span>Waiting for you</span></div>
            <div><b>68</b><span>Entitlements</span></div>
          </div>
          <p class="hz-sect">Your access · 4 roles</p>
          <div class="hz-roles">
            <div class="hz-role" style="--i:0"><span class="hz-role-app hz-mono">FMCES</span><span class="hz-role-name">CES Application User</span></div>
            <div class="hz-role" style="--i:1"><span class="hz-role-app hz-mono">FMCES</span><span class="hz-role-name">CES Policy Admin</span></div>
            <div class="hz-role" style="--i:2"><span class="hz-role-app hz-mono">FMCES</span><span class="hz-role-name">IAM Admin</span></div>
            <div class="hz-role" style="--i:3"><span class="hz-role-app hz-mono">DMF</span><span class="hz-role-name hz-mono">DATA_MODEL_OWNER_ENTITLEMENT</span></div>
          </div>
        </div>

        <!-- PANEL 2 — Module peek (replaces panel 1; the watermark shows "03") -->
        <div class="hz-pv hz-b">
          <p class="hz-kick">Module 03</p>
          <h2>Data Policies</h2>
          <p class="hz-desc">Combine profiles and conditions into policies that limit data.</p>
          <div class="hz-kv">
            <div><span>Draft open</span><b class="is-draft">Policy Versioning Testing · V4</b></div>
            <div><span>Last visit</span><b>8 Oct · 06:43 PM</b></div>
          </div>
          <p class="hz-sect">Quick actions</p>
          <div class="hz-qa">
            <button type="button" class="hz-qb" style="--i:0"><span class="hz-qb-ico"><svg class="hz-ico bold" width="14" height="14" viewBox="0 0 24 24" aria-hidden="true"><path d="M4 20h4L19 9l-4-4L4 16zM13.5 6.5l4 4"/></svg></span><span>Continue Policy Versioning Testing<small>Booking_Location is any of KR, SG</small></span></button>
            <button type="button" class="hz-qb" style="--i:1"><span class="hz-qb-ico"><svg class="hz-ico bold" width="14" height="14" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 5v14M5 12h14"/></svg></span><span>New data policy<small>Application, profiles, conditions</small></span></button>
          </div>
          <div class="hz-pfoot">
            <button type="button" class="hz-btn hz-btn-p">Open Data Policies<svg class="hz-ico bold" width="14" height="14" viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg></button>
            <button type="button" class="hz-btn hz-btn-g">Overview</button>
          </div>
        </div>
      </div>
    </aside>

    <footer class="hz-marq" aria-label="Recently visited">
      <span class="hz-marq-lbl">Recently visited</span>
      <div class="hz-track">
        <div class="hz-mq">
          <!-- the visit list, rendered TWICE in a row (the loop moves by -50%) -->
          <button type="button" class="hz-mit">App Management<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit">Data Profiles<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit">Data Policies<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit">Entitlements<span class="hz-mono">8 Oct · 06:42 PM</span></button>
          <button type="button" class="hz-mit">User Entitlements<span class="hz-mono">8 Oct · 06:41 PM</span></button>
          <button type="button" class="hz-mit" aria-hidden="true" tabindex="-1">App Management<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit" aria-hidden="true" tabindex="-1">Data Profiles<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit" aria-hidden="true" tabindex="-1">Data Policies<span class="hz-mono">8 Oct · 06:43 PM</span></button>
          <button type="button" class="hz-mit" aria-hidden="true" tabindex="-1">Entitlements<span class="hz-mono">8 Oct · 06:42 PM</span></button>
          <button type="button" class="hz-mit" aria-hidden="true" tabindex="-1">User Entitlements<span class="hz-mono">8 Oct · 06:41 PM</span></button>
        </div>
      </div>
    </footer>

  </div>
</div>
```

**Icons** (24×24 stroke, `class="hz-ico"`, plus `bold` where shown):

| Icon | Path `d` |
|---|---|
| search | `M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14zM20 20l-4-4` |
| arrow | `M5 12h14M13 6l6 6-6 6` |
| plus | `M12 5v14M5 12h14` |
| pen (continue draft) | `M4 20h4L19 9l-4-4L4 16zM13.5 6.5l4 4` |
