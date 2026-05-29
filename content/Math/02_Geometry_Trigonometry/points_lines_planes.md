---
title: "Points, Lines & Planes: The Geometry of Every Game World"
tags:
  - math
  - level/Lv1
  - category/geometry_trigonometry
---

# Points, Lines & Planes: The Geometry of Every Game World

> [!abstract] **The Concept in a Nutshell**
> Points, lines, and planes are the most basic building blocks of all game geometry. A **point** is a position with no size. A **line** extends infinitely in both directions, while a **line segment** has two endpoints. A **plane** is a flat, infinite surface in 3D space. Every mesh, every level boundary, every collision surface is built from these primitives. Understanding their mathematical representations unlocks everything from raycasting to level design.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Building a Laser Security System**
> In your stealth game "Shadow Protocol," the player must navigate through a room crisscrossed with laser beams. Each laser is a **line segment** from emitter point $A = (2, 1, 0)$ to receiver point $B = (8, 4, 0)$. The floor is a **plane** at $y = 0$. The walls are planes at $x = 0$, $x = 10$, $z = 0$, and $z = 6$.
>
> When the player (represented as a **point** at their position) moves, you need to check: does the player's movement path (a line segment) intersect any laser beam (another line segment)? If the player crouches, their height changes — now you're checking against the plane of the laser's height.
>
> The reflection of the laser off a mirror is computed using the mirror's surface **plane** and its **normal vector.** Every element in this scene is a point, line, or plane.

---

## The Blueprint (Formula & Structure)

### Points
A **point** represents a position — it has no size, area, or volume.

$$P = (x, y) \quad \text{(2D)}, \qquad P = (x, y, z) \quad \text{(3D)}$$

Points are stored as `Vector2` or `Vector3` in Unity. A point is *not* a direction — it's a location.

### Lines and Line Segments

**Parametric Line Equation:**
A line through point $P_0$ in direction $\vec{d}$:

$$P(t) = P_0 + t \cdot \vec{d}, \quad t \in (-\infty, +\infty)$$

**Line Segment:** Same equation but with bounded parameter:

$$P(t) = P_0 + t \cdot \vec{d}, \quad t \in [0, 1]$$

where $P_0$ is the start point and $P_0 + \vec{d}$ is the end point.

Equivalently, a segment between points $A$ and $B$:
$$P(t) = A + t(B - A) = (1-t)A + tB, \quad t \in [0, 1]$$

- $t = 0$: point $A$
- $t = 1$: point $B$
- $t = 0.5$: midpoint

### Rays
A **ray** starts at a point and extends infinitely in one direction:

$$P(t) = \text{origin} + t \cdot \vec{direction}, \quad t \in [0, +\infty)$$

This is exactly what `Physics.Raycast()` uses.

### Planes (3D)
A plane is defined by a **normal vector** $\hat{n}$ and a **distance** $d$ from the origin:

$$\hat{n} \cdot P = d$$

Or equivalently, a plane through point $P_0$ with normal $\hat{n}$:

$$\hat{n} \cdot (P - P_0) = 0$$

Expanded in 3D:

$$ax + by + cz = d$$

where $(a, b, c) = \hat{n}$ is the unit normal.

**Signed distance** from point $Q$ to the plane:

$$\text{dist} = \hat{n} \cdot Q - d$$

- Positive: $Q$ is on the side the normal points toward
- Negative: $Q$ is on the opposite side
- Zero: $Q$ is on the plane

### Polygons
A **polygon** is a closed shape formed by connecting line segments (edges). In games:
- **Triangle (3 vertices):** The fundamental rendering primitive — all meshes are triangles
- **Quad (4 vertices):** Two triangles sharing an edge — common for UI and terrain tiles
- **N-gon (N vertices):** Must be decomposed into triangles for rendering

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Laser Pointer and the Sheet of Paper**
> - **Point:** The dot a laser pointer makes on the wall — it has a position but no size.
> - **Line:** The laser beam itself — extends infinitely in both directions (if the pointer and wall were removed). A **segment** is the beam between the pointer and the wall. A **ray** is the beam from the pointer onward forever.
> - **Plane:** A perfectly flat, infinite sheet of paper. It has two sides (front and back), determined by which way the **normal** arrow sticks out. If you hold the paper horizontally, the normal points straight up.
>
> The parametric $t$ value is like a progress slider: $t = 0$ is "start," $t = 1$ is "end," and $t = 0.5$ is "halfway." Values outside $[0, 1]$ extend beyond the segment.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Working with points, lines, segments, and planes
using UnityEngine;

public class GeometryPrimitives : MonoBehaviour
{
    // --- Point on a Line Segment (lerp is just the parametric equation!) ---
    public static Vector3 PointOnSegment(Vector3 A, Vector3 B, float t)
    {
        // P(t) = A + t * (B - A) = lerp(A, B, t)
        return Vector3.Lerp(A, B, Mathf.Clamp01(t));
    }

