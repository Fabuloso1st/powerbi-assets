# Projects View modernization — running log

Status key: **[Confirmed]** stated by the dashboard owner. **[Decided]** agreed in session, not yet verified in the live report. **[Open]** unresolved.

Last updated: 2026-11-04. **Current state in one line: the grid was redesigned as v6 (themes as rows, countries as columns, "Paper" palette) with click interactions; the owner confirmed it renders ("Works!") and is refining details. Last two changes (+N expansion, greyed corner icons) are not yet confirmed in the live report.**

Read `CLAUDE.md` in this folder first for the report's overall purpose, data model, and counting rules. This file tracks the Projects-view redesign specifically: decisions made, current measure state, and what's still open — so a new session can pick up mid-stream without re-litigating earlier calls.

---

## 1. Where this started

The original Projects view (left rail of slicers, map, projects-by-country table, country×theme heatmap, projects table, donor bars, agency bars, start-year chart) was reviewed and rated 6/10: the data handling was sound (counting rules, "planned ≠ pipeline" caveat, coverage caveat) but the page had ~10 visual zones repeating the same facts (country totals shown 3 times, status colour used 6 places, agencies/donors each appearing as both a slicer and a chart).

## 2. Layout decisions **[Decided]**

- Trimmed to six zones: KPI tiles, a country×theme coverage grid (the main visual), agency bars, donor bars, project list, a slimmer rail.
- **Default filter: Active + Planned**, with Ended as an opt-in toggle — avoids overstating current coverage from historical projects (e.g. Côte d'Ivoire's high total is mostly ended work).
- Removed from the original: the map, the standalone projects-by-country table, the start-year chart, most "i" info icons, the Organizations slicer, the header flag strip.
- The four questions this view answers: (1) where is work concentrated / where are the gaps, by country and theme; (2) who's already working there, for coordination; (3) what's ongoing vs. coming; (4) which specific projects.
- **Purpose of the grid [Confirmed]:** a visibility/coordination tool for funders ("who works where"), not a results tracker.
- **Scope for now [Confirmed]:** the owner is focusing on the grid only. The rest of the page (KPI tiles, donors chart, donut, Projects Data table, left rail, layout balance) is deferred. A 2026-11-03 review rated the whole page about 7/10.

## 3. "Who works where" — how the grid evolved

**Early rounds (before 2026-11-03):** heatmap (too spreadsheet-like) -> "seats at the table" (agencies as pills, empty dashed seat = gap) -> "Gold-hover pill" style. Versions v1–v5 were country-first (countries as rows, themes as panels); see §4 history.

**Session of 2026-11-03 (v5 features that carried over):** height follows the visual (`calc(100vh - 8px)`), no horizontal scroll, theme icons (Tabler: fingerprint = Law Enforcement, arch = Border Governance, people = Community Engagement, arrows = Cross-Border), red "no project" pill, row hover, selection-aware greying (`ALL()` for the full list, `VALUES()` + `CONTAINSROW` for the selection), country count bubble, Total column removed.

**Session of 2026-11-04 (v6, all [Decided]; owner confirmed it renders):**
1. **Reviewed v5**, with feedback that hover did not reach the countries, the header area felt empty, theme names should be spelled out in full, and a light-grey separator was wanted. Three new directions were prototyped; owner picked **C (theme-first)**: themes as rows, countries as columns, theme names in full in a left column.
2. **Palette:** owner disliked the colours/gradients; chose the flat **"Paper" palette** (white/cream/greys, `#E4E2DB` borders) but **kept the original gold icon badges and gold hover**, and **kept the red "no project" pill**.
3. **Adaptive to the visual:** rem-based sizing with `html{font-size:clamp(11px,min(1.186vw,1.5625vh),26px)}`, tuned for the owner's visual size of **1772 x 1187**; rows are `minmax(7rem,1fr)` so the grid fills the height; no scrollbar at that size.
4. **Crosshair hover:** hovering a cell highlights its row and column, including the country header (via generated `:has()` rules).
5. **Header corner:** the old legend was replaced by "Macro Thematic Areas" plus the four theme icons. The theme names are no longer shown on hover (they are spelled out in the row headers).
6. **Click a corner icon** -> pop-up with that theme's definition (text hardcoded from `CLAUDE.md`).
7. **Click a pill** -> pop-up listing that agency's projects for that country and theme (Project ID, Title, Status, Dates), with the agency logo. **Click a project row** -> detail card (logo, ID, status, title, start/end dates, donor, benefiting countries) with a Back button. Replaces the old native table + tooltip. Esc, the x button, or clicking outside closes it.
8. **"+N" chip** expands the cell's remaining agency pills in place (becomes "- fewer" to collapse); it does **not** open a project list. Each revealed pill is clickable like the top three. Expanded cells grow taller and the whole row grows with them.
9. **Footer** is dynamic: "Top 3 agencies per cell; click +N to show the rest. " + "All project statuses" or "Project status: <selected>", plus a greyed-selection note when applicable. The old hardcoded "Active + Planned" text was removed.
10. **Greyed theme icons:** corner icons of themes outside the current slicer selection are greyed (`.kb.off`); they stay clickable so definitions still open. (Added at end of session, not yet confirmed.)

