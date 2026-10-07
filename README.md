# Simple road network generator for games

**[Live demo](https://tamimasados.github.io/simple-road-network-generator-for-games/)**

A procedural generator for settlements and the roads between them, made for game worlds. It places points in three layers (major, medium and minor), connects them into a road network that follows the terrain, cleans the network up, and exports it as SVG or JSON for use in a game or editor.

Everything runs in the browser from a single HTML file: no install, no server, no dependencies.

## Features

- **Three layers.** Major points get highways, medium points get roads, minor points get trails. Each layer has its own seed and settings, and a later layer only extends the network built before it. Changing layer 3 never moves anything in layers 1 or 2.
- **Connected by construction.** Roads come from a Delaunay triangulation of the points: a minimum spanning tree guarantees every point is reachable, and some of the remaining edges are kept as extra links.
- **Terrain-aware routing.** Roads are routed with A* over a heightmap, with a hard maximum grade, so they wind and switch back on steep slopes instead of climbing straight up. Points prefer suitable heights and gentle slopes. The heightmap is procedural noise or any grayscale image you load.
- **Network cleanup.**
  - Parallel duplicate roads are merged into one.
  - Crossings become real junctions.
  - Clusters of nearby junctions are merged, with a cap on how many roads meet at one junction.
  - Pointless detours, dead-end stubs and leftover seams are removed.
- **Empty zones.** Finds the large areas away from all roads and points, e.g. for forests.
- **Export.**
  - SVG of the map.
  - JSON with points, roads (as evenly spaced points), road class and role, and empty zones, in a grid coordinate space you choose.
- **Deterministic.** The same seeds and settings always give the same map. Terrain routing runs in parallel on Web Workers.

## Usage

Open the [live demo](https://tamimasados.github.io/simple-road-network-generator-for-games/), or download `index.html` and open it in a browser.

1. Set the number of points per layer and press **Rebuild network**. Use the 🎲 buttons for new seeds.
2. Enable **Terrain** for terrain-following roads. Pick procedural noise or load a grayscale heightmap image, then set the map size in km and the elevation range.
3. Tune the per-layer terrain limits (max grade, suitable heights) and the post-processing distances. Distances are in meters, scaled by the map size.
4. Export with **SVG** or **JSON**.

## JSON export

All coordinates are in grid cells. *Canvas (cells)* sets how many cells span the map, and *Points per cell* sets how densely the road points are spaced.

| Field | Content |
|---|---|
| `seeds` | Seeds of the three layers |
| `grid` | Grid size, point density and the coordinate unit |
| `legend` | Meaning of the numeric codes below |
| `nodes` | `{id, x, y, layer, roadKinds}`. `layer`: 1 major, 2 medium, 3 minor, 4 junction |
| `roads` | `{from, to, kind, role, points}`. `kind`: 1 highway, 2 road, 3 trail. `role`: 1 backbone, 2 extra link |
| `emptyZones` | `{id, points, areaCells, centroid}` |

Road end points match their nodes' coordinates exactly, so roads can be matched to points by value as well as by id.
