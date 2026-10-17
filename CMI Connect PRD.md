# CMI Connect (IMC Connect) — Alumni Directory Redesign
### Product/Requirements Snapshot — as of 2026-10-15 (build `2026-10-15-19`)

This documents the current state of the **redesign-in-progress**, which lives
in `index-demo.html` (sandbox). The live, deployed site is `index.html` and
only receives changes from this file once a change is a confirmed functional
fix — never for look-and-feel experimentation. See `CLAUDE.md` for the
sandbox rule in full.

Live artifact (where visual edits are also made via "Send to Claude"
comments): https://claude.ai/artifact/YAALwaBNBTJKwFUJErCcX9

---

## 1. What this is

A single-file vanilla HTML/CSS/JS alumni directory for IMC/CMI students to
browse alumni, filter by industry/expertise, and "pin" people they want to
reach out to for coffee chats. Data is pulled live from a Google Sheet (CSV
export) via PapaParse, with a local CSV fallback for offline/`file://` use.

No build step, no framework, no backend — it's one HTML file.

## 2. Goals

- Let students quickly scan alumni and find people worth reaching out to.
- Make "who's available and how many spots they have left" visually obvious.
- Hand this off to the department in ~2 months with the directory in good
  shape and, ideally, some behavioral usage data already flowing (not yet
  implemented — see Open Items).

## 3. Design system

**Palette** (unified to one purple accent per a color-swatch reference the
user supplied mid-redesign — previously `#6346A2`, now `#401F68` everywhere):

| Token | Value | Use |
|---|---|---|
| `--paper` | `#F9F5FE` | Landing hero background (lavender wash) |
| `--cream` | `#FDFBF4` | Directory page background |
| `--ink` | `#000000` | Primary text |
| `--accent` / `--success` / `--success-on-light` | `#401F68` | Unified interactive purple — buttons, pinned state, links |
| `--accent-light` | `#DAD2F4` | |
| `--accent-tint` | `#EFEAFD` | |
| `--line` / `--line-strong` | `#E9E3FA` / `#DAD2F4` | Hairlines, dividers |
| `--muted` | `#5C5A82` | Secondary text, small-caps labels |
| `--warn` | `#E87518` | |
| `--pin-blue/pink/orange/violet` | various | Small pin-color accents only, never large surfaces |

**Typography**: Libre Baskerville / Cormorant Garamond / DM Serif Display for
display type, Inter for body/UI — loaded via Google Fonts.

**Interaction model**: purple is the one selection signal. "Pin" replaces the
old cart metaphor (`--success` naming is a holdover). A card is either
neutral or pinned/purple — no separate hover-highlight color language.

## 4. Landing page (hero)

- **One viewport, no scroll** by design (`height: 100vh` hard cap on
  `.hero-top`, reduced top padding on `.hero-copy`). This has been the
  single most fragile piece of CSS this project — see Open Items.
- **Nav row** (`.hero-nav`): logo + nav links, divider line raised slightly
  to balance against the rest of the hero (per explicit feedback that the
  nav felt oversized).
