# OpenTTD-Heightmap-Generator

![2026-08-06 115833.png](https://files.seeusercontent.com/2026/08/06/lg7X/2026-08-06-115833.png)

🌟 Project Overview
OpenTTD Heightmap Generator is a modern, high-precision web tool designed to generate realistic 8-bit heightmaps and extract real-world town data for OpenTTD.Powered by OpenStreetMap (OSM) vector data and Terrarium DEM global elevation tiles, it enables players to select any geographic region on Earth, fine-tune terrain smoothing and water levels, and export game-ready PNG heightmaps along with structured JSON town configurations in one click.

### Try Out: https://babel-ttt.github.io/OpenTTD-Heightmap-Generator/

### Note:

* Do not select an area that is too large. It is recommended to keep both the length and width under 8192 pixels, otherwise the export speed will be significantly affected.

* The town export aims at a target count rather than a threshold, so a huge selection no longer comes back empty. It does still have to fetch every candidate node, so a very large box with the settlement floor set to `Any` will be slower.

### ✨ Key Features & Highlights

#### 1. 🍎 Friendly UI & Dual Theme System

* **Day / Night Mode Switch**: Starts in Light Mode by default. Effortlessly switch to Dark Mode via the top-right toolbar button.
* **Fixed Top-Right Control Bar**: Theme toggle and language selection are fixed at the top-right corner to ensure uncluttered map interactions.

#### 2. 📐 Fixed Aspect-Ratio Region Selection

* **Standard OpenTTD Tile Presets**: Supports official tile resolutions ranging from `64x64` up to `1048576`.
* **Locked Aspect Ratio Box**: The selection box automatically locks its proportions to match the selected map dimensions, preventing geographic distortion.
* **Shortcut Drag-to-Move (`Ctrl + Drag`)**: Hold the `Ctrl` key and drag the box to reposition the selection area seamlessly across the globe.

#### 3. 🏔️ DEM Elevation Processing & Terrain Smoothing

* **Bilinear Interpolation Sampling**: Smoothly resamples elevation tiles to prevent pixelation artifacts in scaled maps.
* **Precision Sea Level & Altitude Mapping**: Customize sea level cutoffs and maximum mountain elevations.
* **Multi-pass Box Filter Smoothing**: Flatten jagged mountain cliffs into gentler slopes that are ideal for building railway networks and roads.
* **Inland Lake Detection & Infill**: Built-in flood-fill algorithm automatically detects inland basins below sea level and fills them to prevent unnatural dry craters.
* **Custom Low-Elevation Offset**: Option to forcibly raise coastal or sub-sea-level land to specified target elevations.

#### 4. 🏙️ Real-World OSM Town Data Extraction

* **Live Overpass API Queries**: Fetches cities, towns and villages with their real-world names, coordinates, and populations within the box.
* **Multi-Mirror Endpoint Failover**: Automatically cascades across multiple Overpass API mirrors (`Overpass-API`, `Mail.ru`, `private.coffee`, `Kumi`), with a 30 s timeout per mirror.
* **Two-Phase Fetching**: Cities and towns are fetched first; villages and hamlets are fetched separately and degrade gracefully, so a single oversized query can no longer come back as a silent empty list.
* **Area-Density Sampling**: The selection is cut into equal-area cells and each populated cell contributes at most one town, so the exported density is uniform across the map instead of following real-world population.
* **Population-Driven Sizing**: Population only decides *which* town represents a cell and *how big* it becomes, so an urban agglomeration still stands out by the size of its town while the countryside keeps its small villages.
* **Smart `city` Flag**: `city: true` is set for OSM `place=city` nodes. Districts (`place=suburb` / `quarter` / `neighbourhood`) are not exported as separate towns.

#### 5. 📦 Multiple Export Formats & One-Click ZIP Packaging

* **Real-time Canvas Preview**: View the grayscale heightmap on an integrated preview canvas before downloading.
* **Multiple Town JSON Formats**:
* **OpenTTD / RoadTycoon Official Spec**: Normalized ratio coordinates (0.0–1.0) with swapped X/Y axes and `city` boolean flag.
* **Standard GameScript (GS)**: Absolute tile integer coordinates (`x`, `y`) with scaled population.
* **Minimal Ratio Mode**: Clean ratio coordinates with town names only.


* **One-Click ZIP Packaging**: Powered by JSZip, export both the PNG heightmap and Towns JSON together into a single downloadable ZIP file.

#### 6. 🌐 Intuitive Interaction & Multi-language Support (i18n & Tooltips)

* **Native i18n**: Seamless switching between **English (EN)**, **Chinese (ZH)**, and **Japanese (JA)**.
* **Hover Parameter Documentation**: Hovering over any parameter section displays an interactive tooltip explaining its usage based on OpenTTD specifications.
* **One-Click Reset Button**: Easily revert all sliders, checkboxes, and selections back to default baseline values.

---

### 🛠️ Tech Stack

* **Frontend**: HTML5, CSS3 (Custom Variables, Backdrop Blur Filters), Pure JavaScript (ES6+)
* **Map Engine**: [Leaflet.js](https://www.google.com/search?q=https://leafletjs.com/) (CartoDB Dark Tiles & OpenStreetMap Light Tiles)
* **DEM Data Source**: Amazon S3 Terrarium Elevation Tiles
* **GIS API**: OpenStreetMap Overpass API
* **Archiving**: [JSZip](https://www.google.com/search?q=https://stuk.github.io/jszip/)
* **Bunburya's script**: Bunburya's OpenTTD Heightmap Generator

### 🔰 Parameter Beginner's Guide

#### 1. Water Level Cutoff

* Determines the elevation altitude below which pixels are recognized as sea/water in the heightmap. `0m` (default), fine-tune based on shorelines.

#### 2. Max Elevation

*  Maps the highest real-world mountain peak in your selection to OpenTTD's maximum height steps (default is 2000m).

#### 3. Terrain Smoothing (Box Filtering)

*  Applies a smoothing algorithm to raw elevation tiles to soften jagged noise and sheer cliffs.(default is 2x).

#### 4. Force Lift Sea Level & Below Pixels

*  Forcibly elevates land areas that are at or below sea level by a specified offset, preventing them from being submerged. **Checked** when mapping low-lying coastal regions (e.g., Netherlands, Venice).

#### 5. Force Lift Low-Elevation Land

*  Ensures all land tiles near the coastline maintain a distinct minimum height step above water to avoid rendering artifacts. 

#### 6. Inland Lake Detection & Infill

* Uses a flood-fill algorithm to separate ocean coastal water from inland below-sea-level basins, automatically filling dry interior depressions.

(You could check them on https://github.com/bunburya/bother)

#### 7. Population Range & Curve

* The selected towns are mapped onto the range you set, instead of being multiplied by one constant. Real population is extremely long-tailed, so a plain multiplier either let a single metropolis swallow the map or left every village invisible - which is what made the old "population scale" impossible to tune.
* **Smallest village / Largest city (in-game pop)**: the two endpoints of the output range. Defaults `60` and `30000`.
* **Curve**:
* **Log compressed (default)**: `t = (ln p - ln p10) / (ln p90 - ln p10)`, then `out = exp(ln min + t * (ln max - ln min))`. Town order is preserved and the long tail is flattened.
* **Linear**: `out = p * max / p90`, clamped to the same range. Keeps real-world ratios, normalised so that the 90th percentile reaches the maximum.
* Missing `population` tags fall back to a small, honest default per place class (`city 50000`, `town 10000`, `village 1000`, `hamlet 200`). Ambiguous values such as `1000-2000` or `约 1.2 万` are ignored instead of being guessed at.

#### 8. Target Town Count & Settlement Floor

* **Target town count** (default `64`): the selection is cut into roughly this many equal-area cells, and each populated cell yields exactly one town - its most populous candidate. The count is something you choose, not a side effect of a threshold.
* **Settlement floor** (default `Any`): drops settlements below a real-world population. This is what stops deserts, oceans and unpopulated land from spawning a grid of hamlets - cells with nothing above the floor simply stay empty, and their quota is not redistributed to other cells.
* **Result**: town density is roughly uniform across the map, while urban agglomerations are distinguished by the *size* of their representative town rather than by crowding the map with extra towns.
* The status line above the export buttons reports what actually happened, e.g. `Target 64 - actual 51 - candidates 12842 - 13/64 cells have no settlement` and `Population 120 - 30000 (median 640) - 12 city/cities`.
