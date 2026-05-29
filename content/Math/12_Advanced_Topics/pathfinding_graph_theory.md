---
title: "Pathfinding & Graph Theory: Finding the Way"
tags:
  - math
  - level/Lv3
  - category/advanced_topics
---

# Pathfinding & Graph Theory: Finding the Way

> [!abstract] **The Concept in a Nutshell**
> Graph theory provides the mathematical framework for representing connected systems — worlds made of nodes (locations) and edges (connections). Pathfinding algorithms like **BFS**, **Dijkstra's**, and **A\*** traverse these graphs to find optimal routes. In game dev, this powers NPC navigation, RTS unit movement, puzzle solving, and network routing. **Navigation meshes (navmeshes)** represent walkable areas as connected convex polygons, enabling smooth, natural-looking AI movement across complex 3D environments.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: An Open-World RPG — NPC Merchant Walks to the Market**
> The player hires an NPC merchant to deliver goods across the city. The merchant needs to navigate cobblestone streets, avoid a collapsed bridge (dynamically blocked edge), climb stairs to the upper district, and squeeze through a narrow alley. The city is represented as a **navmesh** — the walkable ground is divided into connected convex polygons. The **A\*** algorithm finds the optimal path from the merchant's house to the market square, using Euclidean distance as a heuristic. The **funnel algorithm** then smooths the path within the navmesh corridor, turning a zigzag sequence of polygon edges into a natural-looking walking path. When the bridge collapses (a graph edge is removed), the merchant dynamically recalculates a detour. All of this — graph construction, optimal search, path smoothing — is pure math.

---

## The Blueprint (Formula & Structure)

### Graph Fundamentals

A graph $G = (V, E)$ consists of:
- **Vertices (nodes)** $V$: locations, waypoints, navmesh polygons
- **Edges** $E \subseteq V \times V$: connections between vertices
- **Weights** $w(u, v)$: cost to traverse an edge (distance, time, danger level)

**Directed** graphs have one-way edges. **Undirected** graphs have bidirectional edges.

### Graph Representations

**Adjacency List** (memory-efficient for sparse graphs):
$$\text{adj}[u] = \{(v_1, w_1), (v_2, w_2), \ldots\}$$

**Adjacency Matrix** (fast lookup for dense graphs):
$$M[i][j] = \begin{cases} w(i, j) & \text{if edge exists} \\ \infty & \text{otherwise} \end{cases}$$

| Representation | Space | Edge Query | All Neighbors |
|---|---|---|---|
| Adjacency List | $O(V + E)$ | $O(\text{deg}(v))$ | $O(\text{deg}(v))$ |
| Adjacency Matrix | $O(V^2)$ | $O(1)$ | $O(V)$ |

### Breadth-First Search (BFS)

Explores nodes layer by layer. Finds **shortest path** in unweighted graphs.

**Complexity:** $O(V + E)$

**Use case:** Grid-based games where each tile costs 1 to traverse.

### Depth-First Search (DFS)

Explores as deep as possible before backtracking. Used for:
- Connectivity detection ("can I reach this area?")
- Topological sorting (dependency resolution)
- Maze generation

**Complexity:** $O(V + E)$

### Dijkstra's Algorithm

Finds shortest paths from a source to all nodes in a **weighted graph** (non-negative weights).

**Algorithm:**
1. Set $\text{dist}[s] = 0$, all others $= \infty$
2. Add source to priority queue
3. While queue is not empty:
   - Extract node $u$ with minimum $\text{dist}[u]$
   - For each neighbor $v$ of $u$:
     - $\text{alt} = \text{dist}[u] + w(u, v)$
     - If $\text{alt} < \text{dist}[v]$: update $\text{dist}[v] = \text{alt}$, set $\text{prev}[v] = u$

**Complexity:** $O((V + E) \log V)$ with a binary heap.

**Limitation:** Explores in all directions — doesn't "know" where the target is.

### A* Algorithm

An extension of Dijkstra's that uses a **heuristic** $h(n)$ to guide the search toward the goal:

$$f(n) = g(n) + h(n)$$

