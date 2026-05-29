---
title: "Distance & the Pythagorean Theorem: Proximity and Optimization"
tags:
  - math
  - level/Lv1
  - category/geometry_trigonometry
---

# Distance & the Pythagorean Theorem: Proximity and Optimization

> [!abstract] **The Concept in a Nutshell**
> Calculating distance is one of the most frequent operations in game development. Whether checking if a player is in range of an explosion, triggering an enemy's alert state, or rendering level of detail (LOD) models, we rely on the Pythagorean theorem. However, computing absolute distance requires a square root ($\sqrt{x}$), which is computationally expensive. Understanding the relationship between actual distance and **squared distance** is one of the most basic and powerful performance optimizations in game programming.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Proximity Triggers and the Cost of Square Roots**
> You are building a horde shooter with $1{,}000$ active enemies. Every frame, each enemy needs to check if the player is within their attack range ($R = 5$ meters).
>
> A naive distance calculation uses the Euclidean distance formula:
> $$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$
>
> Running a square root $1{,}000$ times per frame on the main thread is a waste of CPU cycles. Instead, we can compare the **squared distance** to the **squared range** ($R^2 = 25$). 
> $$(x_2 - x_1)^2 + (y_2 - y_1)^2 < 25$$
> This yields the exact same logical result while completely avoiding the costly square root operation.

---

## The Blueprint (Formula & Structure)

### 2D Pythagorean Theorem & Euclidean Distance
For any right-angled triangle, the square of the hypotenuse ($c$) is equal to the sum of the squares of the other two sides ($a$ and $b$):
$$a^2 + b^2 = c^2 \implies c = \sqrt{a^2 + b^2}$$

To find the distance between two 2D points $P_1(x_1, y_1)$ and $P_2(x_2, y_2)$:
$$d = \sqrt{\Delta x^2 + \Delta y^2} = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

### 3D Euclidean Distance
By applying the theorem twice, we extend it to 3D space. The distance between $P_1(x_1, y_1, z_1)$ and $P_2(x_2, y_2, z_2)$ is:
$$d = \sqrt{\Delta x^2 + \Delta y^2 + \Delta z^2} = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2 + (z_2 - z_1)^2}$$

### Manhattan Distance (L1 Distance)
Instead of a straight line, Manhattan distance measures distance along axes (like navigating city grid blocks):
$$d_{\text{Manhattan}} = |\Delta x| + |\Delta y| + |\Delta z|$$
- **Use Case:** Grid-based games (chess, roguelikes, pathfinding heuristics like A*).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Bubble Comparison**
> Imagine an enemy checking if the player is inside a spherical "alert bubble." 
> - Calculating $d$ calculates the exact radius of the bubble.
> - Calculating $d^2$ expands the bubble's threshold to $R^2$. 
> 
> Comparing the squared values is mathematically identical to comparing their square roots, but it is much faster.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Naive vs. Optimized distance checks
using UnityEngine;

public class DistanceCheckDemo : MonoBehaviour
{
    public Transform player;
    public float attackRange = 5f;

    void Update()
    {
        // ❌ NAIVE WAY: Calculates actual distance (uses sqrt under the hood)
        float distance = Vector3.Distance(transform.position, player.position);
        if (distance < attackRange)
        {
            // Player is in range
        }

        //  OPTIMIZED WAY: Uses squared distance (no sqrt)
        Vector3 offset = player.position - transform.position;
        float sqrDistance = offset.sqrMagnitude; // dx^2 + dy^2 + dz^2
        float sqrRange = attackRange * attackRange;

        if (sqrDistance < sqrRange)
        {
            // Player is in range (much faster calculation!)
        }
    }

    // Manhattan distance implementation for grid-based pathfinding
    public int GetManhattanDistance(Vector2Int a, Vector2Int b)
    {
        return Mathf.Abs(a.x - b.x) + Mathf.Abs(a.y - b.y);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Why is checking squared distance faster than absolute distance? :: It avoids the computationally expensive square root ($\sqrt{}$) operation, replacing it with simple multiplications and additions.
- What is the formula for 3D Euclidean distance? :: $d = \sqrt{\Delta x^2 + \Delta y^2 + \Delta z^2}$.
- When should you use Manhattan distance instead of Euclidean distance? :: In grid-locked movement games (e.g. turn-based games, roguelikes, chess) or as a fast heuristic in pathfinding algorithms like A*.
- What is the squared distance check formula for a range $R$? :: $\Delta x^2 + \Delta y^2 + \Delta z^2 < R^2$.
- Does Unity have a built-in property for squared magnitude of a vector? :: Yes, `Vector3.sqrMagnitude`.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `Vector3.Distance` in tight loops or Update methods for simple range checks.
> - **The Fix:** Cache the squared range (e.g., `range * range`) and compare it to the vector's `sqrMagnitude`.
> - **Why:** A square root operation can be dozens of times slower than simple arithmetic. Multiplying that over hundreds of objects every frame causes unnecessary CPU bottleneck.

---

## Related Topics
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
- [[Math/03_Vectors/magnitude_normalization|Magnitude & Normalization]]
- [[Math/12_Advanced_Topics/pathfinding_graph_theory|Pathfinding & Graph Theory]]
