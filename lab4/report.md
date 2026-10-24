# Lab 4 Report: GeoDjango, PostGIS, Admin and Leaflet

## 1. Versions
- Python 3.14.2
- Django 6.1.2
- PostgreSQL 17.11
- PostGIS 3.6.2 (bundled GEOS 3.14.1, PROJ 8.2.1)
- GDAL 3.13.3 and GEOS 3.15.0 (as seen from Django, via OSGeo4W)
- pgAdmin 4 (9.8)

## 2. Installation errors and fixes
1. **PowerShell blocked venv activation** (`Activate.ps1`, SecurityError / UnauthorizedAccess). Fixed with `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
2. **Stack Builder download error** while fetching the PostGIS bundle: "end of response with 8646720 bytes missing". The installer still completed on retry, and `CREATE EXTENSION postgis;` ran correctly.
3. **pgAdmin connection timeout** to localhost:5432 right after installing. The PostgreSQL service had not finished starting; connecting again worked.
4. **OSGeo4W ran but "Nothing needed to be installed"** because GDAL was not selected. Re-ran the installer and selected the gdal package.
5. **IndentationError in models.py** caused by pasted code being indented. Fixed by removing the extra indent.
6. **Admin login failed**: the superuser name was `admi`, not `admin`.
7. **OpenStreetMap tiles returned 403 Access blocked** when the page ran from 127.0.0.1. Fixed by adding `<meta name="referrer" content="origin">` to map.html.

## 3. Checkpoint screenshots
All checkpoint screenshots are in [webgislab4screenshots.docx](webgislab4screenshots.docx).

### Checkpoint 1: Django install page and version

### Checkpoint 2: PostGIS version in pgAdmin

### Checkpoint 3: GDAL/GEOS versions, check, migrate, tables in pgAdmin

### Checkpoint 4: sqlmigrate output and districts_district columns

### Checkpoint 5: Loader output and shell queries
Layer reported 160 features with MultiPolygon geometry; 160 districts saved; Sindh = 29; no census figure = 25; Lahore population 13004135, MultiPolygon, 231 coordinates.

### Checkpoint 6: Admin panel (Sindh filter, Lahore map, pgAdmin check of the edit)

### Checkpoint 7: Leaflet map with tooltip and the api/districts/ request (995 kB)

### Checkpoint 8: Click popup and area query

## 4. Area table (Part 8)
| District | Stored km2 | Geodesic km2 | Web Mercator km2 | Ratio |
|---|---|---|---|---|
| Gwadar | 11574 | 11575 | 14245 | 1.23 |
| Lahore | 1730 | 1730 | 2385 | 1.38 |
| Gilgit | 4016 | 4015 | 6133 | 1.53 |

Stored and geodesic areas agree. Web Mercator inflates areas by about 1/cos(latitude)^2, so the distortion grows northwards: Gwadar (about 25 N) 1.23, Lahore (about 31.5 N) 1.38, Gilgit (about 36 N) 1.53.

## 5. Answers
1. Instead of every visitor downloading the whole GeoJSON file, the server answers only what is asked. `/api/districts/?province=Sindh` filters in the database and returns one province, and `/api/where/?lat=..&lng=..` returns one district's name, province and population for a clicked point without sending any geometry. The data also lives in one place and can be edited through the admin.
2. settings.py is committed to GitHub, so a password written in it would be public. Reading it from the WEBGIS_DB_PASSWORD environment variable keeps the secret out of the repository and lets each machine use its own.
3. `USING GIST` creates a spatial index on the geom column. It lets PostGIS skip districts whose bounding box cannot contain the query geometry instead of scanning all rows. The `geom__contains` lookup behind `/api/where/` (ST_Contains) benefits from it.
4. The geom column accepts only MultiPolygon, and PostGIS rejects a plain Polygon with a geometry type mismatch. Wrapping each Polygon in a one-part MultiPolygon fixes this without changing the map. In my file all 160 features were already MultiPolygon, so the wrap never triggered, but it protects against mixed files.
5. Only trusted people who maintain the data should have an admin login, using limited accounts where possible (for example change but not delete). The public and ordinary users should not, because anyone who can log in can change or delete every row.
6. Leaflet uses [lat, lng]: `setView([30.3753, 69.3451], 5)`, the `maxBounds` corners, and `e.latlng.lat` / `e.latlng.lng` in the click handler. GEOS `Point` uses (lng, lat): `Point(lng, lat, srid=4326)` in `district_at`. The URL avoids confusion by naming the parameters `lat` and `lng`, and the view reads each one by name.