Where:
- $g(n)$ = actual cost from start to node $n$
- $h(n)$ = estimated cost from $n$ to goal (heuristic)
- $f(n)$ = estimated total cost through $n$

**Admissible heuristic:** $h(n)$ must **never overestimate** the true cost. Common choices:

| Heuristic | Formula | Use Case |
|---|---|---|
| Euclidean | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$ | Any-angle movement |
| Manhattan | $|x_2-x_1| + |y_2-y_1|$ | 4-directional grid |
| Chebyshev | $\max(|x_2-x_1|, |y_2-y_1|)$ | 8-directional grid |
| Octile | $(D_1 - D_2) \cdot d_{\text{card}} + D_2 \cdot d_{\text{diag}}$ | 8-dir, different costs |

Where $D_1 = \max(\Delta x, \Delta y)$, $D_2 = \min(\Delta x, \Delta y)$.

**Why A\* is efficient:** The heuristic prunes branches that lead away from the goal — like Dijkstra with a compass.

### Navigation Meshes (Navmeshes)

A navmesh divides walkable surfaces into **convex polygons**. Advantages over grid-based approaches:
- Fewer nodes (polygons vs. thousands of grid cells)
- Arbitrary polygon shapes fit level geometry precisely
- Agents can move freely within each polygon

**Pathfinding on navmeshes:**
1. Find which polygon contains the start and goal points
2. Run A* on the polygon adjacency graph (edges are shared polygon borders)
3. The result is a **corridor** — a sequence of connected polygons
4. Smooth the path within the corridor using the **funnel algorithm**

### The Funnel Algorithm (Path Smoothing)

Given a corridor of polygons, the funnel algorithm finds the **shortest path** that stays inside:
1. Start with a "funnel" — left and right portal edges from the start
2. For each portal (shared edge between adjacent polygons):
   - Narrow the funnel if the new edge tightens the cone
   - If the funnel collapses (left crosses right), a turn point is found
3. Output: a sequence of waypoints forming the shortest smooth path

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Map App on Your Phone**
> **Dijkstra** is like Google Maps expanding outward from your location in all directions equally — it finds the best route, but it checks lots of irrelevant roads behind you. **A\*** is Google Maps with a sense of direction — it knows roughly where the destination is, so it prioritizes roads heading that way while still guaranteeing optimality. A **navmesh** is like the pedestrian walking areas painted on a city map — instead of thinking about every square meter, the NPC thinks "I'm in the Plaza polygon, I need to get to the Market polygon" and only worries about which polygon connections to take. The **funnel algorithm** is "once I know which areas to walk through, find the straightest path that doesn't cut through walls."

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — A* pathfinding on a grid
using UnityEngine;
using System.Collections.Generic;

public class AStarPathfinder : MonoBehaviour
{
    public class Node
    {
        public int x, y;
        public float gCost, hCost;
        public float fCost => gCost + hCost;
        public bool walkable;
        public Node parent;

        public Node(int x, int y, bool walkable)
        {
            this.x = x; this.y = y; this.walkable = walkable;
            gCost = float.MaxValue;
        }
    }

    private Node[,] grid;
    private int width = 20, height = 20;

    void Start()
    {
        // Create grid (some cells are walls)
        grid = new Node[width, height];
        for (int x = 0; x < width; x++)
            for (int y = 0; y < height; y++)
                grid[x, y] = new Node(x, y, Random.value > 0.25f);

        // Find path from (0,0) to (19,19)
        List<Node> path = FindPath(grid[0, 0], grid[width - 1, height - 1]);
        if (path != null)
            Debug.Log($"Path found with {path.Count} nodes!");
        else
            Debug.Log("No path exists!");
    }