- **Decorative tower illustration** (`.hero-tower-decor`): a Gothic
  university tower + foliage image, anchored bottom-left, sized with
  `width: clamp(280px, min(46vw, 48vh), 760px)` so it shrinks on both narrow
  *and* short viewports (plain `vw` sizing alone let it collide with the
  "Find alumni" button on short/wide windows). Hidden entirely below
  768px — no room for it alongside the copy on mobile.
  - Image file: `Gothic University Tower Amidst Trees Extended.png` — an
    edited version of the user's original (`Gothic University Tower Amidst
    Trees.png`) with the foliage digitally extended further left (mirrored +
    feathered, via PIL/numpy) so the canopy reaches the hero's left edge
    without a hard crop line. The original file is untouched in the project
    folder for reference/rollback.
  - **Enlarged** (2026-10-15): width clamp increased from
    `clamp(280px, min(46vw, 48vh), 760px)` in stages, landing at
    `clamp(360px, min(62vw, 58vh), 1200px)` — the vh factor was pushed to
    72vh in an intermediate pass (to fix the image reading too small on
    large/tall screens, since a low vh cap means the image is governed by
    viewport *height*, not width) but that overlapped the "Find alumni"
    button once the hero copy block was also moved lower (see below) on
    short/wide windows, so it was pulled back to 58vh. Re-checked for
    overlap at several viewport sizes, including the short/wide case
    previously called out as the collision risk — button stays clear in
    all of them.
  - **Hero copy moved lower** (2026-10-15): `.hero-copy`'s top padding
    increased from `clamp(90px, 13vh, 140px)` to
    `clamp(110px, 17vh, 190px)` per request. Re-verified against the
    tower image above for overlap after this change (see previous bullet
    — the two changes interacted and needed a joint recheck).
  - **Doubled again** (2026-10-15): per explicit "twice as big" request,
    added a `@media (min-height: 760px)` override bumping the width to
    `clamp(500px, min(90vw, 100vh), 2000px)` — roughly double the
    previous size on typical laptop/desktop screens. Gated on height
    rather than changing the base rule outright, because a flat increase
    large enough to look "twice as big" on tall screens collided with the
    "Find alumni" button on short/wide ones; below the 760px threshold it
    falls back to the smaller, collision-safe `clamp(360px, min(62vw,
    58vh), 1200px)` from the previous pass. Checked at 1440×900,
    1920×1080, 2000×760, 1600×759/780/800 and 800×760 — no overlap at any
    of them, and the jump at the threshold reads as a reasonable size
    step rather than a glitch.
  - **Hero copy lowered again** (2026-10-15): a second "lower on the page"
    request. Pushing `.hero-copy`'s base padding-top further (to
    `clamp(130px, 21vh, 230px)`) collided with the tower at short/wide
    sizes (e.g. 1600×650), since it compounded with the already-enlarged
    tower from the previous pass. Switched to the same gating pattern as
    the tower: base padding-top stayed at the prior safe
    `clamp(110px, 17vh, 190px)`, with a `@media (min-height: 950px)`
    override pushing it to `clamp(140px, 20vh, 220px)` on tall screens;
    the tower's own tall-screen override (previous bullet) was retuned to
    the same 950px threshold at a smaller `65vh` factor (down from the
    760px/100vh pass) to leave enough headroom for the lower copy block.
    Note for future passes: a bounding-box overlap check (JS
    `getBoundingClientRect`) is *not* reliable for this image — the PNG
    has transparent margin above the visible spire, so bounding boxes can
    report overlap (e.g. −21px "gap") where the actual rendered pixels
    have a comfortable visual gap. Screenshots (zoomed on the button/tower
    junction) are the ground truth; verified clear at 1600×650 (base
    tier), 2400×949 (base), 2400×950/2400×1200/1920×1080 (tall tier), and
    1440×900, 800×700.
  - **Reverted to the bigger size, button made skinnier instead**
    (2026-10-15): per explicit request, the tower went back to the
    `clamp(500px, min(90vw, 100vh), 2000px)` formula from the "twice as
    big" pass (the 65vh-reduced version above is no longer used), and the
    collision was absorbed by the **"Find alumni" button** instead of the
    image: `.landing-option` shrank from `min-width:214px / height:54px /
    padding:0 22px` to `min-width:140px / height:44px / padding:0 16px`.
    Narrowing the button alone wasn't enough at common sizes like
    2000×900 (the tall spire sits well inside the button's horizontal
    span regardless of width) — shortening its *height* was what actually
    bought the clearance, since that directly lowers the button's bottom
    edge. Also had to raise the tower's enlarge-threshold back to
    `min-height: 950px` (from 760px): below ~950px tall the big tower's
    own top edge rises faster than the available headroom, so a
    760px-950px window produced a worse overlap than either side of that
    range (verified empirically — gap at the threshold is the local
    minimum, not a smooth transition, so the threshold itself has to
    clear it). `.hero-copy`'s tall-screen push (previous bullet) stayed
    at its existing 950px/`clamp(140px, 20vh, 220px)` override, unchanged.
    Checked (via real screenshots, not just bounding-box math — see the
    note above) at 1600×650, 2000×760/800/900/949/950, 2400×950/1200,
    1440×900, 1920×1080 and 800×700 — clear at all of them.
  - **Rebuilt as runtime JS sizing, not a CSS clamp** (2026-10-15): after
    this had gone through 4+ rounds of CSS clamp/media-query retuning and
    the user still saw it as "tiny" (their actual browser window was
    presumably shorter than whatever height threshold the clamp of the
    day assumed — there's no way to guess that from here), replaced the
    whole clamp/media-query system with `sizeTowerDecor()` (new script,
    near the other resize handlers): on load, on `resize` (debounced
    100ms) and once `document.fonts.ready` resolves, it measures the real
    gap between `window.innerHeight` and the "Find alumni" button's
    `getBoundingClientRect().bottom`, and sets the image's inline `width`
    to fill that gap (times the image's own 2303×793 aspect ratio),
    capped by `92vw` and a `2600px` ceiling, with a 28px margin above the
    button. This is now the actual mechanism — the CSS `clamp(360px,
    min(62vw, 58vh), 1200px)` left on `.hero-tower-decor` is only the
    pre-JS/no-JS first-paint fallback, not something to keep retuning.
    Deliberately has **no minimum-width floor**: an earlier version
    floored it at 320px and that floor silently overlapped the button on
    a short window (1600×650) where the computed safe width was smaller
    (264px) — removed the floor; a tiny image on an extreme window is
    correct degradation, not a bug. Verified at 1600×650 (gap lands
    exactly on the 28px margin, confirming the math), 1920×1080 (image
    ~1189px wide — clearly large, not "tiny"), and 800×700.
  - **Hero copy vertically centered on the page** (2026-10-15): per
    request, `.hero-copy` switched from in-flow/top-padding-pushed to
    `position: absolute; top: 50%; transform: translateY(-50%)` within
    `.hero-top` (which is `height: 100vh`, i.e. "the page" in this
    one-viewport design) — the whole headline-through-button block is
    now centered in the viewport rather than pinned near the nav. This
    replaced the old padding-top clamp entirely (no more
    `min-height: 950px` push-lower override needed). Mobile (<768px)
    reverts `.hero-copy` to `position: static` / normal in-flow stacking
    as before, with its own `heroCopyInMobile` keyframes (horizontal-only
    slide-in, since there's no vertical offset to preserve there) — see
    `@keyframes heroCopyIn` vs `@keyframes heroCopyInMobile`.
    `sizeTowerDecor()` (previous bullet) needed no changes — since it
    measures the button's real position at runtime, it automatically
    adapts to the button now sitting mid-page instead of near the top.
    Verified: at 1440×900 the copy block's own vertical center lands
    exactly on the viewport's center (measured via
    `getBoundingClientRect` — 450px on both, to sub-pixel precision), and
    1920×1080 / 375×812 (mobile, unaffected/still top-anchored) both
    checked visually.
  - **Doubled, overlap now accepted** (2026-10-15): explicit instruction
    that the tower overlapping the hero text/button is fine — "I'll deal
    with it if it happens." `sizeTowerDecor()` simplified accordingly: it
    still uses the button's position as a sizing reference (so the image
    scales sensibly with the page), but no longer subtracts a safety
    margin or treats the button's edge as a hard limit — the resulting
    width is doubled (`availableHeight * TOWER_ASPECT * 2`), capped by
    `130vw`/`3400px` instead of the old `92vw`/`2600px`. At 1440×900 this
    renders ~1462px wide (was ~522px pre-centering / sized to stay clear
    before), extending well above the button into the headline area, as
    requested. If a future request wants the overlap avoided again, that
    logic (margin subtraction + early-return guard) is in the git history
    of this file, not reproduced here.
  - **Hero copy raised to clear the tree canopy** (2026-10-15):
    `.hero-copy`'s `top: 50%` (vertical centering, see the "vertically
    centered on the page" entry above) changed to `top: 38%` so the
    "Find alumni" button sits just above the top of the visible tree
    branches instead of down inside the canopy, per explicit request -
    **the tower image itself was not touched/shrunk**; `sizeTowerDecor()`
    is unchanged, and since it sizes off the button's position, moving
    the button up if anything grows the image slightly (more vertical
    room above it), not less. Checked at 1440×900, 1920×1080 and 800×700
    - button clears the branches with a small margin at all three; the
    PNG's transparent padding above the visible canopy (see the earlier
    bounding-box-vs-visible-pixels note in this section) means the exact
    clearance has to be judged from a zoomed screenshot, not from
    `getBoundingClientRect()` numbers alone.

## 5. Alumni directory — split-view layout (2026-10-16 redesign)

**The directory is a split view now**, not a scrolling column of cards -
a structural redesign (explicit instruction: preserve all data,
filtering, search and selection logic; this changed *how it's laid
out*, not what it does or contains), built from a reference screenshot
("Chat Suggestion with Portaits.png" in the project folder - a
Figma-style mockup labeled "Split View Directory"). The card-grid layout
described in the rest of this section (below the next heading) is
**superseded** by this - kept for history/context, not as current state.

**Structure** (`#directorySplit`, replacing the old `.toolbar` + `<main>
id="grid">`):
- `.directory-left` (35% width, `border-right: 1px solid var(--line)`):
  - `.directory-left-header` - search box + filter pills (same
    `buildFilterPanel()`/`#filterPanel` as before, untouched) - **not**
    part of the scrolling region. This was deliberate: a filter
    dropdown's popover is `position:absolute`, and if its ancestor has
    `overflow-y:auto`, per the CSS overflow spec that ancestor also
    clips the *other* axis once either axis is non-`visible` - the
    popover would get cut off. Keeping the header out of the scrolling
    element sidesteps that entirely rather than fighting it with z-index
    or escape-hatch positioning.
  - `.directory-list` (`#directoryList`, `overflow-y:auto`) - the only
    part that scrolls independently. Compact entries
    (`.directory-entry`), one per alum: name (Baskerville, natural
    wrap - explicitly **no** truncation/ellipsis and no dynamic font
    shrink, per request) / role / company / location / capacity badge +
    chevron. **No pin/add button on list entries** - per the reference
    and the spec's own field list for this row, the add control lives
    only on the profile panel now. No photo either (portraits are
    profile-panel-only).
