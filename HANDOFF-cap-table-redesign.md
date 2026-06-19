# Handoff — Cap-table redesign (Option A list rows)

> Session memory for callback. Captures what was changed, why, and where, so a
> future chat can pick up without re-deriving context.
> Date: 2026-06-18 · Owner: farheen@equitylist.co
> Sessions: §1–9 = cap-table redesign (Option A). §10+ = manual-Calculate gating,
> git state, dynamic round-name labels, and investigations (later same day).

---

## 1. Goal of the work

Redesign the **Current cap table** step of the SAFE dilution calculator. The
original used tall 2-column **cards** per shareholder (wasted vertical space). A
plain table felt like a spreadsheet, buried the ownership %, and handled long
names badly. We replaced it with a quiet **inline list-row** pattern ("Option A")
that keeps table density without the spreadsheet feel.

Constraints that drove the design:
- Most users have 2–10 shareholders (scale to ~20).
- Categories: Founder, Investor, Option pool, Other.
- Shareholder names can be very long → must truncate gracefully.
- This is a lightweight step inside a calculator, NOT a Carta-style product.

---

## 2. Repos / files (IMPORTANT — two different things)

| Path | What it is | Status |
|---|---|---|
| `/Users/farheenshaikh/safe-updated-ui/index.html` | **THE REAL PRODUCT.** Single ~5,000-line hand-coded HTML/CSS/JS file. All shipped changes are here. | ✅ edited |
| `/Users/farheenshaikh/design_update_safe/cap-table-option-a.html` | Standalone vanilla mockup used to prototype Option A. Reference only. | unchanged reference |
| `/Users/farheenshaikh/design_update_safe/src/*` (React) | A *separate* React "results column" mockup. NOT the product. Ignore for this work. | n/a |

The real product is **plain HTML**, not React. Don't look for components/JSX.

---

## 3. Design decision (Option A) — the "why"

- **List rows, one per shareholder** (not cards, not a 2-col grid). Gives ~44px
  rows vs ~180px cards, and one scannable column of names + one of %.
- **Reuse the app's existing inputs.** Rows use the product's own `.input`,
  `.select`, `.btn-trash`, and `.field-label` typography — so it inherits the
  Inter type scale and tokens automatically. No new colors/fonts were introduced.
- **% promoted into its own aligned OWNED column** (14px, 600, ink) instead of a
  faint trailing label — it's the computed payoff of the tool.
- **Long names truncate with ellipsis**; the name column is the flexible `1fr`
  track so it absorbs the leftover width while numeric columns stay fixed.
- **Delete reveals on hover** in a reserved column (quiet at rest).

We deliberately did NOT add the ownership bars (per-row micro-bars + footer
segmented strip) — they were prototyped then removed at the user's request.

---

## 4. Exact changes made to `safe-updated-ui/index.html`

All edits preserve the existing data model and JS wiring (see §5).

### CSS
- **`#shareholders-body`**: changed from 2-col `grid` → `display: flex;
  flex-direction: column;` (a list).
- **Removed** `.shareholder-card-wrapper` + odd-last-span rules.
- **Added** a shared grid for header + rows:
  ```css
  .sh-col-head, .sh-row {
    display: grid;
    grid-template-columns: 1fr 150px 130px 84px 32px;  /* name cat shares owned trash */
    gap: 12px;
    align-items: center;
  }
  ```
- **`.sh-col-head`**: column header styled to match the app's `.field-label`
  (11px, 600, uppercase, letter-spacing 1.4px, color `var(--ink-3)`). Helper
  classes `.l` (padding-left:8px) and `.r` (text-align right + padding-right:8px)
  align labels to the input text inset.
- **`.sh-row`**: `padding: 6px 8px; border-bottom: 1px solid var(--border);
  border-radius: 8px;` + `:hover { background: var(--surface); }`.
- **`.sh-row .row-name`**: `text-overflow: ellipsis; font-weight:500;` (truncation).
- **`.sh-row .row-pct`**: right-aligned, 14px/600, `var(--ink)`, tabular-nums.

### Hover-reveal delete — applied to ALL THREE steps
Three different parent containers hold the same `.row-trash-btn`:
- Cap table → `.sh-row`
- SAFE terms → `.input-row-card`
- Priced round → `.series-investor-row`

Rule: at rest `opacity:0` + `transition: opacity .15s`; on parent `:hover`
`opacity:1`. Mobile media query forces `opacity:1` (no hover on touch).

