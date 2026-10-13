
# District Explorer — Lab Report

## File Size Comparison

| Version | What was done | Size | % of A |
|---|---|---|---|
| A | Original (pre-processed GeoJSON) | 912 KB | 100% |
| B | Precision 5 | 1,096 KB | 120% |
| C | Simplified ~100m, precision 5 | 1,062 KB | 116% |
| D | Simplified ~1km, precision 5 | 258 KB | 28% |

Note: Versions B and C did not shrink relative to A because the source file was already compact/minified with near-5-decimal precision. QGIS's export re-serializes the file with its own formatting overhead. The real size reduction comes from geometry simplification (Version D), not precision alone.

## Classification
5-class choropleth by `pop_2023`: under 300k, 300k–750k, 750k–1.5m, 1.5m–3m, over 3m, plus grey for missing census data (Azad Kashmir, Gilgit-Baltistan, Lehri — not covered by the 2023 census).

## Live Site
https://andalibsumaira-cmyk.github.io/WEBGIS_539913/

## Answers to Questions

1. Version D would be the wrong choice for a detailed local or city-level view, where ~1km simplification would visibly distort small district boundaries.

2. The GeoJSON is one single request carrying the full vector dataset as structured text. Tiles are dozens of small, independent, pre-rendered 256×256 pixel raster images stitched together, each requested separately.

3. A heading naturally sizes to its text content. An empty `<div>` has no content, so without explicit CSS height it collapses to 0 pixels and Leaflet renders nothing.

4. Without removing the old layer first, each redraw would stack a new layer on top instead of replacing it, slowing the map and showing overlapping/duplicate districts.

5. Two causes for a classmate's districts all being one colour: (a) population stored as text, so numeric comparisons fail and all districts fall into the same class; (b) all values are null, so all districts get the grey "no data" colour. Checking the attribute table (text vs blank) tells them apart.