- `.directory-right` (65% width, `#directoryRight`, `overflow-y:auto`
  independently of the list) - the selected alum's full profile:
  `.profile-toolbar` (availability badge + add/pin button, upper-right;
  a "Back to directory" control that's desktop-hidden/mobile-only - see
  below) → `.profile-header` (portrait placeholder + name/role/company/
  location/LinkedIn) → `.profile-divider` → `.profile-sections`
  (Industry Expertise, Marketing Expertise, Ask me about - same
  `cardIndustryExpertiseColumnsHtml()`/topic-tag markup as the old
  cards, just reused in the new container).
- **Portrait placeholder** (`.profile-portrait`, 176×220px): a person-
  glyph SVG on a `var(--paper)` field with a `var(--line-strong)`
  border - explicitly a placeholder, no real images sourced or
  inserted. Sized/positioned so a real `<img>` could drop in later
  without a layout change.

**Rendering** (`renderGrid()` kept as the function name despite no
longer building a grid - every existing call site, roughly a dozen,
calls it, and renaming would've meant touching all of them for no
functional benefit):
- `renderGrid()` is now the orchestrator: recomputes `filtered`,
  rebuilds the list (`renderDirectoryList()`), makes sure
  `selectedAlumId` still points at something in the filtered set
  (defaults to `filtered[0]` otherwise - "first available alumnus",
  per request), then renders the profile panel (`renderProfilePanel()`).
- `selectAlum(id, {openMobileDetail})` - clicking a list entry calls
  this. It does **not** rebuild the list (only toggles `.selected` on
  the existing entries), so the list's scroll position survives
  selection, per explicit requirement. `openMobileDetail` is only
  passed from an actual entry click, never from `renderGrid()`'s own
  default-selection, so landing on the directory never auto-opens the
  mobile full-screen profile.
