# Projects View modernization — running log

Status key: **[Confirmed]** stated by the dashboard owner. **[Decided]** agreed in session, not yet verified in the live report. **[Open]** unresolved.

Last updated: 2026-11-04. **Current state in one line: the "Who works where" grid has gone through many rounds and the owner is still not satisfied with it. [Confirmed]** The reasons have not been spelled out yet — see §9 before changing anything else.

Read `CLAUDE.md` in this folder first for the report's overall purpose, data model, and counting rules. This file tracks the Projects-view redesign specifically: decisions made, current measure state, and what's still open — so a new session can pick up mid-stream without re-litigating earlier calls.

---

## 1. Where this started

The original Projects view (left rail of slicers, map, projects-by-country table, country×theme heatmap, projects table, donor bars, agency bars, start-year chart) was reviewed and rated 6/10: the data handling was sound (counting rules, "planned ≠ pipeline" caveat, coverage caveat) but the page had ~10 visual zones repeating the same facts (country totals shown 3 times, status colour used 6 places, agencies/donors each appearing as both a slicer and a chart).

## 2. Layout decisions **[Decided]**

- Trimmed to six zones: KPI tiles, a country×theme coverage grid (the main visual), agency bars, donor bars, project list, a slimmer rail.
- **Default filter: Active + Planned**, with Ended as an opt-in toggle (not shown by default) — avoids overstating current coverage from historical projects (e.g. Côte d'Ivoire's high total is mostly ended work).
- Removed from the original: the map (redundant with the landing page's footprint map), the standalone projects-by-country table (folded into the coverage grid), the start-year chart (doesn't serve the page's four core questions), most "i" info icons (trimmed to ~3), the Organizations slicer (redundant with the agency chart), the header flag strip (decorative).
- The four questions this view answers: (1) where is work concentrated / where are the gaps, by country and theme; (2) who's already working there (agencies/donors), for coordination; (3) what's ongoing vs. coming (active vs planned); (4) which specific projects.
- **Scope for now [Confirmed]:** the owner is focusing on the grid only. The rest of the page (KPI tiles, donors chart, donut, Projects Data table, left rail, layout balance) is deferred to a later stage. A 2026-11-03 review rated the whole page about 7/10, with the grid itself the strongest part and the surrounding visuals not yet matching its style.

## 3. "Who works where" — how the grid evolved

**Early rounds (before 2026-11-03):**
- **Heatmap** (country × theme matrix, gold-shaded cells with count + top-2 agencies): worked but felt "too much like a spreadsheet".
- 4 structural alternatives (dot matrix, stacked bars, theme-first columns, country tiles) and 4 more (seats/badges, unit-square chart, theme "flowers") — picked **"seats at the table"**: agencies as pills, empty dashed seat = gap.
- 4 pill styles tested; owner picked the **"Gold-hover pill"**: outline pill, neutral by default, solid gold on `:hover`. No agency logos — agency names/acronyms as text.

**Session of 2026-11-03 (all [Decided] unless noted):**
1. **Responds to the visual's height.** The card height is `calc(100vh - 8px)` so it follows the visual in Power BI. `height:100%` did not work: the HTML visual runs in its own frame whose wrappers have no height. If a gap or clipping shows at the bottom, adjust the `8px`.
2. **No horizontal scrolling.** Only vertical overflow scrolls; the sticky header stays pinned.
3. **Bigger type.** Pills 14.5px, country names 17.5px, theme names ~15.5px, flags 33×22. Small grey text darkened to `#6F6B63` for contrast.
4. **Theme icons.** Inline SVG (Tabler outline set, MIT), chosen in a gold rounded-square badge that turns solid gold on hover: **fingerprint** = Law Enforcement (chosen over a shield), **arch building** = Border Governance, **group of people** = Community Engagement, **opposing arrows** = Cross-Border. Icons were enlarged after feedback (now 44px badge, 27px glyph).
5. **"No project" pill** is light red with red text and a dashed border, so a gap is clearly visible as a gap.
6. **Whole-row hover**: hovering anywhere on a country row highlights the row (pale gold, gold edge strip on the country cell).
7. **Selection-aware behaviour.** Countries and themes chosen in the native slicers are pinned first; the others stay visible but greyed (muted name, grey flag, dashes in cells). Built with `ALL()` for the full list and `VALUES()` + `CONTAINSROW` for the selection. A country or theme also greys when the other slicers leave it with no projects. **Caveat:** `ALL()` also removes report/page-level filters on the Country and Theme columns, so anything hidden by such a filter would now show greyed.
8. **Count bubble next to each country name**; the **Total column on the right was removed** (owner request). The bubble shows that country's project count for the selected themes.
9. **Theme panels (current look).** Each thematic area is a rounded panel with a faint shadow on a white canvas, echoing the About page. Header copies the KPI tiles: icon + uppercase label, large project count, divider, caption "NN% of projects", theme colour as a bar on top. Chosen as option **B** of four (A chart-card header, B KPI-tile header, C dark Key Insight header, D theme-coloured panels).
10. **Columns are proportional** (17% country + 4 × 20.75%) rather than fixed pixels.

**Tried and dropped:**
- Per-cell project counts as a lead number (A), heat tint (B), and a density bar with a "1 agency" label (C): the owner did not like any of those three takes. A thin density bar without the label was then tried, first gold then in theme colours, and finally removed in favour of the country count bubble.
- Greying with `FILTERS()`/removal of rows: replaced by the pinned-and-greyed behaviour above.

## 4. Current measure: `Who Works Where Pills HTML v.5`

Status: **[Decided]**, built and iterated by the owner in Power BI; the owner has not signed it off. The full text is committed alongside this log as `who-works-where-pills-html-v5.dax` (see §8 for how earlier versions are kept).

Behaviour:
- Rows = every country in `BridgeProjectCountry[Country]`; selected ones first, sorted by project count, then greyed ones.
- Columns = every `DimProject[Macro Thematic Focus]`; selected first, sorted by count, then greyed ones.
- Cell = top 3 agencies (`DimProject[Agency]`) by project count as pills with a "+N" chip. Empty cell on an active row and column = red "no project" pill. Cell with projects but no agency recorded = muted "agency not recorded".
- Country flags are inline SVG, hardcoded for the 9 beneficiary countries (not `CountryImageURL`): a 10th country needs its flag added to the `SWITCH` by hand. A grey placeholder shows for unknown countries.
- Theme icons and colours are matched by `CONTAINSSTRING` on the theme name (Border Governance, Law Enforcement, Community, Cross-Border). A renamed or new theme gets no icon and the fallback colour `#8C6D1E`.
- Fonts: Inter (UI) and Newsreader (serif) via a Google Fonts `@import` inside the measure.
- Styling reuses the About page design system (§5); pill borders are neutral grey with solid gold on hover.

Version history (names are only labels; the measure is renamed as it changes):
| Version | What changed |
|---|---|
| v1 | Pill grid in a card, fixed 285px columns, Total column, theme colour bar |
| v2 | Responsive to visual height, no horizontal scroll, bigger pills, red "no project" pill |
| v3 | Theme icons (fingerprint for Law Enforcement), row hover, larger flags and fonts |
| v4 | Selected countries pinned first, others greyed |
| v5 (`v.5`) | Themes greyed the same way, taller header with larger icons, country count bubble, Total column removed, KPI-style theme panels, white canvas and faint shadows |

Known bug already fixed once: flags rendered oversized because `<span>` is inline and ignores width/height — fixed with `display:inline-block;overflow:hidden` on the wrapper and `width:100%;height:100%` on the SVG.

## 5. About Page design system (extracted from the owner's own measures, for reuse)

- **Fonts:** Inter (UI, numbers, tabular-nums), Newsreader (serif, big display headings/numbers), occasionally Public Sans — loaded via `@import url('https://fonts.googleapis.com/css2?family=...')` inside each measure's `<style>` block.
- **Primary accent:** `#fede59` (gold) — CTA button, "Key Insight" badge, hover states, pulse-glow on the Explore Dashboard button.
- **Secondary accent:** teal/green (`#3E7E72`, `#2F6459`, `#7FD3C0`) — reserved for "active" status; not used elsewhere.
- **Macro theme colours** (from the donut legend): Border Governance `#4A3810`, Law Enforcement `#8C6D1E`, Community Engagement `#C9A227`, Cross-Border `#F5D061`. The donut's short label is **"Community Engagement"**, not "Community Security" — match this when pulling `DimProject[Macro Thematic Focus - Short]`.
- **Card chrome:** white, `#E4E2DB` border, 14–22px radius, soft shadow, hover brightens border to `#E3C94F` (KPI cards) or flips the element solid gold (pills, CTA).
- **Type details:** uppercase labels with `.07–.14em` letter-spacing, weight 600–700, `font-variant-numeric: tabular-nums` on numbers.
- **Icons:** small circular/rounded-square badges, pale yellow at rest, solid gold on hover.

## 6. Data caveats carried through every version

- Only **7 of 43** IP-SCG implementing partners have shared data — an empty cell may be unreported work, not a confirmed gap.
- **Decision [Confirmed] 2026-11-03:** the owner does **not** want the grid to depend on partner coverage data ("very ambitious to depend on data from all these other implementing partners"). So no live "x of 43 reporting" figure or computed completeness indicator is to be added; we stay with what the data holds.
  - **Consequence to be aware of:** the current footer no longer carries the older "7 of 43" note, and the red "no project" pill can be read as a confirmed gap. This is an accepted risk for now; if it matters later, a static one-line footnote is the cheapest option.
- Planned ≠ pipeline: the database's "Planned" count (6 at last snapshot) is not the same as the 8-project IBSM pipeline (3 second-call + 5 matching-funds).
- A project can have multiple agencies and multiple donors — pill counts can legitimately exceed the project total. Each macro theme maps to **one** category per project, so theme counts add up to the project total.
- **[Snapshot] 2026-11-03, Active + Planned, all countries:** 55 projects; Law Enforcement 23, Border Governance 18, Community Engagement 12, Cross-Border 2. Nigeria has few projects and no activities (joined recently).

## 7. Separate bug flagged (not part of this redesign)

In the owner's live report, the **Funding Entity slicer appears to be bound to `Macro Thematic Focus` instead of the funding-entity column** — it lists the four macro themes instead of funders. Needs a fix on the field well, unrelated to the Projects-view rebuild. **[Open]** (still unchecked at 2026-11-04).

## 8. Open items

- [ ] **Owner is not satisfied with the grid [Confirmed].** What is missing has not been stated. First step next session: ask what feels wrong (see §9 for hypotheses) rather than iterating on style again.
- [ ] **GitHub write access.** Claude has no GitHub write access to this repo (no GitHub App install / not linked to this org), so the measure and this log are handed over as files and committed by the owner. The v.5 measure is supplied as `who-works-where-pills-html-v5.dax`. Earlier versions exist only in past sessions.
- [ ] Confirm v.5 in the live report after the last change (white canvas and faint panel shadows) renders as intended, including the shadow joining cleanly down each panel, and that the greyed panels look right.
- [ ] Tall header: the panel headers are ~150px. Check the grid still fits the visual's height with all 9 countries without scrolling.
- [ ] Rows share one height across all four panels, so a wrapped pill in one panel makes that row taller in every panel.
- [ ] Not yet decided: does the pill grid replace the separate "Projects Data" table, or sit alongside it?
- [ ] Not yet decided: should the Ended-projects toggle persist the user's choice, or always reset to Active+Planned on load?
- [ ] The rank-based "which theme is #1/#2/#3/#4" logic in the donut legend was not reused in the grid (the grid sorts by live count). Decide whether the two should share one ranking.
- [ ] Report/page-level filters on Country or Theme are overridden by the greying logic (`ALL()`); confirm none are used, or scope the measure differently.
- [ ] Wording: title says "for West Africa" while the description says West and Central Africa; the Navigation Guide still says "8 focus countries" vs 9. (Page-level, deferred.)

## 9. Why the owner may still be unsatisfied — hypotheses to test, not decisions

None of these have been confirmed. Ask the owner which, if any, apply.
- **Density and height:** the panel headers plus tall rows might leave too little room, or push the grid into scrolling on the real page.
- **Cell content:** a cell shows agencies but not how much work sits there (counts per cell were tried three ways and rejected as styles, not as an idea). The country count bubble is the only number left.
- **Concentration:** nothing in the grid shows where work is concentrated or thin besides the red gap pills; lone-agency cells (e.g. most of Cross-Border is IOM alone) are not marked after the "1 agency" label was rejected.
- **Visual weight:** panels with shadows may feel heavier than the rest of the page, which has not been restyled yet.
- **Gaps vs missing data:** red pills still look like confirmed gaps (see §6).
- **Reading flow:** whether the country-first layout is right, versus theme-first, given that funders compare themes across countries.
- **Prototypes** (private to the owner's claude.ai account; useful to reopen): count-lead / heat tint / density bar — https://claude.ai/artifact/WmaanK8LAsBMjxNbNRoUJs , https://claude.ai/artifact/BFuUexSrXVSKLLc2g6r7Hm , https://claude.ai/artifact/CEZaCpHcmW2hwZ57NwwSP7 ; theme panels A–D — https://claude.ai/artifact/M6ucPeqHvrYNuXs66a7zV4
