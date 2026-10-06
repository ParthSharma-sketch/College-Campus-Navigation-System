# Campus Navigation System — Acropolis College

> Every path on campus, one tap away.

A zero-dependency web app that finds the shortest walking route between places on the Acropolis College campus. It runs entirely in the browser, with no API keys and no build step.

## Run it
- **Locally:** open `index.html` in any browser.
- **GitHub Pages:** push the repo, then go to Settings → Pages → Deploy from branch `main` / root. The site will be live at `https://<your-username>.github.io/<repo-name>/`.

## Features
- Interactive SVG campus map: roads, footpaths, gardens, round trees, pine forests, buildings, labs, washrooms
- Natural-language search ("Where is the AI Lab?", "DS Lab from Main Gate", "nearest washroom")
- Highlighted route with distance, walking time, places passed and written directions
- "I'm here" (tap the map or use the dropdown), recent searches, demo chips, destination details
- Accessible mode that avoids stairs and uses the lift

## Architecture
Everything lives in `index.html`, in four layers:
1. **Data:** `D` (nodes: id, name, x, y, type, description) and `E` (weighted edges). `AL` holds search aliases.
2. **Algorithm:** `dijkstra()` with a binary min-heap, plus `pathTo()` and `steps()` for directions.
3. **NLP:** `parse()` and `resolve()` map text to source and destination nodes.
4. **UI:** `draw()` renders the map from the same node data; `render()` draws the route and result card.

## The graph
- About 50 nodes: road junctions, building entrances, labs, washrooms, gardens
- Undirected edges; weight = Euclidean map distance × 0.5 m/px (+14 m for the lift)
- Edges can be flagged as stairs (`'s'`) or lift (`'l'`)
- Main road runs west to east: Gate → Management → Canteen → Bus junction → B.Tech blocks → East. Side paths link everything else.

## Dijkstra
Start with distance 0 at the source and infinity elsewhere. Repeatedly pop the closest unvisited node from the min-heap, then relax its edges. Stop when the heap is empty and rebuild the path from the `prev` map. Edge weights are non-negative, so the result is optimal and deterministic. In accessible mode, stair edges are skipped during relaxation.

## NLP layer (local, rule-based)
1. Detect `from X to Y`, `X from Y`, `X to Y`; otherwise treat the whole query as the destination.
2. Remove stop-words, then score each place's aliases by token overlap, with typo tolerance (Levenshtein ≤ 1).
3. If several places match equally (e.g. "washroom") or the query says "nearest", pick the one with the smallest Dijkstra distance.

## Complexity
- Dijkstra: O((V + E) log V). For V ≈ 50, E ≈ 50 this is effectively instant.
- Search: O(P × A × T) for P places, A aliases, T query tokens.
- Rendering: O(V + E).

## Judge Q&A
- **Why Dijkstra and not an AI model?** Routing must be exact and repeatable. NLP only picks the nodes; Dijkstra finds the path.
- **Why no API?** It works offline, costs nothing and has no keys to leak.
- **How do I add a place?** Add a row to `D`, edges to `E`, aliases to `AL`, and a rectangle to `R` if it is a building.
- **Why not A*?** The graph is tiny, so A* would give no visible gain. The coordinates are already stored if you want to add it.
- **How accurate are the distances?** They are scaled from the map drawing. Replace the weights with surveyed values for real use.
- **How does accessible mode work?** Stair edges are excluded; the AI Lab is reached through the lift instead.
