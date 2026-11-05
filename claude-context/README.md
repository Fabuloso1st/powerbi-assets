# Landing Page map (IBSM Dashboard, Deneb / Vega)

Tilted West Africa map for the landing page of the Integrated Border Stability Mechanism (IBSM) Power BI dashboard. It highlights the 9 focus countries, labels them, and cross-filters the rest of the page.

- **Visual:** Deneb custom visual, Vega language (not Vega-Lite)
- **Spec file:** `spec/landing_page_map.json` (about 40 KB)
- **Last verified:** 2026-11-05, in a local Vega 6.4.0 renderer (the version Deneb's editor showed). Not run end to end in Deneb by the author of the spec; see "Status".

## What it shows

- Africa only, tilted 50° so it reads as seen from above, framed on West Africa.
- 9 focus countries filled pale gold, each with a name tag and a dot: Côte d'Ivoire, Ghana, Benin, Burkina Faso, Togo, Mali, Niger, Senegal, Nigeria.
- Every other country is white on a light grey ocean.
- Morocco and Western Sahara are separated by a dotted line along 27°40′N, as on the original dashboard map. This is a disputed territory, so the split must not change without sign-off.
- Clicking a country filters other visuals; other visuals filter the map. A selected country turns a darker gold.

Dashboard disclaimer shown under the map: "This map is for illustration purposes only. The boundaries do not imply official endorsement or acceptance by the IBSM or its agencies."

## Power BI setup

1. Add a Deneb visual and choose **Vega** as the provider.
2. Put `BridgeProjectCountry[Country]` in **Values**. It is the only field the spec reads. Do not add `# Projects`: the bubbles that used it were removed.
3. Paste the spec into the Specification tab, press play.
4. **Project setup > Cross-filtering:** on, management **Simple**, data point limit 50 (there are 9 rows).
5. **Project setup > Tooltips:** on. In the visual's Format pane, Tooltips type = Report page, and pick the tooltip page (that page needs "Allow use as tooltip page").
6. **Other visuals filtering the map:** Format > Edit interactions, select the source visual, click the Filter icon on the map. The default is Highlight, which does nothing here because no measure is in the Deneb fields.

Cross-highlighting is toggled on in Project setup but unused.

## Controls (top of the spec, `signals`)

| Signal | Default | Effect |
|---|---|---|
| `zoom` | 1 | 1 fits the countries to the visual; 1.2 zooms in 20% |
| `panX` | 0 | Pixels, positive moves the map right |
| `panY` | 45 | Pixels, positive moves the map down |

Pitch cannot be changed here. It is baked into the geometry (see below). `k` is the computed scale, `min(width * 0.9 / 317.8, height * 0.84 / 134.9) * zoom`; the two numbers are the width and height of the 9 countries in baked units.

## Look

| Item | Value |
|---|---|
| Ocean (`background`) | `#F0F0EF` |
| Unmapped countries | `#FFFFFF` |
| Focus countries | `#F7ECC2`, selected `#F0D77A` |
| Country borders | `#A59E8D`, width 1.3 |
| Morocco / W. Sahara line | grey underlay `#A59E8D` width 1.5, white dashes `[2,4]` width 1.4 (left untouched on purpose when country borders were thickened) |
| Label tag | `#5B5646` at 80% opacity, radius 3, height 19, width = name length x 6.2 + 14 |
| Label text | white, bold, 11 px, Segoe UI |
| Dots | `#B3A47F`, outline `#7F6A2C`, size 42 |
| Leader lines | only Togo and Benin, `#5B5646` width 0.8 |

Label placement is in the `centroids` data: `x`,`y` is the anchor in baked units, `dx`,`dy` the tag offset. Default offset is `(0, -10)`. Togo is `(10, 12)` and Benin `(16, -12)`, which also switches on their leader lines (drawn when `dx != 0`).

Card styling around the map (white, 1px `#E4E2DB`, 14px radius, Inter, 28px `#FFF6CC` icon circle) comes from the KPI strip measure on the same page and is set in Power BI, not in the spec.

## How the spec works

- **Data:** `dataset` (Deneb rows) -> `world` (TopoJSON, joined to the rows by country name) and `labels` (rows joined to `centroids`).
- **Join:** each shape copies `Country`, `__row__` and `__selected__` from the matching row. `__row__` is what lets Deneb's Simple cross-filtering resolve a click on a shape to a dataset row. Shapes with no row (grey countries) have no tooltip and do nothing on click.
- **Name matching:** a `shapeName` formula maps "The Gambia", "Gambia", "Cote d'Ivoire", "Côte d'Ivoire" (straight and curly apostrophes) and "Ivory Coast" to the map's spelling. Other names must match the map exactly.
- **Projection:** `identity`, not geographic. The geometry is already projected and tilted, so the spec only scales and translates.
- **Marks, in draw order:** country shapes, Western Sahara border underlay, its white dashes, leader lines, dots, tag rectangles, tag text.
- `tooltip` on the shapes is `datum` for mapped countries, `null` otherwise, so the report-page tooltip appears only on focus countries.

## How the geometry was built

Done offline, not in the spec. The scripts are not in this repo; the parameters are recorded so it can be recreated.

1. **Source:** `world-atlas` `countries-50m.json` (Natural Earth, public domain; package under ISC).
2. **Morocco / Western Sahara:** merged, then re-split at latitude 27.6667°N to match the original dashboard map. The detailed data assigns part of Western Sahara to Morocco, so using it as is would be wrong. The dotted line is the cut.
3. **Africa only:** European and Asian countries removed, rest clipped to lon -34 to 42, lat -14 to 40.
4. **Tilt:** Mercator (scale 560, centre lon -1.5, lat 14.2), then pitched 50° with perspective distance 850. This is a baked transform, not a camera.
5. **Recentred** on the bounding box of the 9 countries.
6. **Tiny islands** under 3 px dropped (never for focus countries).
7. **TopoJSON:** built with `topojson-server`, `presimplify` + `simplify` (area 0.05), quantized to 10000. Using shared arcs matters: simplifying each country on its own made neighbouring borders drift apart when zoomed in.
8. **Western Sahara border** extracted as a mesh from the same topology.

Changing the pitch means re-running the pipeline with a new angle and pasting in the new geometry and the two numbers in `k`.

## Status and known limits

- **Confirmed in Power BI by the report owner:** the map filters other visuals; other visuals filter the map once interactions are set to Filter.
- **Not confirmed:** the tooltip page on hover, and the final look of the labels in Deneb (checked in a Vega renderer only).
- A country in the data that is not one of the 9 (for example Gambia) is filled gold but gets no label.
- Label tag widths are estimated from character count; a different font can make tags tight.
- At high zoom-out or steeper tilts the top edge is blank, because Europe was removed.
- The tooltip passes the whole shape row, geometry included. Trimming it could speed hover slightly but was left alone, since the exact fields Deneb needs for the tooltip lookup were not verified.
- Performance was measured and is not a problem: about 7 ms to build, under 3 ms to draw. Slowness in Power BI is Deneb loading, not this spec.

## Gotchas found along the way

- Use `pbiContainerWidth` / `pbiContainerHeight`. Deneb logs "Migrated legacy pbiContainer signal reference(s)", which is harmless. `denebContainerWidth` is not recognised in this version.
- A Vega data set must be defined before any data that looks it up, or Deneb reports `Undefined data set name`.
- Do not build a scale from data that is itself derived from that scale ("Cycle detected in dataflow graph").
- Cross-filtering needs the clickable mark to carry `__row__`. A basemap shape without it is not clickable.
- The first versions were named `pitch30` but were built at 50°. Only the last file is named correctly.

## History

| Step | Change |
|---|---|
| 1 | Simple cross-filtering on a Vega map; dataset rows joined onto basemap shapes |
| 2 | West Africa map with 9 countries, framed to match the original Azure Maps visual |
| 3 | Styled like the world map: white borders, dashed Western Sahara line |
| 4 | Pitched geometry, then Natural Earth 50m for sharper borders |
| 5 | Western Sahara split restored to the original 27°40′N line |
| 6 | Africa-only geometry, zoom/pan controls |
| 7 | Ocean `#F0F0EF`, unmapped `#FFFFFF`, borders 1.3, name tags, larger `#B3A47F` dots |
| 8 | TopoJSON with shared borders (optimized file, 40 KB) |

Explored and rejected: a gold footprint card, a raised plane with shadow, a projects-by-country choropleth with spotlight, and proportional bubbles. The bubbles were built and later removed, which is why `# Projects` is no longer a field.

## Suggested repo layout

```
Landing Page map/
  README.md
  spec/landing_page_map.json
```
