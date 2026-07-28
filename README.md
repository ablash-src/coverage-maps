# Coverage Maps

Interactive two-layer maps showing demand (units) and store coverage across marketplaces.

## Live Maps

- **UK**: https://ablash-src.github.io/coverage-maps/
- **DE**: https://ablash-src.github.io/coverage-maps/de.html
- **ES**: https://ablash-src.github.io/coverage-maps/es.html
- **FR**: https://ablash-src.github.io/coverage-maps/fr.html
- **IT**: https://ablash-src.github.io/coverage-maps/it.html
- **UK**: https://ablash-src.github.io/coverage-maps/uk.html
- **US**: https://ablash-src.github.io/coverage-maps/us.html

## How It Works

1. GeoJSON files are generated from returns + store data using Python notebooks
2. `generate_map.py` converts each GeoJSON into a self-contained HTML map
3. HTML maps are pushed to this repo
4. GitHub Pages serves them as live HTTPS websites
5. These URLs can be embedded in QuickSight dashboards via the "Custom visual content" (iframe) widget

## Map Layers

Each map includes:
- **Units (demand)** — light blue circles, sized proportionally to return volume
- **Total Store Count** — red-orange circles, max radius 1/10th of units
- **Partner stores** — individual toggleable layers per partner, auto-detected from the GeoJSON properties

## How to Update

1. Regenerate the GeoJSON from your Python notebook
2. Run the map generator:
   ```
   python "C:\Users\amlash\Downloads\map_generator\generate_map.py"
   ```
3. Pick the marketplace GeoJSON file
4. Copy the generated HTML to this repo folder
5. Commit and push:
   ```
   git add .
   git commit -m "Update maps"
   git push
   ```
6. Maps auto-deploy in ~1 minute

## Embedding in QuickSight

1. In your QuickSight analysis, add a "Custom visual content" widget
2. Paste the HTTPS URL for the marketplace map
3. The map is interactive (zoom, pan, hover tooltips, layer toggles) inside the dashboard

## Tech Stack

- Leaflet.js for map rendering
- CartoDB Dark Matter basemap
- GitHub Pages for hosting
- No server required — fully static HTML
