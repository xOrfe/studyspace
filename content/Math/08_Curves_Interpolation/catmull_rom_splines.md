---
title: "Catmull-Rom Splines — Smooth Paths Through Every Point"
tags:
  - math
  - level/Lv3
  - category/curves_interpolation
---

# Catmull-Rom Splines: Smooth Paths Through Every Point

> [!abstract] **The Concept in a Nutshell**
> A Catmull-Rom spline is a type of cubic interpolating spline that passes **through** every control point — unlike Bézier curves, which only touch their endpoints. Given a sequence of points, the spline automatically generates smooth curves between them, using neighboring points to calculate tangents. This makes it the go-to tool for camera rails, NPC patrol routes, and any path defined by waypoints.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Cinematic Camera Rail in a Cutscene**
> Your RPG has a dramatic cutscene: the camera swoops from behind a castle tower, glides over the courtyard, dips toward the hero, then rises to reveal the dragon on the distant mountain. The cinematic designer places 6 waypoints in the editor. A Catmull-Rom spline threads through *every single one*, producing a buttery-smooth camera path with zero manual tangent tweaking. The director moves a waypoint, and the curve updates globally while staying smooth. You evaluate the spline over time and feed the position + tangent to the camera's transform and look-at direction.

---

## The Blueprint (Formula & Structure)

### The Catmull-Rom Formula

Given four sequential control points $\mathbf{P}_{i-1}, \mathbf{P}_i, \mathbf{P}_{i+1}, \mathbf{P}_{i+2}$ and parameter $t \in [0, 1]$ (interpolating between $\mathbf{P}_i$ and $\mathbf{P}_{i+1}$):

$$\mathbf{C}(t) = 0.5 \begin{bmatrix} 1 & t & t^2 & t^3 \end{bmatrix} \begin{bmatrix} 0 & 2 & 0 & 0 \\ -1 & 0 & 1 & 0 \\ 2 & -5 & 4 & -1 \\ -1 & 3 & -3 & 1 \end{bmatrix} \begin{bmatrix} \mathbf{P}_{i-1} \\ \mathbf{P}_i \\ \mathbf{P}_{i+1} \\ \mathbf{P}_{i+2} \end{bmatrix}$$

Expanded:

$$\mathbf{C}(t) = 0.5 \big[ (2\mathbf{P}_i) + (-\mathbf{P}_{i-1} + \mathbf{P}_{i+1})t + (2\mathbf{P}_{i-1} - 5\mathbf{P}_i + 4\mathbf{P}_{i+1} - \mathbf{P}_{i+2})t^2 + (-\mathbf{P}_{i-1} + 3\mathbf{P}_i - 3\mathbf{P}_{i+1} + \mathbf{P}_{i+2})t^3 \big]$$

### Key Properties

| Property | Detail |
|---|---|
| **Interpolating** | The curve passes through every control point |
| **$C^1$ continuous** | Tangent direction and magnitude are smooth at each point |
| **Local control** | Moving one point only affects the two adjacent curve segments |
| **Requires 4 points** | Each segment needs 2 neighbors: one before, one after |

### Tangent Calculation

The tangent at point $\mathbf{P}_i$ is determined by its two neighbors:

$$\mathbf{T}_i = \frac{\mathbf{P}_{i+1} - \mathbf{P}_{i-1}}{2}$$

This is the "chord" tangent — it points from the previous point toward the next, scaled by half. This automatic tangent is what makes Catmull-Rom splines so easy to use.

### Tension Parameter

The standard Catmull-Rom uses a tension of $\tau = 0.5$. A generalized **Cardinal spline** parameterizes this:

$$\mathbf{T}_i = (1 - \tau) \cdot \frac{\mathbf{P}_{i+1} - \mathbf{P}_{i-1}}{2}$$

| Tension $\tau$ | Effect |
|---|---|
| $0$ | Loose — large, sweeping curves (Catmull-Rom default) |
| $0.5$ | Moderate tightening |
| $1$ | Maximum tightness — approaches straight line segments |

### Converting a Catmull-Rom Segment to Cubic Bézier

Given the Catmull-Rom tangent $\mathbf{T}_i$ at each point, the equivalent cubic Bézier control points are:

$$\mathbf{B}_0 = \mathbf{P}_i, \quad \mathbf{B}_1 = \mathbf{P}_i + \frac{\mathbf{T}_i}{3}, \quad \mathbf{B}_2 = \mathbf{P}_{i+1} - \frac{\mathbf{T}_{i+1}}{3}, \quad \mathbf{B}_3 = \mathbf{P}_{i+1}$$

This lets you leverage existing Bézier rendering or evaluation code.

### Open vs Closed Splines

- **Open spline**: The first and last segments need phantom points. Common strategies:
  - Duplicate the first/last point: $P_{-1} = P_0$, $P_{n} = P_{n-1}$
  - Reflect: $P_{-1} = 2P_0 - P_1$
- **Closed (looping) spline**: The points wrap around — $P_{-1} = P_{n-1}$ and $P_{n} = P_0$, $P_{n+1} = P_1$.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Threading Beads on a Flexible Wire**
> Imagine your control points are beads fixed on a table. A Catmull-Rom spline is like threading a flexible wire through every bead's hole. The wire can't skip any bead — it must pass through each one. Between any two beads, the wire curves smoothly because it "looks ahead" at the next bead and "remembers" the previous one to decide which direction to bend. Moving a single bead reshapes the wire locally around it but doesn't disturb distant sections.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Catmull-Rom spline evaluation for camera rails
using UnityEngine;
using System.Collections.Generic;

