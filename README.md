# Coverage Maps

Interactive two-layer maps showing demand (units) and store coverage across marketplaces.

## Live Maps

- **UK**: https://ablash-src.github.io/coverage-maps/uk.html
- **DE**: https://ablash-src.github.io/coverage-maps/de.html
- **ES**: https://ablash-src.github.io/coverage-maps/es.html
- **FR**: https://ablash-src.github.io/coverage-maps/fr.html
- **IT**: https://ablash-src.github.io/coverage-maps/it.html
- **UK**: https://ablash-src.github.io/coverage-maps/uk.html
- **US**: https://ablash-src.github.io/coverage-maps/us.html

## Map Layers

Each map includes:
- **Units (demand)** — light blue circles, sized proportionally to return volume
- **Total Store Count** — red-orange circles, max radius 1/10th of units
- **Partner stores** — individual toggleable layers per partner, auto-detected from the GeoJSON properties

## Tech Stack

- Leaflet.js for map rendering
- CartoDB Dark Matter basemap
- GitHub Pages for hosting
- No server required — fully static HTML