    List<Node> FindPath(Node start, Node goal)
    {
        var openSet = new SortedSet<Node>(Comparer<Node>.Create((a, b) =>
            a.fCost != b.fCost ? a.fCost.CompareTo(b.fCost) : a.GetHashCode().CompareTo(b.GetHashCode())));
        var closedSet = new HashSet<Node>();

        start.gCost = 0;
        start.hCost = Heuristic(start, goal);
        openSet.Add(start);

        while (openSet.Count > 0)
        {
            Node current = openSet.Min;
            openSet.Remove(current);

            if (current == goal) return ReconstructPath(goal);
            closedSet.Add(current);

            foreach (Node neighbor in GetNeighbors(current))
            {
                if (!neighbor.walkable || closedSet.Contains(neighbor)) continue;

                float tentativeG = current.gCost + Distance(current, neighbor);

                if (tentativeG < neighbor.gCost)
                {
                    neighbor.parent = current;
                    neighbor.gCost = tentativeG;
                    neighbor.hCost = Heuristic(neighbor, goal);

                    // SortedSet doesn't re-sort on value change, so remove and re-add
                    openSet.Remove(neighbor);
                    openSet.Add(neighbor);
                }
            }
        }
        return null; // no path
    }

    float Heuristic(Node a, Node b) =>
        Mathf.Sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y)); // Euclidean

    float Distance(Node a, Node b) =>
        (a.x != b.x && a.y != b.y) ? 1.414f : 1f; // diagonal vs cardinal

    List<Node> GetNeighbors(Node node)
    {
        var neighbors = new List<Node>();
        for (int dx = -1; dx <= 1; dx++)
            for (int dy = -1; dy <= 1; dy++)
            {
                if (dx == 0 && dy == 0) continue;
                int nx = node.x + dx, ny = node.y + dy;
                if (nx >= 0 && nx < width && ny >= 0 && ny < height)
                    neighbors.Add(grid[nx, ny]);
            }
        return neighbors;
    }

    List<Node> ReconstructPath(Node goal)
    {
        var path = new List<Node>();
        Node current = goal;
        while (current != null) { path.Add(current); current = current.parent; }
        path.Reverse();
        return path;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the A* evaluation function and what do its components mean? :: $f(n) = g(n) + h(n)$. $g(n)$ = actual cost from start to $n$. $h(n)$ = heuristic estimated cost from $n$ to goal. A* always expands the node with the lowest $f(n)$.
- What does "admissible heuristic" mean and why is it important? :: A heuristic is admissible if it never overestimates the true cost to the goal. This guarantees A* finds the optimal path. If $h$ overestimates, A* might find a suboptimal path by ignoring the true shortest route.
- When would you use Dijkstra's over A*? :: When there's **no specific goal** (finding shortest paths to ALL reachable nodes), or when no good heuristic exists. A* requires a goal and a heuristic. Dijkstra's is A* with $h(n) = 0$.
- What is a navigation mesh and why is it preferred over grid-based pathfinding? :: A navmesh represents walkable areas as convex polygons. It requires far fewer nodes than a grid (hundreds vs. thousands), conforms precisely to level geometry, and allows free movement within polygons — producing more natural paths.
- What does the funnel algorithm do? :: Given a corridor of connected navmesh polygons from A*, the funnel algorithm finds the shortest geometric path within that corridor. It produces smooth, natural-looking paths with minimal waypoints instead of zigzagging through polygon centers.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using Euclidean heuristic on a 4-directional grid, or Manhattan heuristic for free-movement — both produce suboptimal search behavior.
> - **The Fix:** Match the heuristic to the movement model: Manhattan for 4-dir, Chebyshev/Octile for 8-dir, Euclidean for free movement or navmeshes.
> - **Why:** An overestimating heuristic (Manhattan on 8-dir grid) makes A* inadmissible — it may find longer paths. An underestimating but loose heuristic (Euclidean on 4-dir grid) is admissible but slow — it explores too many nodes.

> [!danger] **Watch Out!**
> - **The Mistake:** Recalculating the entire path every frame for moving targets, causing performance spikes.
> - **The Fix:** Use incremental replanning algorithms (D* Lite) or recalculate only when the target moves beyond a threshold. Cache and reuse partial paths.
> - **Why:** A* on a large graph can take milliseconds — running it every frame for multiple agents causes frame rate drops. Smart replanning only updates the affected portion of the path.

---

## Related Topics
- [[Math/01_Foundations/boolean_logic_sets|Boolean Logic & Sets]]
- [[Math/10_Physics_Math/collision_detection|Collision Detection]]
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
