# IBSM 3.0 Dashboard: context for review and modernization

Purpose of this file: give Claude (or any reviewer) the full context needed to review, critique and modernize the IBSM Dashboard. Read it before looking at any screenshot, measure or design.

Status key used below: **[Confirmed]** stated by the dashboard owner. **[Inferred]** deduced from screenshots or materials, to verify. **[Snapshot]** a figure that changes as data is recorded; never hardcode it.

---

## 1. What the dashboard is for

The dashboard gives one shared, evidence-based picture of the portfolio of projects and activities that the Integrated Border Stability Mechanism (IBSM) coordinates across West Africa. It shows who is doing what and where, so that gaps, overlaps and chances to connect work across countries and themes become visible. [Confirmed]

It is a **visibility and coordination tool, not a results tracker**. Pipeline (planned) work must never be presented as achieved results.

The financial partners are a formal body, and they use the dashboard picture to inform their funding decisions. The dashboard shows a wider set of funders than the formal financial partners, because it records every funder of every project. [Confirmed]

### Audiences
- **Financial partners** (formal body): where funding already goes, where gaps and overlaps are, to decide new funding.
- **National authorities / National Focal Points:** how their validated priorities are being covered.
- **Implementing partners (IP-SCG members):** where their expertise fills a gap or complements others.
- **Communities and civil society:** secondary audience.
- Visitors at events (for example the Abidjan Border Forum, 16 Oct 2026) who are new to IBSM.

---

## 2. About the IBSM (background)

Source: the IBSM Stand Coach material (Oct 2026). Figures here are as of mid-October 2026 and move over time.

