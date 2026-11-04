# Projects View modernization — running log

Status key: **[Confirmed]** stated by the dashboard owner. **[Decided]** agreed in session, not yet verified in the live report. **[Open]** unresolved.

Last updated: 2026-11-04 (end of day). **Current state in one line: v6.3 is confirmed rendering and working in the live report (pills, project list, detail card); v6.4 (planned-only pill outline, keyboard focus, larger pill text, clearer footer) was produced from v.6.3 at the end of the session and is not yet pasted into or checked in the report.**

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

**Session of 2026-11-04, second half: review and v6.4** (all [Decided] unless noted):
1. **Review of the grid against the dashboard's promise** (visibility and coordination for funders, not a results tracker). Score went 74 -> 84 -> 88 as screenshots and owner clarifications came in (§10). Expected after v6.4: about 90-92.
2. **Owner clarifications [Confirmed]:** the 7-of-43 reporting gap is not to be counted against the grid (consistent with §6), and the red "no project" pill is self-explanatory. The pills are clickable: hover shows a gold highlight and a tooltip ("UNOCT: 3 projects (click for the list)"), a click opens the project list, a row click opens the detail card. Screenshots show all three levels rendering, with the agency logo.
3. **Check from the screenshots:** UNOCT, Côte d'Ivoire, Law Enforcement shows a pill of 3 and a list of 3 projects (Temp.OCT.001 Planned; UNOCT-2020-II-UAS Active; UNCCT-2017-58 Active). Theme totals 23 + 18 + 12 + 2 = 55 and the shares sum to 100%.
4. **v6.4 changes** (built from the owner's v.6.3 measure):
   - Pills whose projects are all planned get a dashed blue outline (blue matches the Planned badge in the pop-up). The tooltip adds "(planned)" or "(n planned)". The footer shows a "Dashed = planned projects only" legend, only while Planned is in the status selection.
   - Pills are keyboard-focusable (`tabindex`, `role=button`, Enter and Space open the list); visible focus ring on pills, "+N" and corner icons.
   - Pill text 0.857rem -> 0.9rem, pill height 1.857rem -> 1.95rem; pill and cell padding trimmed and the theme column narrowed (24% -> 22%) so "INTERPOL 10" still fits.
   - Footer now starts with "Click an agency to list its projects."
5. **Left as is on purpose:** equal row heights (content-based heights would break the fill-the-height behaviour); slicer naming (see §8).

**Tried and dropped:** per-cell counts as a lead number, heat tint, density bar (all rejected as styles); `FILTERS()`-based row removal; hover tooltip with theme names.

## 4. Current measure: `Who Works Where Pills HTML v.6.4`

Status: **[Decided]**, not yet verified in the live report. The full text is `who-works-where-pills-html-v6.4.dax` (to be committed next to this log). The `who-works-where-pills-html-v6.dax` already in the repo is an older copy; the owner's model carried it as `v.6.3` before v.6.4 (names are only labels). There is no v5 file in the repo.

Behaviour:
- Rows = every `DimProject[Macro Thematic Focus]`; columns = every `BridgeProjectCountry[Country]`. Selected ones first (sorted by count), greyed ones after.
- Cell = top 3 agencies as pills (ranked by project count, then name) + "+N" chip. Empty cell on an active row/column = red "no project". Cell with projects but no agency = "agency not recorded".
- **Planned-only pills (v.6.4):** `_agCells` carries `@pl` (planned projects for that country, theme and agency, computed with `KEEPFILTERS` so the status slicer still applies), `@cls` (` pn` when every project of the pill is planned) and `@ttl` (tooltip text). Assumes the status value is exactly `Planned`; the footer legend test uses `CONTAINSSTRING` on `Plan`.
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
| v6.4 | Planned-only pills (dashed blue outline, tooltip, footer legend), keyboard access and focus ring, larger pill text, "click an agency" footer hint |

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
- **Decision [Confirmed] 2026-11-03:** the grid must not depend on partner coverage data. No live "x of 43 reporting" figure or completeness indicator. The "+N" and "other partners might add data" idea is why the +N chip is clickable. **[Confirmed] 2026-11-04:** the owner considers the red "no project" pill self-explanatory, so no footnote is planned, and reviews should not deduct points for the 7-of-43 gap.
- Planned ≠ pipeline: the database's "Planned" count (6 at last snapshot) is not the 8-project IBSM pipeline (3 second-call + 5 matching-funds).
- Projects are counted by benefiting country. A project can have multiple agencies and donors, so pill counts can exceed the project total. Each macro theme maps to **one** category per project, so theme counts add up.
- **[Snapshot] 2026-11-03, Active + Planned, all countries:** 55 projects; Law Enforcement 23, Border Governance 18, Community Engagement 12, Cross-Border 2. Nigeria has few projects and no activities (joined recently).
- **[Snapshot] 2026-11-04 screenshot, Active + Planned, Border Governance only:** 18 projects; Côte d'Ivoire 9, Mali 9, Burkina Faso 8, Ghana 8, Niger 7, Togo 7, Benin 6, Senegal 6, Nigeria 3.

## 7. Separate bug flagged (not part of this redesign)

In the owner's live report, the **Funding Entity slicer appeared to be bound to `Macro Thematic Focus`** instead of the funding-entity column — it listed the four macro themes. In the 2026-11-04 screenshots the left rail reads "Funding Entities / Select FundingEntity", so it looks fixed; confirm the values list the funders. **[Open, probably resolved]**

## 8. Open items

- [ ] **Confirm in Power BI:** "+N" expansion and "- fewer" toggle; greyed corner icons when a single theme is selected. In the Active + Planned screenshots no cell had more than 3 agencies, so "+N" was never visible; test with Ended included.
- [ ] **Verify pill count = list rows.** Checked once: UNOCT, Côte d'Ivoire, Law Enforcement = pill 3, list 3 rows. Still to check: INTERPOL, Côte d'Ivoire, Law Enforcement should list 10 projects in the Active + Planned view.
- [ ] **Performance / length** with all projects embedded as cards; re-check if the project count grows (string-length limit, see §4).
- [ ] `:has()` crosshair hover needs a Chromium/WebView2 renderer; confirm in Power BI Service as well as Desktop.
- [ ] Logos load from SharePoint URLs; confirm they render for other viewers (permissions).
- [ ] Expanded "+N" cells make the whole row taller (grid rows are shared); acceptable for now.
- [ ] **GitHub write access:** Claude can read this repo (anonymous clone) but cannot write to it, so files are handed over and committed by the owner. Commit `who-works-where-pills-html-v6.4.dax` and this updated log into `claude-context/`. (Per the project instructions, DAX measures are handed over as an artifact page with a Copy button.)
- [ ] A new country needs its flag added by hand to the `SWITCH` in `_header`.
- [ ] `ALL()` overrides report/page-level filters on Country and Theme; confirm none are used.
- [ ] Not yet decided: does the pill grid replace the separate "Projects Data" table, or sit alongside it?
- [ ] Not yet decided: should the Ended toggle persist, or reset to Active + Planned on load?
- [ ] Rank logic in the donut legend not shared with the grid (grid sorts by live count).
- [x] Dropped 2026-11-04: static footnote on the red "no project" pill (owner: self-explanatory).
- [ ] **Confirm v6.4 in Power BI:** (a) "INTERPOL 10" still fits its column in Côte d'Ivoire (if it clips, set the pill font to 0.88rem); (b) the planned-only dashed pills and the footer legend show; (c) Tab, Enter and Space open a pill's list; (d) the status value is exactly `Planned`.
- [ ] **Slicer naming mismatch:** the left-rail "Macro Thematic Areas" slicer shows short labels ("Community Engagement", "Cross-Border"); the grid rows show full names ("Community Security & Resilience", "Cross-Border Cooperation & Evidence"). Fix in the slicer's field (use `Macro Thematic Focus`) or align the `- Short` column. Not a measure change.
- [ ] Left rail again shows the Organizations slicer (§2 says it was removed); confirm whether it is intended.
- [ ] Decided to keep equal row heights (rows fill the visual); revisit only if a content-based layout is wanted.
- [ ] Wording: title says "for West Africa" while the description says West and Central Africa; Navigation Guide says "8 focus countries" vs 9. (Page-level, deferred.)
- [ ] Rest of the page (KPI tiles, donor/agency charts, left rail) still needs to match the v6 grid style; deferred.

## 9. Prototypes (private to the owner's claude.ai account)

- v6.4 measure, copyable page: https://claude.ai/artifact/WCxYFeEtPPTdjvLLmRP1Bv
- v6 directions A/B/C: https://claude.ai/artifact/SnxTKcgQbTEEmCPVo54Kxj
- v6 direction C flattened (Paper palette): https://claude.ai/artifact/RJzuCjDQfWkgqZBvdwGbpW
- Older: count-lead / heat tint / density bar — https://claude.ai/artifact/WmaanK8LAsBMjxNbNRoUJs , https://claude.ai/artifact/BFuUexSrXVSKLLc2g6r7Hm , https://claude.ai/artifact/CEZaCpHcmW2hwZ57NwwSP7 ; theme panels A–D — https://claude.ai/artifact/M6ucPeqHvrYNuXs66a7zV4

## 10. Review scoring of the grid (2026-11-04, for the next session)

Rated against the dashboard's promise (visibility and coordination for funders, not results tracking). Rubric: answers "who works where"; honest counting; drill-down to evidence; readability; accessibility; consistency.

| Round | Score | Why it moved |
|---|---|---|
| Measure and log only | 74 | Coverage gap and red pill counted against it; assumed counts and click affordance were missing |
| With first screenshot | 84 | Coverage and red pill no longer deducted (owner decision); counts per pill already present |
| With hover and pop-up screenshots | 88 | Hover tooltip, project list with status badges and detail card all present; pill count matched list |
| Expected after v6.4 | about 90-92 | Planned-only outline, footer hint, larger text, keyboard access |

Remaining deductions after v6.4: slicer names differ from row names (about 2); equal row heights give little emphasis to dense or sparse themes (about 3); pill text is still small (about 2); pill counts only partly verified and flags are hardcoded (about 1); no EN/FR.
