# Coverage Maps

Interactive two-layer maps showing demand (units) and store coverage across marketplaces.

## Live Maps

- [DE](https://ablash-src.github.io/coverage-maps/DE_Returns_Store_Mapping_map.html)
- [ES](https://ablash-src.github.io/coverage-maps/ES_Returns_Store_Mapping_map.html)
- [FR](https://ablash-src.github.io/coverage-maps/FR_Returns_Store_Mapping_map.html)
- [IT](https://ablash-src.github.io/coverage-maps/IT_Returns_Store_Mapping_map.html)
- [UK](https://ablash-src.github.io/coverage-maps/UK_Returns_Store_Mapping_map.html)
- [US](https://ablash-src.github.io/coverage-maps/US_Returns_Store_Mapping_map.html)

## Map Layers

Each map includes:

- **Units (demand)** - light blue circles, sized proportionally to return volume
- **Total Store Count** - red-orange circles, max radius 1/10th of units
- **Partner stores** - individual toggleable layers per partner, auto-detected from the GeoJSON properties

## How to Update

1. Regenerate the GeoJSON from your Python notebook
2. Run: `python "C:\Users\amlash\Downloads\map_generator\generate_map.py"`
3. Pick the marketplace GeoJSON file
4. Copy the generated HTML to `C:\Users\amlash\Downloads\coverage-maps\`
5. Open GitHub Desktop, commit, and push

Maps auto-deploy in ~1 minute.

## Tech Stack

- Leaflet.js for map rendering
- CartoDB Dark Matter basemap
- GitHub Pages for hosting
- No server required - fully static HTML
