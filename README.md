# Leitig

Point your phone at a Swiss high-voltage line and see its route (Trassee), operator and voltage.

**Live:** https://m-kuehne.github.io/leitig/

## Stack

- Plain HTML, CSS and JavaScript, all in `index.html`. There's no framework, no build step, no backend and no dependencies apart from Google Fonts.
- `manifest.webmanifest` and `icons/` let you install it to the home screen as a web app (standalone display, portrait).
- Hosted as a static site on GitHub Pages. The camera, geolocation and orientation APIs need HTTPS (a secure context).

## Data

The data is the Federal Office of Energy (BFE) dataset *Elektrische Anlagen mit einer Nennspannung von über 36 kV* (geodata no. 219.1), queried live through the swisstopo REST API:

```
GET https://api3.geo.admin.ch/rest/services/api/MapServer/identify
    ?geometryType=esriGeometryEnvelope&geometry=<minLon,minLat,maxLon,maxLat>
    &sr=4326&layers=all:ch.bfe.elektrische-anlagen_ueber_36
    &tolerance=0&returnGeometry=true&geometryFormat=geojson&lang=de&offset=<n>
```

- The API returns about 200 features per call, and `limit` doesn't raise that. The app pages with `offset` until a page comes back short.
- The results include lines (`MultiLineString`) and stations (polygons and points). Only line geometries are used.
- Attributes used: `bezeichnung` (route), `eigentuemer` (operator), `spannung` (e.g. `S380kV`), `leitungtyp` (overhead line / underground cable), `stromnetztyp` (general grid / railway power), `frequenz`, and `fid`.
- Lines are re-fetched when you move more than 300 m, or when you zoom the compass beyond the loaded radius.
- Coverage is incomplete: some operators haven't delivered data yet. There's nothing below 36 kV.

## Heading and tilt

- **iOS:** `deviceorientation` plus `webkitCompassHeading` (magnetic). Permission is requested with `DeviceOrientationEvent.requestPermission()` from a user gesture.
- **Android / Chromium:** `deviceorientationabsolute`. The direction out of the back camera is computed from the ZXY rotation matrix (α, β, γ):
  ```
  v = -R·ẑ = ( -(cosα·sinγ + sinα·sinβ·cosγ),
               -(sinα·sinγ − cosα·sinβ·cosγ),
               -cosβ·cosγ )               // x=E, y=N, z=up
  heading = atan2(vx, vy),  pitch = asin(vz)
  ```
  When |pitch| > 55° (phone held flat), the heading is taken from the phone's top edge (`-α`) instead.
- Magnetic declination of +3° (approximate for Switzerland) is added, then a user offset (±30°). The heading is smoothed with an exponential moving average on sin/cos, so it doesn't jump at 0°/360°.

## Matching

1. Project the line vertices into local east/north metres around the user, using a flat approximation (fine at a few km).
2. Cast a ray from the user along the heading. For each segment, compute the ray/segment intersection with 2D cross products. If the ray misses, use the angular offset to the nearer endpoint.
3. Discard anything more than 20° off. Score = `offset_deg + distance_m / 100`. Keep the best segment per feature (`fid`) and show the top 5.
4. The expected elevation of the wires is shown as `atan(30 m / distance)`, as a plausibility check against the measured pitch.

Coordinates convert between WGS84 and LV95 (EPSG:2056) with swisstopo's approximate formulas, accurate to about 1 m. This is only used for map links and when a feature comes back in LV95.

## UI modes

- **Camera:** rear camera via `getUserMedia({facingMode:'environment'})`, crosshair, a small north-up radar, and results.
- **No camera:** a heading-up compass with labelled lines, pinch-zoomable from 250 m to 5 km. It's used when you choose it at start or when camera access is denied.
- Debug panel: raw sensor values, API stats, candidate scores, and a manual position/heading mode for testing on a desktop.

## Run locally

```bash
npx serve .            # or any static server
```

Desktop browsers have no compass. Use *Einstellungen & Debug → Manuell*. To test sensors on a phone, serve over HTTPS (e.g. GitHub Pages, or a tunnel).

## Known limitations

- Phone magnetometers are typically off by 5–15°, and more near vehicles, pylons or reinforced concrete.
- Portrait orientation only. Landscape and `screen.orientation` aren't handled.
- Online only. Nothing is cached.
- Data accuracy depends on what the operators delivered to the BFE.

## Attribution

Data © BFE / swisstopo, available on [opendata.swiss](https://opendata.swiss/en/dataset/elektrische-anlagen-mit-einer-nennspannung-von-uber-36-kv). Use is permitted, including commercial use; attribution is required.