- `toggleCart()` no longer calls the heavy `renderGrid()` - since the
  add/pin button now only ever exists once (on the profile panel, for
  whichever alum is selected), pinning only needs
  `renderProfilePanel()` + `renderPinsDrawer()`, which also means
  pinning someone no longer resets the list's scroll position (a side
  effect fix, not just a perf one).
- Dead code removed as part of this rewrite (not left in place, since
  it sat directly in code being rewritten anyway):
  `splitCardName()`/the forced first-name-line-break + drop-cap
  treatment, and `hashId()`/`pinRotationFor()`/the per-card tilt
  effect - both were specific to the old `.alum-row` card and have no
  equivalent in the list/profile split. The old `.alum-row` family of
  CSS rules (`.alum-split` as a grid, `.alum-right-top`, `.card-action`
  flex-end placement reused above, etc.) was **not** hunted down and
  removed - left as unused/dead CSS given the scope already involved
  here; safe to prune later.

**Independent scroll heights** (`sizeDirectorySplit()`, mirrors
`sizeTowerDecor()`'s "measure, don't guess" approach): measures the
masthead's real rendered height and stores it as a CSS custom property,
`--masthead-h`, on `document.documentElement` - used two ways:
- **Desktop (≥1024px)**: `#directorySplit` gets an inline
  `height: calc(100vh - masthead)px`, so the two panels fill exactly
  the space below the nav, independently scrollable, no page-level
  scroll (nav stays visible, fixed).
- **Mobile/tablet (<1024px)**: `#directorySplit` height is cleared
  (`''`) - the list just flows with the page (natural scroll, no
  internal scroll region). The full-screen detail view
  (`.directory-right` when `.mobile-detail-open`) is `position:fixed`
  with `top: var(--masthead-h)`, *not* `position:absolute` against
  `.directory-split` - an earlier version used `position:absolute;
  inset:0` and the detail view rendered as a blank ~60px sliver,
  because `.directory-split`'s own height collapses to near-0 once
  `.directory-left` (its only in-flow child at that point) is
  `display:none` - an absolutely-positioned child can't stretch to fill
  a containing block whose own height is `auto`/collapsed.
  `position:fixed` sidesteps that by sizing against the viewport
  instead of the (collapsed) parent.

**Responsive breakpoint: 1024px**, distinct from the existing 700px
breakpoint used elsewhere (nav/hero mobile behavior) - chosen because
the spec specifically called out "1024px and wider" for the split view.
Below it: single-column directory, selecting an entry opens the profile
as a full-screen view with "← Back to directory" (`.profile-back-btn`,
desktop-hidden) instead of squeezing both panels side by side, per
explicit instruction not to do that.

**Verified**: desktop split view at 1440×900 and right at the 1024px
boundary (both the split and the narrower-but-still-split 1024px case,
names wrapping naturally in the tighter column as expected); tablet
width (900px) falls back to single-column as required; mobile
(375px) list → tap entry → full-screen detail → Back button → list
with scroll position and selection both intact; search, a live filter
toggle, the industry "+N more" popover, and pinning (badge updates,
Review and Submit drawer still correct) all re-checked working against
the new structure. No console errors at any of these.

### 5.1 Layout update (2026-10-17): shared toolbar, resizable divider, collapsible panel

Second reference image this round: "Chat Suggestion with Portaits -
Flow.png" (5-panel flow mockup: initial split view, panel collapsed,
adjustable column width, expanding again, mobile view). Explicit
framing: "a layout update, not a rebuild" - everything below changed
*arrangement*, not the data/filtering/selection logic underneath it
(all untouched from §5 above).

- **Shared toolbar** (`.directory-toolbar`): search box, all 5 filters,
  "Clear all", and Review & Submit now span the full page width *above*
  both panels, instead of search+filters living inside `.directory-left`
  with Review & Submit as a separate element. The old rotated vertical
  edge-tab version of `.pins-toggle-btn` is gone - it's an ordinary
  inline button now, `margin-left: auto` to sit at the toolbar's right
  edge, same `#pinsToggleBtn` id and click handler as before (unchanged
  JS wiring, only the CSS/position changed). `#statusArea` and
  `#pinCapNotice` sit just below the toolbar, above the split.
- **`{N} alumni` count** (`#directoryCount`, in a new
  `.directory-list-header` row above the list) - added because the
  reference shows it; the count itself is just `filtered.length`, not
  new data. The reference also shows a "Sort by: Relevance" control next
  to it - **deliberately not implemented**, since there's no relevance
  score or sort logic anywhere in this data model and the brief said not
  to add new functionality; inventing a sort order wasn't a safe
  judgment call to make silently.
- **Resizable divider** (`.directory-resizer`, `initDirectoryResizer()`):
  a 10px-wide drag handle between the two panels. Pointer events (not
  separate mouse/touch handlers) so the same code drags on trackpad and
  touchscreen. Sets `#directoryLeft`'s inline `flex-basis` directly on
  drag - clamped between its own `min-width` (280px, in CSS) and
  `(split width − 320px − 10px)` so the profile panel can never be
  squeezed thinner than its own min-width. Arrow-key nudging on the
  resizer too (`role="separator"`, 24px per press). The chosen width
  persists on its own with no extra state-tracking needed: re-rendering
  the list or profile only replaces `#directoryList`'s/`#directoryRight`'s
  *innerHTML*, never `#directoryLeft` itself, so its inline style simply
  survives every re-render, filter change, and collapse/expand - "preserve
  panel width" fell out of the existing render architecture for free
  rather than needing to be deliberately engineered.