**Tried and dropped:** per-cell counts as a lead number, heat tint, density bar (all rejected as styles); `FILTERS()`-based row removal; hover tooltip with theme names.

## 4. Current measure: `Who Works Where Pills HTML v.6`

Status: **[Decided]**. The full text is committed alongside this log as `who-works-where-pills-html-v6.dax` (the owner's model carries it under the name `v.6.3`; names are only labels). `who-works-where-pills-html-v5.dax` is kept for reference.

Behaviour:
- Rows = every `DimProject[Macro Thematic Focus]`; columns = every `BridgeProjectCountry[Country]`. Selected ones first (sorted by count), greyed ones after.
- Cell = top 3 agencies as pills (ranked by project count, then name) + "+N" chip. Empty cell on an active row/column = red "no project". Cell with projects but no agency = "agency not recorded".
- Flags are inline SVG, hardcoded for the 9 beneficiary countries: a 10th country needs its flag added to the `SWITCH` by hand (grey placeholder otherwise). Theme icons, colours and definitions match by `CONTAINSSTRING` on the theme name; a renamed theme gets no icon, fallback colour `#8C6D1E`, and an empty definition.
- Fonts: Inter via Google Fonts `@import`. Colours as in §5.
- Small inline `<script>` (technique taken from the owner's CV report measure: overlay + `classList.toggle`).

**Architecture note (important, learned the hard way):** the measure once failed with *"Function 'PLACEHOLDER' encountered a Text that exceeds the maximum allowable length"*. Fix: heavy HTML is written **once** and reused by id. Each agency logo is one `<img id='lg-<agency>'>` (`_logos`), each project is one card (`_projCards`), and each pill's list (`_lists`) holds only project codes. The script assembles popups at runtime. Keep this pattern when adding anything. Also keep the JS **free of double quotes** (it sits inside a DAX string).

Data-model notes:
- Project donor and benefiting countries are fetched per project with `CALCULATE(SELECTEDVALUE(...), REMOVEFILTERS(), DimProject[Project Code] = code)`. Adding `FactProjects` columns to `SUMMARIZECOLUMNS` caused an "Error fetching data" earlier.
- Assumes one agency per project row via `SELECTEDVALUE`; a project with several agencies would show a blank agency/logo in its detail card.
- Logos come from `DimProject[SharePointLogoURL]`; heights per agency are set in `_logos` (IOM, INTERPOL, UNOCT, UNODC, CSSM).

Version history:
| Version | What changed |
|---|---|
| v1–v5 | Country-first pill grid (see 2026-11-03 notes): responsive height, icons, row hover, selected-first greying, count bubble, KPI-style theme panels |
| v6 (`v.6`, model `v.6.3`) | Theme-first flat grid, crosshair hover, icon legend + definition pop-ups, pill -> project list -> project detail, logos, "+N" expansion, dynamic footer, greyed corner icons |

## 5. About Page design system (extracted from the owner's own measures, for reuse)

- **Fonts:** Inter (UI, numbers, tabular-nums), Newsreader (serif display), occasionally Public Sans — via `@import` inside each measure's `<style>`.
- **Primary accent:** `#fede59` (gold) — CTA, badges, hover states.
- **Secondary accent:** teal/green (`#3E7E72`, `#2F6459`, `#7FD3C0`) — reserved for "active" status.
- **Macro theme colours:** Border Governance `#4A3810`, Law Enforcement `#8C6D1E`, Community Engagement `#C9A227`, Cross-Border `#F5D061`. Short label is **"Community Engagement"**.
- **Card chrome:** white, `#E4E2DB` border, 14–22px radius; hover brightens to `#E3C94F` or flips solid gold.
- **Type details:** uppercase labels with `.07–.14em` letter-spacing, weight 600–700, tabular-nums on numbers.
- **Icons:** rounded-square badges, pale yellow at rest, solid gold on hover.

## 6. Data caveats carried through every version

- Only **7 of 43** IP-SCG implementing partners have shared data — an empty cell may be unreported work, not a confirmed gap.
- **Decision [Confirmed] 2026-11-03:** the grid must not depend on partner coverage data. No live "x of 43 reporting" figure or completeness indicator. The "+N" and "other partners might add data" idea is why the +N chip is clickable. Accepted risk: the red "no project" pill can be read as a confirmed gap; a static one-line footnote is the cheapest fix if it matters later.
- Planned ≠ pipeline: the database's "Planned" count (6 at last snapshot) is not the 8-project IBSM pipeline (3 second-call + 5 matching-funds).
- Projects are counted by benefiting country. A project can have multiple agencies and donors, so pill counts can exceed the project total. Each macro theme maps to **one** category per project, so theme counts add up.
- **[Snapshot] 2026-11-03, Active + Planned, all countries:** 55 projects; Law Enforcement 23, Border Governance 18, Community Engagement 12, Cross-Border 2. Nigeria has few projects and no activities (joined recently).
- **[Snapshot] 2026-11-04 screenshot, Active + Planned, Border Governance only:** 18 projects; Côte d'Ivoire 9, Mali 9, Burkina Faso 8, Ghana 8, Niger 7, Togo 7, Benin 6, Senegal 6, Nigeria 3.

## 7. Separate bug flagged (not part of this redesign)

In the owner's live report, the **Funding Entity slicer appears to be bound to `Macro Thematic Focus`** instead of the funding-entity column — it lists the four macro themes. Needs a fix on the field well. **[Open]**

## 8. Open items

- [ ] **Confirm in Power BI:** "+N" expansion and "- fewer" toggle; greyed corner icons when a single theme is selected.
- [ ] **Verify pill count = list rows.** Example: INTERPOL, Côte d'Ivoire, Law Enforcement should list 10 projects in the Active + Planned view. The lists use a `SUMMARIZECOLUMNS` table (`_pl`); not verified against a DAX engine.
- [ ] **Performance / length** with all projects embedded as cards; re-check if the project count grows (string-length limit, see §4).
- [ ] `:has()` crosshair hover needs a Chromium/WebView2 renderer; confirm in Power BI Service as well as Desktop.
- [ ] Logos load from SharePoint URLs; confirm they render for other viewers (permissions).
- [ ] Expanded "+N" cells make the whole row taller (grid rows are shared); acceptable for now.
- [ ] **GitHub write access:** Claude cannot write to this repo, so files are handed over and committed by the owner. Commit `who-works-where-pills-html-v6.dax` next to this log.
- [ ] A new country needs its flag added by hand to the `SWITCH` in `_header`.
- [ ] `ALL()` overrides report/page-level filters on Country and Theme; confirm none are used.
- [ ] Not yet decided: does the pill grid replace the separate "Projects Data" table, or sit alongside it?
- [ ] Not yet decided: should the Ended toggle persist, or reset to Active + Planned on load?
- [ ] Rank logic in the donut legend not shared with the grid (grid sorts by live count).
- [ ] Optional: static footnote that the red "no project" may be unreported work.
- [ ] Wording: title says "for West Africa" while the description says West and Central Africa; Navigation Guide says "8 focus countries" vs 9. (Page-level, deferred.)
- [ ] Rest of the page (KPI tiles, donor/agency charts, left rail) still needs to match the v6 grid style; deferred.

## 9. Prototypes (private to the owner's claude.ai account)

- v6 directions A/B/C: https://claude.ai/artifact/SnxTKcgQbTEEmCPVo54Kxj
- v6 direction C flattened (Paper palette): https://claude.ai/artifact/RJzuCjDQfWkgqZBvdwGbpW
- Older: count-lead / heat tint / density bar — https://claude.ai/artifact/WmaanK8LAsBMjxNbNRoUJs , https://claude.ai/artifact/BFuUexSrXVSKLLc2g6r7Hm , https://claude.ai/artifact/CEZaCpHcmW2hwZ57NwwSP7 ; theme panels A–D — https://claude.ai/artifact/M6ucPeqHvrYNuXs66a7zV4
