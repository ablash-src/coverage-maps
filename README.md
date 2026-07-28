# Coverage Maps

Interactive two-layer maps showing demand (units) and store coverage across marketplaces.

## Live Maps

* **DE**: https://ablash-src.github.io/coverage-maps/DE\_Returns\_Store\_Mapping\_map.html
* **ES**: https://ablash-src.github.io/coverage-maps/ES\_Returns\_Store\_Mapping\_map.html
* **FR**: https://ablash-src.github.io/coverage-maps/FR\_Returns\_Store\_Mapping\_map.html
* **IT**: https://ablash-src.github.io/coverage-maps/IT\_Returns\_Store\_Mapping\_map.html
* **UK**: https://ablash-src.github.io/coverage-maps/UK\_Returns\_Store\_Mapping\_map.html
* **US**: https://ablash-src.github.io/coverage-maps/US\_Returns\_Store\_Mapping\_map.html

## Map Layers

Each map includes:

* **Units (demand)** — light blue circles, sized proportionally to return volume
* **Total Store Count** — red-orange circles, max radius 1/10th of units
* **Partner stores** — individual toggleable layers per partner, auto-detected from the GeoJSON properties

## Tech Stack

* Leaflet.js for map rendering
* CartoDB Dark Matter basemap
* GitHub Pages for hosting
* No server required — fully static HTML

