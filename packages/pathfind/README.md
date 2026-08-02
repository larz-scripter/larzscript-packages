# lz-pathfind

Real A* search over a 2D grid: an open set ordered by f = g + h, a closed
set, path reconstruction via a came-from map - not a maze-solver toy. Built
on this stack's own [`heap`](../heap) for the open set's priority queue.
4-directional by default (Manhattan heuristic), or 8-directional with
diagonals (Euclidean heuristic) - pass your own heuristic function for
anything else. Install: `larzscript pkg install pathfind`.

```
import "pathfind" as pf
let g = pf.grid_new(5, 5)
pf.grid_block(g, 1, 1); pf.grid_block(g, 2, 1); pf.grid_block(g, 3, 1)
let path = pf.astar(g, {"x": 0, "y": 0}, {"x": 4, "y": 4})
print(len(path))          # steps from start to goal, inclusive
for p in path { print(p["x"], p["y"]) }
```

See it live, including an unreachable-goal case, at
[larzos.com/stack/larzpathfind/](https://larzos.com/stack/larzpathfind/).