- Launched **September 2023**. A joint platform that aligns international support with national priorities by strengthening coordination among international partners and participating States.
- IBSM **coordinates; it does not implement.** Projects are delivered by implementing organizations under their own mandates. Four core agencies, plus a wider partnership.
- **Nine beneficiary countries:** Senegal, Mali, Burkina Faso, Niger, Togo, Côte d'Ivoire, Ghana, Benin, Nigeria (Nigeria joined recently).
- **Six financial partners:** United Kingdom, Netherlands, Italy, United States, Germany, Canada.
- **Joint Baseline Assessments (JBAs):** seven complete (Benin, Burkina Faso, Côte d'Ivoire, Ghana, Mali, Niger, Togo); Senegal and Nigeria ongoing. They give a common evidence base; validated recommendations guide programme adjustments and new projects.
- **Pipeline:** eight projects (three under the second call for proposals, five under matching funds). This is future programming, not results.
- **IP-SCG** (Implementing Partners Strategic Coordination Group): where partners share information, identify complementarities and align on common geographic and thematic priorities.
- Seeks coherence with the Global Counterterrorism Forum (GCTF) and CSSM mechanisms.
- **Principles:** national ownership, human rights, gender equality, youth participation, accountability.
- **Strategy 2027–2029:** Pillar I strategic coordination (via the Coordination Unit); Pillar II operational action.
- The coach and the report are bilingual (EN/FR).

---

## 3. Report structure (Power BI)

The report is called **IBSM 3.0**. [Confirmed] It covers projects and activities implemented by multiple agencies under the IBSM. Pages:

| Page | What it shows |
|---|---|
| **Landing page** | Overview. Map of the footprint, Key Insight carousel, five KPI tiles (total projects, active projects, focus countries, total activities, implementing partners), Projects Geographic Reach bar chart, Thematic Distribution donut, donor and partner logos, "Explore dashboard" button, "How to read this report" link. |
| **Navigation Guide** | A numbered overlay explaining each visual on the landing page. Built as a DAX measure that returns HTML; the Key Insight text and the number of themes are computed from the current selection, and the rest is static text. |
| **Projects view** (PROJECTS mode) | Left rail of slicers: Countries, Project Status, Thematic Focus, Donors, Funding Entity, Project Code. KPIs: projects, active projects, planned projects, agencies. Funding Sources by # Projects (donors). Organizations category (IBSM / Other IPs) and the agencies. Projects Data table (ID, title, status). Projects by Country matrix (Active / Ended / Planned). |
| **Activities view** (ACTIVITIES mode) | Slicers: Countries, Project Status, Donors, Funding Entity. KPIs: activities, agencies. Sub-national map. Activities Data (searchable by Project Code, Activity Category, Activity Description). Activities Density by Regions (activities and number of IPs per sub-region). Activity Type Ranking (Activity Category) and Sub-Activity ranking. |

Technical notes seen so far: a bridge table `BridgeProjectCountry` links projects to benefiting countries; measure `[# Projects D]`; columns `Macro Thematic Focus` and `Macro Thematic Focus - Short`. The visual style uses a gold and yellow palette on a cream dotted-map header, serif headings, country flags beside country names, and the IBSM logo.

---

## 4. Data model and counting rules [Confirmed]

- **Projects mode:** count projects by the **benefiting country** relationship. A project can benefit several countries at once, so country counts add up to more than the total number of projects.
- **Activities mode:** count activities by **where the activity took place** (country or sub-national region), not by where the project is declared.
- **Projects count is always higher than Activities count** for the same country. This is by design: not all projects have activities recorded yet.
- A project can have **multiple donors** and **multiple implementing agencies**.
- **Project Status:** Active, Planned, Ended.
- **Agency Category:** `IBSM` (core mechanism agencies) or `Other IPs`. [Inferred: IBSM = IOM, UNOCT, UNODC, INTERPOL; Other IPs include MAG, UNITAR, R4P. Verify.]
- **Donor** = high-level funding category. **Funding Entity** = the specific funder within that category.
- **Macro Thematic Focus:** each project is mapped to the **one** category that best describes its objectives, although in reality a project can touch more than one. The donut therefore adds up to 100%.
- **Activity hierarchy:** Activity Category, then Sub-Activity, then Activity Description. Sub-Activity is the broad level within a category (for example Training, Workshop, Equipment); Activity Description is the specific detail of what was done.

---

## 5. Taxonomies (target definitions)

These replace some older labels still visible in the current report.

### Macro Thematic Focus (project level)

| Category | Definition |
|---|---|
| **Border Governance & Institutional Capacity** | The legal, institutional and policy architecture that determines how borders are managed. National border management strategies, standard operating procedures, migration data systems such as MIDAS, national coordination frameworks, capacity building of border personnel, and the physical infrastructure (border posts, solar systems, patrol equipment, IT systems) that gives the state a functioning presence at points of entry. |
| **Community Security & Resilience** | The human dimension of border stability, investing in the communities most exposed to violence, recruitment, displacement and marginalization. Community dialogue and social cohesion, civil-military trust-building, farmer-herder conflict resolution, livelihoods support, access to health and education, legal identity including civil registration, resilience indices, explosive ordnance risk education, youth and women's empowerment, psychosocial support. |
| **Cross-Border Cooperation & Evidence** | The connective tissue that makes the other pillars work across borders and over time. Cross-border coordination mechanisms, joint border commissions, tri-border meetings, regional conferences, high-level diplomatic facilitation, study visits, security perception studies, threat assessments, regional indices, programme evaluations, learning systems. |
| **Integrated Law Enforcement Capabilities & Investigative Support** | The operational response to terrorism, organized crime, illicit trafficking and cross-border security threats. Intelligence, counter-terrorism investigations, IED detection and forensics, weapons and ammunition management and destruction, counter-terrorism financing, biometric identification, digital forensics, INTERPOL operations and tools, fusion cells, passenger data systems, prison radicalization detection, professionalization of border security forces, joint law enforcement operations. |

Short labels used on the landing page donut: Border Governance, Law Enforcement, Community Engagement, Cross-Border. [Inferred; the short label for Community Security & Resilience should probably be aligned with the full name.]

### Activity Category (activity level)

| Category | Definition |
|---|---|
| **Capacity Building & Training** | Training programs for officers, authorities and community members; provision of equipment; Border Management Information Systems and databases. |
| **Community Engagement & Policing** | Awareness raising, sensitization and community information sharing; outreach programmes, special events and information materials; feedback and complaint mechanisms. |
| **Cooperation & Dialogue** | Cross-border meetings and workshops, coordination forums, multi-stakeholder engagement; interagency, regional and international cooperation; international and regional conferences. |
| **Infrastructure Development** | Construction and rehabilitation of facilities; mobile infrastructure. |
| **Legal, Policy Support & Advisory** | Technical assistance for policy development, implementation and institutional strengthening (SOPs, regulations, manuals, protocols, guidelines, directives, contingency plans); consultations; advice and advocacy. |
| **Research, Assessments & Analysis** | Studies, surveys, needs assessments, analysis. |
| **Investigative & Operational Support** | Intelligence gathering, case analysis, forensic assistance and technical expertise to detect, investigate and prosecute cross-border crimes, trafficking networks and security threats; adjudication, corrections and prisons. Operational support including deployment, use and interconnection of operational systems and databases. |
| **Institutional Service Delivery** | Support to legal identity documentation issuance and mobile legal identity service delivery. |

The two taxonomies are independent: an activity category such as training can serve any macro theme. Do not assume a one-to-one mapping.

---

## 6. Data caveats to keep visible

1. **Incomplete reporting.** The landing page shows 43 implementing partners (the IP-SCG membership), but only 7 agencies have shared data so far. Most IP-SCG partners have not yet shared their data. A "gap" on the dashboard may simply be unreported work. This matters because funders use the picture for decisions. [Confirmed]
2. **Figures keep evolving** as new projects and activities are recorded. Do not hardcode numbers; treat all figures as snapshots.
3. **Projects vs Activities** counts differ by design (see section 4).
4. **Donors on the dashboard are not the same as the formal financial partners.** The dashboard lists all funders of any project (including, for example, Japan, EU, Norway, Denmark, France, UN, Multi-Donor and the IBSM Trust Fund).
5. **Planned projects in the database (6 at the time of the snapshot) are not the same as the 8-project pipeline** described in the coach (3 second call plus 5 matching funds). Keep the definitions distinct.
6. **Nigeria** joined recently: few projects and no activities recorded yet, and it is missing from the Activities country slicer.
7. The **map is illustrative**; boundaries do not imply official endorsement. Keep the disclaimer.

---

## 7. Snapshot (2 Nov 2026), illustrative only [Snapshot]

- Landing page: 139 projects, 49 active (35%), 9 focus countries, about 1.4K activities, 43 implementing partners.
- Projects view: 139 projects, 49 active, 6 planned, 7 agencies.
- Projects by country (total): Côte d'Ivoire 81, Ghana 57, Benin 51, Burkina Faso 50, Togo 49, Mali 49, Niger 47, Senegal 46, Nigeria 5.
- Macro themes: Border Governance 35%, Law Enforcement 35%, Community Engagement 19%, Cross-Border 11%.
- Activities: about 1.35K, led by Capacity Building & Training (675).

---

## 8. Observations to check during the review [Inferred, from screenshots]

- The title reads "for West Africa" everywhere; the report description mentions West and Central Africa. Confirm intended scope wording now that Nigeria is in.
- The Navigation Guide still says "8 focus countries", while the dashboard shows 9.
- Activities KPI shows "1K" and the landing page "1.4K". Likely rounding; make the formatting consistent.
- Legacy activity labels ("Research & Assessments", "Institutional Service") coexist with the new ones.
- "Implementing partners" (43) and "Agencies" (7) are used without explanation. Label them clearly and show data coverage.
- KPI numbers in the Projects view are pale gold on cream; check contrast.
- The country flag strip in the header looks clipped at the right edge in the Projects view.
- The Activities view has no Thematic Focus or Project Code slicers, unlike the Projects view.
- Landing page has no way to switch directly into Projects or Activities mode; check the "Explore dashboard" flow.
- No bilingual (EN/FR) support, while the audience and the coach are bilingual. Confirm whether this is wanted.

---

## 9. Guidance for the review and modernization

Keep:
- The counting rules in section 4 (they are the report's integrity).
- The honesty of the Projects vs Activities distinction and the "pipeline is not results" rule.
- The neutral map disclaimer.

Look for opportunities to:
- Show **data completeness** (how many of the 43 partners have reported) wherever gaps or overlaps are shown.
- Make **gaps and overlaps** easy to read, for example country by macro theme, or sub-national activity density, since funders use this for decisions.
- Keep figures **data-driven** rather than hardcoded, including the Key Insight text and the Navigation Guide.
- Clarify terms (Donor vs Funding Entity, Agency vs Implementing Partner, Activity Category vs Sub-Activity) for first-time viewers.
- Improve accessibility (contrast, keyboard use, screen readers), small-screen behavior and visual consistency across pages.
- Consider EN/FR support.

Open questions for the owner: intended geographic wording (West vs West and Central Africa); whether the dashboard should publish or hide the formal financial-partner list; how often data refreshes; how to present partners who have not yet shared data.
