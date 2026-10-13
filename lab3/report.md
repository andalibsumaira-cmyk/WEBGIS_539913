# District Dashboard — Lab 3 Report

## Known issue
The Carto Positron basemap currently requires an API key, so it shows "API KEY REQUIRED" watermarks on tiles. OpenStreetMap and Esri Satellite basemaps work correctly. The district and population layers loaded and rendered successfully (confirmed via browser: both the choropleth shading and proportional circles display correctly on OpenStreetMap and Esri basemaps).

## Point layer
district_points.geojson: 35 KB, 160 features confirmed.

## Esri tile URL
Example: https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/5/13/21
Format is /MapServer/tile/{z}/{y}/{x} — row (y) comes before column (x), opposite of OpenStreetMap/Carto's {z}/{x}/{y} convention.

## Answers to Questions

1. A large, thinly populated district would appear dark (high population) on a population-shaded map simply due to its size, misleadingly suggesting high density. A small, crowded district would appear understated despite extreme crowding.

2. Without the square root, a large city's circle would appear vastly oversized relative to population, since circle area grows with the square of the radius — a district with four times the population would look sixteen times larger rather than four times larger.

3. The layer control holds districtGroup (not the raw L.geoJSON layer) because the district layer is rebuilt every time the province filter changes; a control bound to a discarded layer would stop working after the first filter change.

4. Drawing circles in overlayPane instead of a dedicated pane would let polygons drawn or re-styled afterward (e.g. via bringToFront on hover) render on top of the circles, breaking the intended visual layering between density shading and population symbols.

5. map.off can remove showCoordinates because it was assigned to a named function reference, which both on and off can refer to by name. An anonymous function passed directly to map.on has no stored reference, so there is nothing to pass to off to remove it.

6. A metre-based L.circle (25 km radius) represents a real evacuation zone, since it scales correctly with the map at any zoom level. A pixel-based L.circleMarker is better for marking a fixed point like a campus, since it stays a constant visual size regardless of zoom.