- **Collapsible profile panel**: an "×" button
  (`#profileCollapseBtn`, next to the existing add/pin button in
  `.profile-toolbar`) adds `.profile-collapsed` to `#directorySplit`,
  which hides `.directory-right`/`.directory-resizer` and lets
  `.directory-left` fill the full width via flex. A floating "›"
  (`#directoryReopenBtn`, `position:absolute` on the split's right edge,
  vertically centered) reopens it. Clicking any directory entry while
  collapsed also reopens the panel (`selectAlum`'s new `reopenProfile`
  option, only passed from an actual entry click - never from
  `renderGrid()`'s own default-selection, so loading/filtering the
  directory never force-opens the panel on its own). Selection and
  divider width are both untouched by collapsing - see above, same
  reasoning.
  - **Not implemented**: the reference's "columns for alumni identity,
    role/company, and location/availability" in the expanded directory
    reads, in the actual mockup pixels (not just the text brief), as the
    *same* stacked row just given more breathing room at full width,
    not a distinct 3-column grid - the existing `.directory-entry`
    layout already does this naturally once it's wider. Flagged here in
    case a literal 3-column grid turns out to be what was actually
    wanted on closer look.
- **Portrait/text stacking at narrow widths** (`.directory-right.narrow`,
  `watchProfilePanelWidth()`): a `ResizeObserver` on `#directoryRight`
  (not a CSS media query) toggles `.narrow` below 420px wide, switching
  `.profile-header` from a row to a column. A `ResizeObserver` rather
  than a viewport media query specifically because the panel's width
  changes from *dragging the divider*, independent of the viewport -
  a media query has no way to see that.
- **`sizeDirectorySplit()` reworked**: previously measured
  `header.site`'s height and called that "where the content starts".
  With the toolbar (plus status/pin-cap banners) now also stacked above
  the split, summing each piece's height by hand would've been exactly
  the kind of brittle guess this project has repeatedly hit before -
  instead it now reads `#directorySplit.getBoundingClientRect().top`
  directly, which is correct regardless of what's stacked above it.
  Stored as `--content-top` (renamed from `--masthead-h`, since it's no
  longer just the masthead); the mobile full-screen detail view's
  `position:fixed; top: var(--content-top)` picks up the same number.