    // --- Closest point on a line segment to an external point ---
    public static Vector3 ClosestPointOnSegment(Vector3 A, Vector3 B, Vector3 point)
    {
        Vector3 AB = B - A;
        float lengthSq = AB.sqrMagnitude;
        if (lengthSq < 0.0001f) return A; // degenerate segment

        // Project point onto the line, clamp to segment
        float t = Mathf.Clamp01(Vector3.Dot(point - A, AB) / lengthSq);
        return A + t * AB;
    }

    // --- Signed distance from a point to a plane ---
    public static float SignedDistanceToPlane(Vector3 point, Vector3 planeNormal, float planeD)
    {
        // Plane: n · P = d, where n is unit normal
        return Vector3.Dot(planeNormal, point) - planeD;
    }

    // --- Which side of a plane is a point on? ---
    public static string PlaneSide(Vector3 point, Plane plane)
    {
        float dist = plane.GetDistanceToPoint(point);
        if (dist > 0.001f) return "Front (normal side)";
        if (dist < -0.001f) return "Back (opposite side)";
        return "On the plane";
    }

    // --- 2D Line Segment Intersection (for laser security beams) ---
    public static bool SegmentsIntersect2D(
        Vector2 A1, Vector2 A2, Vector2 B1, Vector2 B2, out Vector2 intersection)
    {
        intersection = Vector2.zero;
        Vector2 d1 = A2 - A1;
        Vector2 d2 = B2 - B1;

        float cross = d1.x * d2.y - d1.y * d2.x;
        if (Mathf.Abs(cross) < 0.0001f) return false; // parallel

        Vector2 diff = B1 - A1;
        float t = (diff.x * d2.y - diff.y * d2.x) / cross;
        float u = (diff.x * d1.y - diff.y * d1.x) / cross;

        if (t >= 0f && t <= 1f && u >= 0f && u <= 1f)
        {
            intersection = A1 + t * d1;
            return true;
        }
        return false;
    }

    void Start()
    {
        // Midpoint of a segment
        Vector3 mid = PointOnSegment(Vector3.zero, new Vector3(10, 0, 0), 0.5f);
        Debug.Log($"Midpoint: {mid}"); // (5, 0, 0)

        // Signed distance to the ground plane (Y=0, normal=up)
        Vector3 birdPos = new Vector3(3, 5, 2);
        Plane ground = new Plane(Vector3.up, 0f);
        Debug.Log($"Bird is {ground.GetDistanceToPoint(birdPos)} above ground"); // 5

        // Laser intersection test
        Vector2 playerPath1 = new Vector2(1, 0);
        Vector2 playerPath2 = new Vector2(5, 3);
        Vector2 laserStart  = new Vector2(0, 2);
        Vector2 laserEnd    = new Vector2(6, 1);
        if (SegmentsIntersect2D(playerPath1, playerPath2, laserStart, laserEnd, out Vector2 hit))
            Debug.Log($"Player hits laser at {hit}!"); // Intersection point
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the parameter $t$ represent in the parametric line equation $P(t) = A + t(B - A)$? :: It represents the fractional progress along the line. $t=0$ gives point $A$, $t=1$ gives point $B$, $t=0.5$ gives the midpoint. Values outside $[0,1]$ extend beyond the segment.
- How is a ray different from a line and a line segment? :: A **line** extends infinitely in both directions ($t \in (-\infty, +\infty)$). A **ray** starts at a point and extends infinitely in one direction ($t \in [0, +\infty)$). A **segment** has two endpoints ($t \in [0, 1]$).
- What is the implicit equation of a plane? :: $\hat{n} \cdot P = d$, where $\hat{n}$ is the unit normal vector and $d$ is the signed distance from the origin to the plane.
- How do you determine which side of a plane a point is on? :: Compute the signed distance $\hat{n} \cdot Q - d$. Positive means the point is on the normal's side, negative means the opposite side, zero means it's on the plane.
- Why are all game meshes made of triangles? :: Triangles are always planar (three points always define a plane), always convex, and have well-defined normals. This makes rasterization, collision detection, and interpolation straightforward. Quads and n-gons must be decomposed into triangles.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Confusing a **point** (position) with a **vector** (direction). Both are stored as `Vector3`, but `(3, 5, 0)` as a position means "the location (3,5,0)" while as a direction it means "3 units right, 5 units up."
> - **The Fix:** Use comments and naming conventions — `enemyPos` vs `moveDir`. In homogeneous coordinates, points have $w=1$ and vectors have $w=0$, which makes the distinction explicit.
> - **Why:** Adding two positions makes no geometric sense, but adding two vectors does. Subtracting two positions gives a vector (the displacement between them). The math is the same, but the meaning is different.

---

## Related Topics
- [[Math/07_Geometric_Primitives/parametric_lines_rays|Parametric Lines & Rays]]
- [[Math/07_Geometric_Primitives/planes_implicit|Planes & Half-Spaces]]
- [[Math/07_Geometric_Primitives/triangles_meshes|Triangles & Meshes]]
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