### Hover background — applied to ALL THREE steps
`.sh-row:hover`, `.input-row-card:hover`, `.series-investor-row:hover` →
`background: var(--surface)` (#f6f5fe) with `transition: background .15s`.
`.series-investor-row` got `padding: 6px 0; border-radius: 8px;` so the band has
breathing room. **Do NOT** reintroduce negative horizontal margins here — an
earlier attempt (`margin: 0 -8px`) caused an 8px horizontal overflow; fixed by
using vertical-only padding.

### SAFE trash spacing fix
`.safe-card-header` got `gap: 12px;` so the trash icon no longer butts against
the name field edge.

### HTML template
`#shareholder-card-template` rewritten from the card markup to a single row:
```html
<div class="sh-row">
  <input class="input row-name" type="text" placeholder="Shareholder name" />
  <select class="select row-category"></select>
  <input class="input row-input-right row-shares" type="text" />
  <div class="row-pct"></div>
  <button class="btn-trash row-trash-btn"></button>
</div>
```

### JS
`renderShareholders()` now prepends the column header into `#shareholders-body`:
```html
<div class="sh-col-head"><div class="l">Name</div><div class="l">Category</div>
<div class="r">Shares</div><div class="r">Owned</div><div></div></div>
```
Everything else in `renderShareholders` is unchanged.

### Mobile
`@media` block: old `.shareholder-card-wrapper` rules replaced with `.sh-row`
stacking (header hidden, row becomes a 2-col mini-card, trash absolute top-right).

---

## 5. Wiring that MUST stay intact (do not break)

The presentation reuses query-classes the JS depends on. Keep these class names
on the row elements or the calculator breaks:
- `.row-name`, `.row-category`, `.row-shares`, `.row-pct`, `.row-trash-btn`

Untouched logic (relied on by SAFE conversion math):
- `state.rowData` (rows filtered by `CapTableRowType.Common`)
- `updateRow(id, field, value)`, `deleteRow(id)`, `addRow('common')`
- formatters: `formatNumberWithCommas`, `formatInputLive`, `safeFormatPercent`
- `#cap-table-footer` total (Total fully diluted shares)

---

## 6. How to preview

Static file server (configs live in
`/Users/farheenshaikh/design_update_safe/.claude/launch.json`):
- `safe-updated-ui` → serves the real product on **:8092** (`index.html`)
- `option-a` → serves the standalone mockup on **:8091**

Use the Claude Preview tool with the server name, or:
`python3 -m http.server 8092 --directory /Users/farheenshaikh/safe-updated-ui`

Note: the app appears to persist some state to localStorage, so default numbers
may differ between loads. Use **Reset** for a clean state.

---

## 7. Verified in-browser

- Cap table: add (3→4), edit name/category/shares with live % recalc, long-name
  truncation ("Sequoia Capital O…"), delete (4→3, total restored). 0 console errors.
- Columns align with headers; OWNED % and total correct.
- Hover background + trash reveal confirmed on cap table, SAFE 1, and investor row.
- SAFE trash now has 12px gap from name.

---

## 8. Open items / possible follow-ups (not done)

1. **Total footer placement** — still sits BELOW the Back/Next nav row (original
   position). Mockup put it directly under the list. One DOM move if desired.
2. **Optional "shape of ownership" bar** — Option B idea: a slim segmented 100%
   strip as a bridge to the dilution step. Explicitly removed for now; revisit
   only if a visual payoff is wanted.
3. **Mobile** — list collapses to mini-cards; given a stacked 2-col grid. Works
   but only lightly tested; worth a real device pass.
4. **Delete UX** — instant delete (no confirm/undo). Fine for a calculator; add
   an undo toast before shipping if desired.

---

## 9. Quick-resume prompt (paste into a new chat)

> Continue work on the SAFE calculator cap-table redesign. The real product is
> `/Users/farheenshaikh/safe-updated-ui/index.html` (plain HTML, ~5000 lines).
> Read `HANDOFF-cap-table-redesign.md` in that folder for full context. We
> converted the cap-table step to Option A list rows reusing existing
> `.input`/`.select`/`.field-label` styles, added hover bg + hover-reveal delete
> across all three steps, and fixed SAFE trash spacing. Preview via the
> `safe-updated-ui` launch config on :8092.

---

# SESSION 2 — Manual Calculate gating + round-name labels (2026-06-18, later)

## 10. Manual "Calculate" gating (SHIPPED, committed)

**Goal:** stop the priced-round results from recalculating on every keystroke.
Results now only update when the user clicks **Calculate ▸**. Any input change
before clicking visually freezes the stale results behind a blur + "Outdated"
overlay.

### Behavior
- A **`Calculate ▸`** button sits in the priced-round card. Clicking it computes
  results. Button label is **always** "Calculate ▸" — never relabel to
  "Recalculate" (explicit user requirement).
- When any input changes before clicking, BOTH result regions freeze:
  - the results column (`.results-column-sticky`), and
  - the "Cap table after [round]" breakdown table (`#breakdown-section`).
- Each frozen region blurs (4px) and shows a `.stale-overlay` → `.stale-card`
  with an amber "Outdated" badge, "Showing previous results." and a
  **`Recalculate ▸`** button (the overlay button IS labeled Recalculate; only the
  in-card button stays "Calculate ▸"). We skipped the "N changes" counter shown
  in the reference screenshot.

### Implementation (all in `index.html`)
- **CSS** (added after `.results-column-sticky`, which was switched
  `position: static → relative`): `.is-stale` blur rules that use
  `:not(.stale-overlay)` so the overlay itself isn't blurred; `.stale-overlay`,
  `.stale-card` (rectangle, horizontal layout, `background:#FEF9EC`,
  `border:1px solid #f0cf72`, `max-width:420px`), `.stale-badge`,
  `.stale-recalc-btn` (`#b45309`), `.priced-calc-btn` (`#5f46ff`, full-width 48px).
- **HTML**: `.priced-calc-row` button in the priced-round card; two `.stale-overlay`
  blocks — one as first child of `.results-column-sticky`, one inside
  `#breakdown-section`.
- **JS state + gating** (after `let state`):
  ```js
  let resultsStale = false;
  const setResultsStale = (stale) => { resultsStale = stale;
    document.querySelector(".results-column-sticky")?.classList.toggle("is-stale", stale);
    document.getElementById("breakdown-section")?.classList.toggle("is-stale", stale); };
  window.calculateResults = () => updateUI({ compute: true });
  ```
- **`updateUI()` was split** into:
  - `renderInputs()` — always runs (validation, input formatting, renderSAFEs,
    renderSeriesInvestors, renderShareholders).
  - `computeResults()` — gated (fitConversion, results-card values, breakdown
    table, pie chart, AI advisor). **NOTE:** the `.display-round-name` span
    population (see §11) lives HERE, so round-name labels only refresh on compute.
  - thin orchestrator:
    ```js
    const updateUI = (opts = {}) => {
      const compute = !(opts && opts.compute === false);
      try { renderInputs();
        if (compute) { computeResults(); setResultsStale(false); }
        else { setResultsStale(true); }
      } catch (error) { console.error("Error updating UI:", error); }
    };
    ```
- **6 mutators** call `updateUI({ compute: false })` (mark stale, don't compute):
  `updateRow`, `addRow`, `deleteRow`, `togglePricedRound`, `updateGlobal`,
  `calculateSafeDiscount_UI` error path. **Left as compute=true (default):**
  `resetCalculator`, `initSAFEApp`, `DOMContentLoaded` init — so first paint and
  reset show computed (unblurred) results.

Verified in browser: initial load computed/unblurred → edit freezes both regions
+ shows overlay → Calculate/Recalculate recomputes + clears blur.

## 11. Dynamic round-name labels (SHIPPED this session)

**Request:** the "Before round" / "After round" labels in the **Founder
Ownership** comparison card should show the round name dynamically, exactly like
"Impact of [Series A] on founder(s)".

**Change:** the two hardcoded labels became `.display-round-name` spans:
```html
<label class="sc-comparison-label">Before <span class="display-round-name"></span></label>
<label class="sc-comparison-label">After <span class="display-round-name"></span></label>
```
All `.display-round-name` spans (now 8: impact heading, ownership/dilution
sublabels, the note, breakdown H2, + these 2) are populated by one loop inside
`computeResults()` (~line 3900) from `state.roundName` (the Round name field in
the priced-round card). So they refresh on Calculate — identical behavior to the
impact heading. Verified: setting round name to "Seed Extension" → all labels read
"Before/After/Impact of/Cap table after Seed Extension" after Calculate.

## 12. Git state (IMPORTANT — read before any PR work)

Repo `/Users/farheenshaikh/safe-updated-ui` has **3 commits**, current branch
`feature/manual-calculate-results`:
- `958c21e` Create index.html (initial)
- `dc2f0ee` "Update index.html" — **Vignesh Shankaran**. Made the round-name
  labels dynamic (`.display-round-name`), fixed the MFN type label, and added PDF
  report fields (`valuation`, `raised`, option-pool display).
- `946fc27` "Gate priced-round results behind a manual Calculate button" — mine
  (authored `Farheen Shaikh <farheenshaikh@Farheens-MacBook-Air.local>`, auto-
  detected identity). **+341 / −120 in `index.html`.**

⚠️ **`946fc27` bundles TWO unrelated bodies of work** because the cap-table
redesign (§1–9) was still uncommitted in the working tree when the commit was
made: (a) the manual-Calculate gating (§10), and (b) the Option-A shareholder
list redesign (`.sh-row`/`.sh-col-head`, hover/trash reveals, mobile stacking).
They can't be cleanly split after the fact since the redesign was never separately
snapshotted. This very file (`HANDOFF-cap-table-redesign.md`) is the only local
file NOT committed — it's still untracked (`??`).

> ⚠️ **SUPERSEDED — see §14 (Session 3).** As of 2026-06-19 the branch has a 4th
> commit, repo-local git identity was set, and the branch was pushed to `origin`.
> The "no PR / don't push" instruction below is no longer current.

**(Historical, as of Session 2)** No PR has been created. `gh` is not installed
(no CLI). User explicitly said to wait — do NOT push or open a PR without a fresh
go-ahead.

## 13. Investigations this session (read-only — no code changed)

- **PDF generation recently changed?** No. PDF is server-side (frontend decodes
  `result.pdfBase64` via atob → Uint8Array → Blob `application/pdf`, downloads
  `SAFE_Calculator_Report_<date>.pdf`, ~lines 4986 & 5119). Unchanged since the
  initial commit; only the report *data fields* were touched in `dc2f0ee`.
- **Are current-cap-table names tied to the "after Series A" breakdown?** Yes, by
  design — no separate fix was made. Names flow live from `state.rowData`: both
  `buildEstimatedPreRoundCapTable` (~3464) and `buildPricedRoundCapTable` (~3589)
  spread the source row (`...c`/`...s`/`...se`), preserving `name`.
  `renderBreakdownTable` (~4358) matches pre/post rows by **`id`** (not name) and
  displays `post.name || pre.name || "—"` (line ~4391). Editing a name flows
  through to the breakdown on recompute because the row keeps the same `id`.
- **Where is dilution calculated?** Only **founder dilution**, no separate "round
  dilution". At ~line 3986: `dilution = totalFounderPctPre - totalFounderPctPost`
  (percentage points). `totalFounderPctPre` is **post-SAFE, pre-priced-round**
  founder ownership (incl. option-pool top-up); `totalFounderPctPost` is after the
  round. So it captures dilution from the round + pool top-up only — NOT the SAFE
  conversion itself (the baseline is already post-SAFE). Shown in
  `founder-dilution-val`, the `dilution-summary-note`, and the PDF `dilution` field.

---

# SESSION 3 — Color / accent refinements + git push (2026-06-19)

## 14. Goal of the session

Reduce the "everything is purple" feeling of the results UI so the purple-tinted
background reads as a deliberate brand wash, then commit/push the work. The page
background itself was **not** changed (user constraint: can't change it). Instead
we pulled purple back to a few intentional accent roles and neutralized the rest.

**Purple is now reserved for: the hero stat (`56.04%`), primary buttons
(Calculate / Next), the active wizard tab, the founder ownership bars, the
"SAFE" wordmark, and the "Cap table after [round]" section (dots + heading).**
Everything else that was purple was muted.

## 15. Exact changes made to `safe-updated-ui/index.html`

All edits are CSS/inline-style/JS-palette only — no markup/logic changes.

### Accent pull-back (CSS)
- **`.step-item`** inactive label color `#ccc6ff` → `#9ca3af` (grey).
- **`.step-number`** default `background:#eae8f5; color:#ccc6ff` →
  `background:#eceef2; color:#9ca3af`. Active step number kept purple.
- **`.step-item.completed`** label `#5f46ff` → `#444266`; its `.step-number`
  `background:#eae7ff; color:#5f46ff` → `background:#eceef2; color:#444266`.
- **`.sc-anchor-btn`** ("View full results"): dashed border `#ccc6ff` → `#cbd5e1`,
  `background:#f0eefd` → `#f4f5f7`, text `#5f46ff` → `#444266`; hover border
  `#5f46ff` → `#9ca3af`.
- **`.additional-shares-note`** ("+N shares will be added") `#5f46ff` → `#444266`.

### Founder ownership bars — two-tone (CSS)
- **`.sc-bar-fill`** base stays brand purple `#5f46ff` (this is the "After" bar).
- **Added `#founder-before-bar { background: #c4bfff; }`** so the "Before" bar is
  a muted lavender. Rationale: encode the dilution visually (Before 93% muted →
  After 56% solid) instead of two identical purple bars. (Intermediate attempts
  used slate `#64748b`; reverted to purple per user, then two-toned.)

### Conversion note callout (inline)
- The `<p>` note ("All before/after figures assume all SAFEs have converted…")
  got `background:#FEF9EC; padding:10px 12px; border-radius:8px;` added to its
  inline style. Text color left `#9ca3af` (slightly low contrast on cream — open
  item if a darker warm text like `#8a6d3b` is wanted).

### Impact-card accent bar (CSS)
- **`.sc-accent-bar`** `background:#5f46ff` → **`#F3F0FF`** (very light lavender,
  intentionally low-attention). History: tried `#201547` (deep indigo) and
  `#F7F5F3` (too light/invisible) before settling on `#F3F0FF`.

### "Cap table after [round]" — left/restored to PURPLE (do not mute)
- **`getRowColor()`** (~line 4329, drives the breakdown-table `.name-dot`):
  Founders `#5f46ff`, SAFEs `#7464ff`, default `#5f46ff` — these were briefly
  changed to slate/blue then **reverted to purple** at user request. This section
  is the bottom full-width "Cap table after Series A" table.
- The "Cap table after [round]" H2 round-name accent span inline color is
  `#5f46ff` (briefly set to `#0d0a40`, reverted).

### Pie-chart legend palette — left as slate (NOT reverted)
- **`categoryPalettes.Founder`** (~line 4447) is `["#475569","#64748b","#94a3b8"]`
  (slate). This feeds the **pie chart** (`pieChartInstance`), a *different*
  component from the breakdown table. Intentionally left muted. ⚠️ Don't confuse
  this with `getRowColor` above — the table dots and the pie legend have separate
  color sources.

## 16. Git state (current — supersedes §12)

Branch `feature/manual-calculate-results`, now **4 commits**, pushed to `origin`
(`https://github.com/Vigneshshankaran/safe-updated-ui.git`):
- `958c21e` Create index.html
- `dc2f0ee` Update index.html (Vignesh Shankaran)
- `946fc27` Gate priced-round results behind a manual Calculate button
- `3f6f700` **Reduce purple accent load and refine results-panel styling** — this
  session. Includes the §15 styling work, the previously-untracked
  `HANDOFF-cap-table-redesign.md`, and the §11 dynamic-label edits that were still
  uncommitted in the working tree.

**Git identity:** set **repo-local only** (not global) to
`farheen-cyber <farheen@equitylist.co>`; commit `3f6f700` was amended with
`--reset-author` to use it (the original was the auto-detected
`Farheen Shaikh <…@Farheens-MacBook-Air.local>`).

**PR:** `gh` still not installed and no API token in the environment, so the PR
was **not** created programmatically. A pre-filled GitHub compare URL was handed
to the user to click "Create pull request":
`https://github.com/Vigneshshankaran/safe-updated-ui/compare/main...feature/manual-calculate-results?expand=1` (+ title/body params). PR base is `main`, so it
includes both `946fc27` and `3f6f700`.

## 17. Open items / follow-ups

1. **Note callout text contrast** — `#9ca3af` on `#FEF9EC` is low-contrast;
   consider darkening to a warm `#8a6d3b` (accessibility).
2. **Step indicator "active" pill** still uses `background:#eae7ff` (lavender) —
   intentionally kept as the active highlight; revisit only if more de-purpling
   is wanted.
3. **PR not opened** — only the compare URL was generated; confirm whether the
   user opened it.
4. **SAFE badge chips** (`MFN SAFE`, `PRE-MONEY SAFE`, `POST-MONEY SAFE`, the
   `.sc-tag` class) were left purple text on light-purple — a candidate for
   muting if a future pass wants even less purple.

## 18. Verified in-browser (Claude Preview, :8092)

Each change was confirmed via screenshot + `getComputedStyle` reads:
- Step indicators: only the active tab/number is purple; completed/inactive grey.
- Founder bars: Before `rgb(196,191,255)` (#c4bfff), After `rgb(95,70,255)`
  (#5f46ff).
- Breakdown dots after revert: Founders `rgb(95,70,255)`, SAFEs `rgb(116,100,255)`.
- Accent bar: `rgb(243,240,255)` (#F3F0FF).
- "Cap table after" H2 accent: `rgb(95,70,255)`.
