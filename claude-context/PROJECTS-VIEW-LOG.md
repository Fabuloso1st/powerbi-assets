# Projects View modernization — running log

Status key: **[Confirmed]** stated by the dashboard owner. **[Decided]** agreed in session, not yet verified in the live report. **[Open]** unresolved.

Read `CLAUDE.md` in this folder first for the report's overall purpose, data model, and counting rules. This file tracks the Projects-view redesign specifically: decisions made, current measure state, and what's still open — so a new session can pick up mid-stream without re-litigating earlier calls.

---

## 1. Where this started

The original Projects view (left rail of slicers, map, projects-by-country table, country×theme heatmap, projects table, donor bars, agency bars, start-year chart) was reviewed and rated 6/10: the data handling was sound (counting rules, "planned ≠ pipeline" caveat, coverage caveat) but the page had ~10 visual zones repeating the same facts (country totals shown 3 times, status colour used 6 places, agencies/donors each appearing as both a slicer and a chart).

## 2. Layout decisions **[Decided]**

- Trimmed to six zones: KPI tiles, a country×theme coverage grid (the main visual), agency bars, donor bars, project list, a slimmer rail.
- **Default filter: Active + Planned**, with Ended as an opt-in toggle (not shown by default) — avoids overstating current coverage from historical projects (e.g. Côte d'Ivoire's high total is mostly ended work).
- Removed from the original: the map (redundant with the landing page's footprint map), the standalone projects-by-country table (folded into the coverage grid as a Total column), the start-year chart (doesn't serve the page's four core questions), most "i" info icons (trimmed to ~3), the Organizations slicer (redundant with the agency chart), the header flag strip (decorative).
- The four questions this view answers: (1) where is work concentrated / where are the gaps, by country and theme; (2) who's already working there (agencies/donors), for coordination; (3) what's ongoing vs. coming (active vs planned); (4) which specific projects.

## 3. "Who works where" — the coverage grid

This went through several rounds; **current direction is locked in as a pill-badge grid** (see §4). Earlier explorations, kept here for reference / in case of reversal:

- **Heatmap** (country × theme matrix, gold-shaded cells with count + top-2 agencies): first working version, DAX built, worked but felt visually heavy/cluttered per owner feedback ("too much like a spreadsheet").
- **4 structural alternatives** explored (dot matrix, thematic-mix stacked bars, theme-first columns, country-tile profiles) — not picked.
- **4 more alternatives, explicitly not a map or a Sankey** (seats/badges, unit-square chart, theme "flowers") — picked **"seats at the table"**: agencies shown as badges sized by project count, empty dashed seat = gap.
- Decision: **no agency logos** — organisation logos aren't reproduced; agency names/acronyms are used as text instead.
- **4 pill styles** tested (solid, outline+count-chip, tinted-with-lead-solid, minimal-tag) — owner picked a direction, then asked for a re-do once About Page system context (§5) was shared.
- **Final 4, built against the About Page design system** (§5) — owner picked **"Gold-hover pill"**: outline pill, neutral by default, solid gold fill + dark text on `:hover` (same interaction language as the KPI cards / CTA button elsewhere in the report).

## 4. Current measure: `Who Works Where Pills HTML`

Status: **[Decided]**, iterating on polish. Last known-good full text of the measure is in the session that produced it — **not yet copied into this repo as a `.dax` file**; see Open Items.

Behaviour:
- Rows = `BridgeProjectCountry[Country]`, sorted by total project count descending.
- Columns = `DimProject[Macro Thematic Focus]`, sorted by total project count descending, fixed to **285px each** (owner request).
- Cell content = top 3 agencies by project count (`DimProject[Agency]`) as pills, with a "+N" overflow chip if more than 3. Empty cell (0 projects) shows a dashed "no project" pill. Cell with projects but no agency recorded shows "N · agency not recorded" as plain text.
- Country flags are **inline SVG, hardcoded for the 9 beneficiary countries** directly in the DAX (`SWITCH(TRUE(), Country = "Benin", "<svg>...", ...)`) — deliberately **not** sourced from `BridgeProjectCountry[CountryImageURL]`, to avoid network calls / broken-image risk inside the HTML Content visual on every filter-driven redraw. Trade-off: if a 10th beneficiary country is ever added, its flag SVG must be added to the `SWITCH` by hand.
- `Total` column kept on the right (differs from the earlier heatmap version, where it was removed as redundant — it's not redundant in this layout).
- Styling pulled from the About Page's own measures (`KPI Strip HTML V11`, `Lead Insight Flip Card HTML (SelfMod)`, `Donut Chart HTML (Rev 9) - Legend Only`, `Page 1 - Tutorial Guide V3`, `_Button_ExploreDashboard V2`):
  - Fonts: **Inter** (UI/numbers) + **Newsreader** (serif, used for the Total column's big number), loaded via Google Fonts `@import` inside the measure's own `<style>` block — confirmed this pattern already works in this report (Inter is not silently falling back).
  - Macro theme colours, pulled from the donut legend measure, used **only on each column's top bar**, not on the pills: Border Governance `#4A3810`, Law Enforcement `#8C6D1E`, Community Engagement `#C9A227`, Cross-Border `#F5D061`.
  - Card chrome matches the KPI strip exactly: white background, `#E4E2DB` 1px border, 14px radius, soft shadow, small gold dot + uppercase tracked header label.
  - Pills: **neutral light grey border (`#D9D2BE`) by default for every theme** (owner explicitly reversed an earlier "pill border = theme colour" decision), filled solid gold (`#fede59`) with dark text (`#2a1c08`) on hover — pure CSS `:hover`, no JS.
- Known bug already fixed once: flags rendered oversized because `<span>` is inline by default and ignores explicit `width`/`height` — fixed with `display:inline-block` + `overflow:hidden` on the flag wrapper and `width:100%;height:100%` on the nested `<svg>`.

## 5. About Page design system (extracted from the owner's own measures, for reuse)

- **Fonts:** Inter (UI, numbers, tabular-nums everywhere), Newsreader (serif, big display headings/numbers), occasionally Public Sans — all loaded via `@import url('https://fonts.googleapis.com/css2?family=...')` inside each measure's own `<style>` block.
- **Primary accent:** `#fede59` (gold) — CTA button, "Key Insight" badge, hover states, pulse-glow animation on the Explore Dashboard button.
- **Secondary accent:** teal/green (`#3E7E72`, `#2F6459`, `#7FD3C0`) — reserved for "active" status (pulsing dot in the KPI strip), not used elsewhere.
- **Macro theme colours** (from the donut legend): Border Governance `#4A3810`, Law Enforcement `#8C6D1E`, Community Engagement `#C9A227`, Cross-Border `#F5D061`. Note: the donut's short label is **"Community Engagement"**, not "Community Security" — match this exactly when pulling `DimProject[Macro Thematic Focus - Short]`.
- **Card chrome:** white, `#E4E2DB` border, 14–22px radius, soft shadow, hover state brightens border to `#E3C94F` with a lifted shadow (KPI cards) or flips a whole element solid gold (pills, CTA button).
- **Type details:** uppercase labels with `.07–.14em` letter-spacing, weight 600–700, `font-variant-numeric: tabular-nums` on every numeric display.
- **Icons:** small circular/rounded-square badges (~24–28px), pale yellow at rest, solid gold on hover.

## 6. Data caveats carried through every version

- Only **7 of 43** IP-SCG implementing partners have shared data — an empty cell may be unreported work, not a confirmed gap. This note is on the coverage grid card.
- Planned ≠ pipeline: the database's "Planned" count (6 at last snapshot) is not the same as the 8-project IBSM pipeline (3 second-call + 5 matching-funds) described in the coach material.
- A project can have multiple agencies and multiple donors — counts in the pill grid, agency bars, and donor bars can each legitimately exceed the project total.

## 7. Separate bug flagged (not part of this redesign)

In the owner's live report, the **Funding Entity slicer appears to be bound to `Macro Thematic Focus` instead of the funding-entity column** — it lists the four macro themes instead of funders. Needs a fix on the field well, unrelated to the Projects-view rebuild.

## 8. Open items

- [ ] **This measure has not been committed to this repo as a `.dax` file yet.** This session could clone `powerbi-assets` but could not push — Claude has no GitHub write access configured for this org/repo (no GitHub App install, or it's not linked to this org). An org owner needs to install the Claude GitHub App (https://github.com/apps/claude/installations/select_target) or reconnect GitHub under claude.ai → Settings → Connectors, after which a future session can commit the actual `.dax` measure files here instead of only this log.
- [ ] Confirm the owner's latest screenshot after the two most recent fixes (flag sizing, 285px equal columns + grey pills) looks right.
- [ ] Column widths (285px × 4) were sized for the current agency-count distribution. If Cross-Border (currently only 2 projects total) grows, or any cell needs more than 3 agency pills + overflow, re-check for wrapping/cramping.
- [ ] Not yet decided: does this pill grid replace the separate "Projects Data" table, or sit alongside it as a second visual?
- [ ] Not yet decided: should the Ended-projects toggle state (on/off) persist a user's choice, or always reset to Active+Planned on page load?
- [ ] The rank-based "which theme is #1/#2/#3/#4" logic in the donut legend measure was *not* reused in the pill grid — the pill grid instead sorts columns by live project count under the current filter context each time. Worth deciding if these two should use the same ranking approach for consistency.