public class CatmullRomSpline : MonoBehaviour
{
    public List<Transform> waypoints;
    public bool isLooping = false;
    public int segmentResolution = 20;

    /// <summary>
    /// Evaluate the Catmull-Rom spline at a global parameter t ∈ [0, numSegments].
    /// </summary>
    public Vector3 EvaluateAt(float globalT)
    {
        int count = waypoints.Count;
        if (count < 2) return waypoints[0].position;

        int numSegments = isLooping ? count : count - 1;
        int seg = Mathf.FloorToInt(globalT);
        seg = Mathf.Clamp(seg, 0, numSegments - 1);
        float localT = globalT - seg;

        Vector3 p0 = GetPoint(seg - 1);
        Vector3 p1 = GetPoint(seg);
        Vector3 p2 = GetPoint(seg + 1);
        Vector3 p3 = GetPoint(seg + 2);

        return CatmullRom(p0, p1, p2, p3, localT);
    }

    /// <summary>
    /// Core Catmull-Rom interpolation for a single segment.
    /// </summary>
    public static Vector3 CatmullRom(Vector3 p0, Vector3 p1, Vector3 p2, Vector3 p3, float t)
    {
        float t2 = t * t;
        float t3 = t2 * t;

        return 0.5f * (
            (2f * p1) +
            (-p0 + p2) * t +
            (2f * p0 - 5f * p1 + 4f * p2 - p3) * t2 +
            (-p0 + 3f * p1 - 3f * p2 + p3) * t3
        );
    }

    /// <summary>
    /// Tangent (first derivative) for orientation along the spline.
    /// </summary>
    public static Vector3 CatmullRomTangent(Vector3 p0, Vector3 p1, Vector3 p2, Vector3 p3, float t)
    {
        float t2 = t * t;
        return 0.5f * (
            (-p0 + p2) +
            (4f * p0 - 10f * p1 + 8f * p2 - 2f * p3) * t +
            (-3f * p0 + 9f * p1 - 9f * p2 + 3f * p3) * t2
        );
    }

    /// <summary>
    /// Get point with proper wrapping for closed splines or clamping for open.
    /// </summary>
    private Vector3 GetPoint(int index)
    {
        int count = waypoints.Count;
        if (isLooping)
            index = ((index % count) + count) % count;
        else
            index = Mathf.Clamp(index, 0, count - 1);
        return waypoints[index].position;
    }

    // --- Gizmo visualization ---
    void OnDrawGizmos()
    {
        if (waypoints == null || waypoints.Count < 2) return;

        int numSegments = isLooping ? waypoints.Count : waypoints.Count - 1;
        Gizmos.color = Color.cyan;

        for (int seg = 0; seg < numSegments; seg++)
        {
            Vector3 prev = EvaluateAt(seg);
            for (int i = 1; i <= segmentResolution; i++)
            {
                float t = seg + i / (float)segmentResolution;
                Vector3 curr = EvaluateAt(t);
                Gizmos.DrawLine(prev, curr);
                prev = curr;
            }
        }

        Gizmos.color = Color.yellow;
        foreach (var wp in waypoints)
            Gizmos.DrawSphere(wp.position, 0.15f);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How does Catmull-Rom differ from Bézier in terms of interpolation? :: Catmull-Rom passes *through* every control point; Bézier only touches its first and last points.
- What determines the tangent at a Catmull-Rom control point? :: The tangent at $P_i$ is $\frac{P_{i+1} - P_{i-1}}{2}$ — half the vector from the previous point to the next.
- How many control points does one Catmull-Rom segment need? :: 4 points: the segment endpoints plus one neighbor on each side.
- What does the tension parameter $\tau$ control? :: $\tau = 0$ gives the loosest curves; $\tau = 1$ approaches straight lines between points.
- How do you handle endpoints in an open Catmull-Rom spline? :: Add phantom points — either duplicate the first/last point or reflect across them.
- How do you convert a Catmull-Rom segment to a cubic Bézier? :: Set $B_0 = P_i$, $B_1 = P_i + T_i/3$, $B_2 = P_{i+1} - T_{i+1}/3$, $B_3 = P_{i+1}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to handle the first and last segments of an open spline, resulting in index-out-of-bounds errors or weird curves at the start/end.
> - **The Fix:** Clamp or duplicate the boundary indices so that $P_{-1}$ and $P_{n}$ resolve to valid points.
> - **Why:** Each segment needs four points. The first segment lacks $P_{-1}$ and the last lacks $P_{n+1}$ unless you explicitly provide them.

> [!danger] **Watch Out!**
> - **The Mistake:** Using Catmull-Rom for paths with sharp corners, expecting the corner to be preserved.
> - **The Fix:** Insert duplicate or near-duplicate control points at the corner to tighten it, or break the spline into separate segments.
> - **Why:** Catmull-Rom guarantees $C^1$ continuity — it always rounds off corners. The tangent averaging prevents true discontinuities.

---

## Related Topics
- [[Math/08_Curves_Interpolation/bezier_curves|Bézier Curves]]
- [[Math/08_Curves_Interpolation/curve_sampling_speed|Curve Sampling & Constant Speed]]
- [[Math/08_Curves_Interpolation/lerp_fundamentals|Lerp Fundamentals]]