- **Location filter flyouts now open horizontally** (separate explicit
  request this round, not from either mockup): each region's
  (`United States`/`Asia`/`Latin America`) nested checkbox list opens as
  a flyout to the *right* of its row (`position:absolute; left:100%`)
  instead of expanding below it. Closed-state chevron now points right,
  rotating to point down when open, signaling "opens sideways." Needed
  `.filter-dropdown-panel-grouped{ overflow: visible }` (overriding the
  base panel's `overflow-y:auto`) - without it the flyout was clipped,
  because per the CSS overflow spec, an ancestor with `overflow-y:auto`
  also clips the x-axis once either axis is non-`visible`; same bug
  class as a popover-clipping issue fixed earlier this project, same
  fix shape.
- **Verified**: desktop at 1440×900 and 1600×900 - toolbar/count/resizer
  all correct; drag-resize in both directions (including dragging the
  profile panel narrow enough to trigger the portrait/text stack);
  collapse → directory fills width, reopen chevron appears → reopen
  restores the same selected alum; Location's horizontal flyout opens
  uncropped. Mobile (375px): toolbar wraps, Review & Submit reads as a
  full-width button (not the old edge tab), tapping an entry still opens
  the full-screen detail view correctly, collapse/resizer controls
  correctly absent (desktop-only by design). No console errors at any
  of these.

### 5.2 Alumni data source switched: "For Pilot" → "For Launch"

**Done** (2026-10-17) - the user supplied the "For Launch" tab's
published-CSV URL (gid `1834383749`, same spreadsheet, was gid
`1298862545`/"For Pilot"); `SHEET_CSV_URL` updated to it.

**Verification note for future reference**: neither this session's
built-in browser pane nor the published Artifact's own sandboxed iframe
could actually complete the live `Papa.parse(SHEET_CSV_URL)` fetch
against the new URL in testing - both silently fell back to the bundled
`alumni-data.csv` snapshot (by design, see the `error:` handler in
`loadData()`), landing on two *different* stale counts (37 locally, 47
on the published Artifact) that reflect whatever each fallback snapshot
happened to contain, not the real tab. This is the same documented
sandboxing limitation the code's own comments already call out for
Artifact previews - not a new problem from this change, and not
something that affects a real visitor's ordinary browser.
The URL itself was independently confirmed valid and correct via a
direct fetch from outside any browser sandbox: HTTP 200, CSV
structure intact (`Timestamp, Name, Current company...` header), 82
data rows (vs. the Pilot tab's 47) starting with the same
`Koushank Ahuja` / `Jasmine Chao` / ... — consistent with "For Launch"
being the Pilot responses plus additional ones collected since, not a
different unrelated sheet. The real deployed site, opened in an
ordinary (non-sandboxed) browser, reaches Google Sheets the same way the
Pilot tab already did and should load the new tab's live data normally -
this is the gap this project's own fallback mechanism exists to paper
over specifically for sandboxed preview contexts.

## 5a. Alumni directory — card layout (superseded, kept for history)

One alumnus per card (changed from an earlier 2-column grid, which caused
inconsistent name-wrapping between short and long names).

Each card (`.alum-row` → `.alum-split`):
- **`.alum-left`**: name, role, company, city, LinkedIn — always left,
  always this content only.
- **`.alum-divider`**: 1px vertical rule separating the two halves (becomes
  a horizontal rule on mobile, where the split stacks).
- **`.alum-right`**: everything else — capacity badge, pin/plus button,
  Industry + Marketing Expertise columns, "Ask me about."

**Capacity badge** (`capacityBadgeHtml()`): a small person-pair icon +
pill reading "N spots available" (or similar), styled per a reference image
the user supplied. Icon recolors to white when the card is pinned (purple
background). Sits immediately to the left of the "+" button (2026-10-15 -
`.card-action` changed from `justify-content: space-between`, which put
the badge at the far-left edge of the whole right-hand column, to
`justify-content: flex-end`, clustering badge + button together at the
right).

**Badge colors folded in from the reference image** (2026-10-15): the
3-tier colors (`.capacity-badge-open/-low/-full`) were replaced with
hex values sampled pixel-by-pixel from the reference screenshot (the
same one `capacityBadgeHtml()` above was already styled after, now with
exact values instead of an eyeballed approximation):
- `open` (the reference's High + Moderate tiers share one treatment):
  `background: #E2E5DA; color: #535C55` (was `#E3F5EA` / `#1F7A46` —
  a brighter green)
- `low`: `background: #EEE6DA; color: #7C6B5C` (was `#FDEBDD` /
  `var(--warn)` — an orange-tinted pill)
- `full`: `background: #E2DFDC; color: #747681` (was
  `var(--accent-tint)` / `var(--muted)` — a purple-tinted pill)

Scoped to just the capacity badge — the reference's purple accent
(`#45246F`) was close enough to the site's existing `--accent: #401F68`
that it wasn't touched, and the reference's cream page background
(`#F6F4EC`) wasn't folded in since that would mean replacing the site's
whole `--paper`/`--cream` background system, a much bigger change than
"fold the color scheme in" was read to ask for here — flag if that's
wanted too.

**Pinning**: clicking the plus button highlights the card purple; no
separate pin-icon mark is shown on the card itself (removed per explicit
feedback — "highlight the card purple but do not add the pin icon").

**"Please rank your top 5" drawer** (2026-10-15 redesign — the drawer
opened by the "Review and Submit" tab): header text changed from "My
Pins"/"Review your pins" (it used to say one or the other depending on
list-vs-review mode) to a single constant "Please rank your top 5",
regardless of mode, matching the button's own renamed label. The
per-alum card inside it (`.review-card`, `renderPinCardsInto()`) was
rebuilt to match the main grid card's content, minus the parts this
context doesn't need:
- No pin icon any more (`.review-card-pin` and the thumbtack image are
  gone from this template - `pinHueClassFor()`/`PIN_HUE_CLASSES` were
  only used there and were removed as dead code; `hashId()`/
  `pinRotationFor()` are still used for the main grid's card tilt).
- No topics list (`.review-card-topics` removed).
- Added: location + LinkedIn line (reuses `cardContactLineHtml()` and
  `.alum-linkedin`, the same pieces the main grid card uses).
- Name and the capacity badge ("N spots available") sit on one row,
  badge to the right of the name (`.review-card-header`,
  `justify-content: space-between`).
- The remove ("×") button is `position:absolute` top-right, but now
  sits in **its own row above** the name/badge row rather than beside
  it - `.review-card` padding-top bumped to `46px` (from `20px`) to
  clear the button's height, since putting it beside a now-2-line name
  (see below) made the first line of the name crowd right up against
  it. (The tab's own pin icon and color were already changed in an
  earlier pass the same day - see further down.)

**Industry/Marketing Expertise + field labels revised** (2026-10-15):
- `.field-label` (Industry/Marketing Expertise, Ask me about headers)
  reverted to Inter sans-serif (was briefly Baskerville, per a request
  earlier the same day, reverted the same day — "sans serif again
  sorry").
- Industry Expertise no longer shows a single primary value; it now
  renders as a `.topic-tags industry-tags` pill row, same mechanism as
  Marketing Expertise and Ask me about (`collapsePillRowOverflow`,
  called for `.industry-tags` from `collapseAskMeAboutOverflow`) — so
  it collapses to 1 line + a "+N more" popover the same way. Replaces
  the old primary-industry-plus-popover pattern
  (`primaryIndustryOrderedList()` is now consumed directly as the pill
  list; `.alum-col-value` is unused/removed).
  - `.topic-tag` no longer gets a bold weight — matches Marketing's
    treatment now that both are pills.
- Industry Expertise and Marketing Expertise sit side by side as two
  columns again (`.alum-columns{flex-direction:row}`) — "Ask me about"
  is a separate block rendered after `.alum-columns` in the template, so
  it's unaffected and stays full-width below both columns.
  - Mobile (≤700px) reverts to stacked columns
    (`.alum-columns{flex-direction:column}` in the existing mobile media
    query) — two 140px-min columns side by side would immediately
    collapse both pill rows to "+N more" on a phone-width card.
  - Bug caught while building this: `.alum-col{flex:1 1 140px}` sets
    140px as the flex-basis on whichever axis is the *main* axis for the
    parent — 140px-wide on desktop (row), but 140px-*tall* once mobile
    switches the parent to column, which left a ~140px dead gap under
    Industry on phones. Mobile override now also sets
    `.alum-columns .alum-col{flex:1 1 auto}` to cancel that.
- `.topic-tag` filled with light lavender (`var(--accent-tint)`,
  `#EFEAFD`) per the reference screenshot, replacing the old
  outlined/unfilled pill (`border: 1px solid var(--line-strong)`, no
  fill). Pinned-card state changed from a border-color override (there's
  no border to color any more) to `background: rgba(255,255,255,0.18)`
  so pills stay visible against the solid-purple pinned card.
  `white-space:nowrap` + `text-overflow:ellipsis` + `max-width:100%`
  added so a single long value (e.g. a long industry name) truncates
  instead of wrapping inside the pill and forcing the row to 2 lines —
  the narrower 2-column layout made this an actual problem where it
  wasn't before.

**"My Pins" → vertical "Review and Submit" tab** (2026-10-15): moved out
of the search-row toolbar entirely, per request — now a fixed-position
element (`#pinsToggleBtn`, sibling of `.toolbar`, not inside it) rotated
-90° to read as a vertical tab docked to the right edge of the viewport,
vertically centered on it:
```css
.pins-toggle-btn{
  position: fixed; bottom: 50%; right: 0;
  transform-origin: bottom right;
  transform: rotate(-90deg) translateX(50%);
}
```
`position:fixed` means it stays put as the alumni grid scrolls under/
past it, rather than scrolling away with the toolbar. Renamed "My
Pins" → "Review and Submit" (button label only — the drawer it opens
still internally calls itself "My Pins"/"Review your pins", untouched).
Getting the rotation to dock flush against the edge *and* center
vertically needed both the `transform-origin: bottom right` (pivots
around the point that's already correctly positioned by
`bottom:50%;right:0`) and the `translateX(50%)` correction (without it,
the tab's length extends entirely to one side of center rather than
being centered on it) — arrived at empirically via
`getBoundingClientRect()` checks, not derived analytically; if this
needs further adjustment, measure again rather than assuming the
geometry. Pin icon removed (text-only now) and the button recolored to
solid purple/white (`background: var(--accent)`, `color: #fff` — was a
white/outlined pill) per follow-up request the same day; hover darkens
to `--accent-deep` instead of the old border/text color swap, and the
"active" (filter-toggled) state still overrides to `--success-on-light`
as before.

**Section headers** (`.field-label` — "Industry Expertise", "Marketing
Expertise", "Ask me about"): briefly set in Baskerville on 2026-10-15,
reverted the same day ("sans serif again sorry") — back to Inter,
weight 500, still uppercase/small-caps/letter-spaced.

**Card name now 2 lines, location/LinkedIn centered against "Ask me
about"** (2026-10-15): `.alum-name` reserves 2 lines again (was
single-line + ellipsis), split explicitly in the template — first name
always line 1, everything else always line 2 (`splitCardName()`,
first-space split), rather than left to wrap naturally, so the break
point doesn't drift with column width or font metrics. Note this
reverts yet undoes an *earlier* reversion in this same project's
history (there was a natural-wrap 2-line version before the single-line
one this replaces — see git history if that behavior is ever wanted
back instead of the hard first-name split used now).

**Drop-cap experiment** (2026-10-15, explicitly framed as "could we
experiment with"): `.alum-name::first-letter{ float:left; font-size:
1.9em; ... }` — floats the first letter large, which (as a side effect
of how CSS floats interact with the rest of the block) also pushes line
2 to the right, not just line 1, since a float affects every subsequent
line in its containing block, not just the one it's on. No color
override, so it still follows the pinned/white-text state via
`currentColor`. Flagged as experimental in the CSS comment too - revisit
if it doesn't hold up once there's a full page of real names to look at
together, rather than assuming this one is the final answer.

To get the location/LinkedIn line vertically centered against "Ask me
about" (header + pills, as one block) on the *other* side of the
divider, `.alum-split` changed from `display:flex` to `display:grid`
with named `grid-template-areas` (`info`/`contact`/`divider`/`top`/
`bottom`) — the `.alum-left`/`.alum-right` wrapper divs are gone, their
former children (`.alum-info`, `.alum-contact-line`, `.alum-divider`,
a new `.alum-right-top` wrapper holding the badge/plus/Industry/
Marketing, and `.ask-me-about`) are now direct grid children of
`.alum-split`, placed by area name. This matters because CSS Grid
shares row *heights* across columns natively - `.alum-contact-line` and
`.ask-me-about` sit in the same grid row ("contact"/"bottom" areas,
same row track) with `align-self:center`, so the row auto-sizes to the
taller of the two (almost always `.ask-me-about`) and the shorter one
centers inside it for free. Flexbox has no equivalent for this across
two independent column flex-items; that's the specific reason this
needed to become a grid. `.alum-right-top` keeps the same
`position:relative` role the old `.alum-right` had, so the badge/plus
button's `position:absolute` top-corner placement is unaffected.
Mobile (≤700px) swaps in a 1-column `grid-template-areas` (same 5
children, stacked in reading order) rather than touching DOM order.

**Card divider bug + Industry Expertise reverted to plain text**
(2026-10-15, same-day follow-up to the grid change above): the vertical
divider between the two columns had gone invisible in the grid
rewrite - `.alum-split` uses `align-items: start` (so content sizes to
itself instead of stretching), but that also meant `.alum-divider`
(an empty div with only `width:1px`) collapsed to 0 height with nothing
to give it height. Fixed with `.alum-divider{ align-self: stretch; }`,
overriding the container's `start` just for this one item.
Separately, Industry Expertise went back to plain text (only the
`industryColHtml` output changed back - the pill-row version from
earlier the same day, with its `.industry-tags`/`collapsePillRowOverflow`
wiring, was undone): shows the primary industry as text, with a
"+N more" *pill-button* (not a `.topic-tag`) for anything beyond it -
same `industry-more-btn`/popover pattern this used before the pill-row
version. `.alum-col-value` (the primary-industry text style) was
restored at `font-weight: 400` (not the `600` it had before any of
today's changes - "need not be boldened" from the earlier request still
applies). No character limit/truncation on the value itself - a long
industry name is allowed to wrap and grow the card, per explicit
"feel free to increase the height of the alumni cards if needed."
Industry/Marketing Expertise's vertical position also moved down to
align with the role/title line (not the name) on the left - now that
the name reserves a fixed 2 lines, that offset is a fixed `69px`
(measured via `getBoundingClientRect`, not computed from font metrics
by hand) rather than needing to react to variable name height.

**Location filter** (2026-10-15 redesign): restructured from one flat
checkbox list of cities into 3 fixed top-level regions — United States,
Asia, Latin America — each a clickable header whose nested city/country
checkboxes stay hidden until clicked (`buildLocationRegionGroup()` in
the directory JS, `.location-region*` CSS). Region classification
(`regionForCity()`) reads the ", Country" suffix `normalizeCityValue()`
already leaves on non-US entries, checked against `ASIA_COUNTRIES` /
`LATAM_COUNTRIES` lists, plus a small `CITY_REGION_OVERRIDES` map for
well-known non-US cities written without a country (e.g. "Singapore",
"Tokyo"). **Anything unmatched — a bare US city, or a country outside
those two lists (e.g. Canada, UK) — defaults to "United States"**, since
that's the de facto catch-all bucket for this 3-option structure, not a
literal claim those places are in the US. Revisit if alumni outside
those three regions become common. Current live data is 4 US cities
only (Atlanta, Chicago, New York, San Francisco); Asia and Latin America
show "No alumni listed here yet." until real data populates them — this
is speculative/forward-looking scaffolding, not yet exercised by real
non-US data.

**Logo font** (2026-10-15): "CONNECTS" in the `IMC | CONNECTS` logo
switched from Baskerville to Inter per request; "IMC" stays Baskerville/
purple. Applies everywhere the logo appears (landing + masthead).

**Filter pill buttons → Baskerville** (2026-10-15): `.filter-dropdown-
toggle` (the Industry/Marketing Area/Conversation Topics/Location/Brand
vs Agency pills themselves) changed from `font-family: inherit` (Inter)
to Baskerville. Scoped to just the toggle buttons - everything inside
the opened panels (checkbox labels, the Location region headers) stays
Inter, per explicit "keep the filter contents inter."

## 6. Data

- Primary source: Google Sheet, CSV-exported, loaded via PapaParse.
- Local fallback: base64-embedded CSV baked into the file, for when it's
  opened directly as `file://` (CORS blocks the live fetch in that case).
- `alumni-data.csv` / `live_sheet*.csv` in the project root are working
  snapshots/exports used during development — not loaded by the page at
  runtime unless explicitly wired in.
- An in-page update-checker compares a `BUILD_VERSION` string embedded in
  the page against the currently-published version, to warn visitors on
  stale cached copies that a newer deploy exists (this exists on the
  roadmap toward `index.html`, not just the demo).

## 7. Known fragile points / open items

- **Auto-reply pipeline collisions**: leaving a comment on the published
  artifact while also editing via chat triggers two independent edit/publish
  flows that can conflict and even silently revert each other's changes
  (most notably, the hero's no-scroll CSS has been reverted and re-applied
  several times this way). Recommended going forward: use one channel per
  change, not both.
- **Hero no-scroll constraint is not self-documenting** — it was requested
  in chat, not as a durable comment/spec on the artifact, so a future
  editing pass (automated or human) could reintroduce scrolling without
  realizing it's a hard requirement.
- **Analytics/behavioral tracking**: not yet implemented. Options discussed
  (GA4 vs. a custom logging sheet) but no decision made yet — needed before
  the ~2-month handoff if behavioral data is a goal.
- **Tower image enlargement**: done (see §4) — image is ~20% larger,
  verified clear of the "Find alumni" button across viewport sizes.

## 8. File map (project root, relevant files only)

| File | Purpose |
|---|---|
| `index-demo.html` | **Active sandbox** — all visual/layout work happens here |
| `index.html` | Live deployed site — only functional fixes, needs explicit go-ahead |
| `Gothic University Tower Amidst Trees.png` | Original hero illustration (untouched) |
| `Gothic University Tower Amidst Trees Extended.png` | Edited version currently in use on the hero |
| `alumni-data.csv`, `live_sheet*.csv` | Data snapshots/exports |
| `Chat Suggestion with Portaits.png` | Reference mockup for the split-view directory redesign (§5) |
| `Chat Suggestion with Portaits - Flow.png` | Reference mockup for the toolbar/resizer/collapse layout update (§5.1) |
| `CLAUDE.md` | Working rules for this repo (sandbox rule, preview-pane behavior) |
| `Archive/` | Old CMI Connect site — not affected by any of the above |
