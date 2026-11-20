# Leitig – Swiss power line identifier

Point your phone at a Swiss high-voltage line and see its route (Trassee), operator and voltage.
There are two modes: a camera viewfinder, or a large compass with labelled lines when you don't use the camera.
You can install it to your home screen: it includes a web app manifest and icons in `icons/`.
It uses GPS and the compass heading, matched against the Swiss Federal Office of Energy (BFE) dataset
"Elektrische Anlagen mit einer Nennspannung von über 36 kV" from api3.geo.admin.ch.

## Run
The page is a single static file (`index.html`), with no build step and no backend.
It must be served over **HTTPS**, because the camera, GPS and compass are blocked otherwise.

- GitHub Pages: Settings → Pages → Deploy from a branch → `main` / root
- Locally on a desktop: `npx serve .`, then use the "Manuell" mode in the settings panel (there are no sensors)

## How matching works
1. Load the BFE line geometries within 1–3 km of you.
2. Cast a ray from your position along the compass heading. The heading is corrected by +3° for magnetic declination, plus the manual offset slider.
3. Rank line segments within ±20° by `angle offset + distance / 100 m`.
4. Show the best match and up to 4 alternatives. Underground cables are excluded unless you turn them on.

## Known limits
- The BFE dataset is incomplete: some operators haven't delivered data yet. It contains nothing below 36 kV.
- Phone compasses are off by 5–15°, and more near steel or cars.
- Only portrait orientation is supported.
- Calling the API straight from the browser hasn't been tested yet.

Data © BFE / swisstopo (opendata.swiss, attribution required).
