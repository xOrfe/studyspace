---
title: "Bézier Curves — Sculpting Paths with Control Points"
tags:
  - math
  - level/Lv3
  - category/curves_interpolation
---

# Bézier Curves: Sculpting Paths with Control Points

> [!abstract] **The Concept in a Nutshell**
> A Bézier curve is a parametric curve defined by a set of control points. The curve is *pulled toward* each control point without necessarily passing through them (except the endpoints). Quadratic Béziers use 3 points, cubic Béziers use 4. They are the backbone of vector graphics, font rendering, animation paths, and level-design tools in games.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Homing Missile Trajectory**
> In your twin-stick shooter, you fire a homing missile at a drone. The missile shouldn't travel in a straight line — it needs a dramatic, curving arc. You define a cubic Bézier where $P_0$ is the turret position, $P_3$ is the drone, and $P_1, P_2$ are offset upward and sideways to create a sweeping flight path. Each frame, you advance $t$ and evaluate the curve: the missile swoops upward, banks right, then dives onto the target. The level designer tweaks $P_1$ and $P_2$ in the editor to adjust how wide and aggressive the arc looks — no code changes needed.

---

## The Blueprint (Formula & Structure)

### Quadratic Bézier (3 Control Points)

$$\mathbf{B}(t) = (1-t)^2 \mathbf{P}_0 + 2(1-t)t\, \mathbf{P}_1 + t^2 \mathbf{P}_2, \quad t \in [0,1]$$

- $\mathbf{P}_0$: start point (on-curve)
- $\mathbf{P}_1$: control point (off-curve — the curve is *pulled toward* it)
- $\mathbf{P}_2$: end point (on-curve)

### Cubic Bézier (4 Control Points)

$$\mathbf{B}(t) = (1-t)^3 \mathbf{P}_0 + 3(1-t)^2 t\, \mathbf{P}_1 + 3(1-t)t^2\, \mathbf{P}_2 + t^3 \mathbf{P}_3$$

This is the most commonly used form — CSS transitions, SVG paths, font outlines, and Unity's `AnimationCurve` all rely on cubic Béziers.

### Bernstein Polynomial Form

The general degree-$n$ Bézier curve can be written using Bernstein basis polynomials:

$$\mathbf{B}(t) = \sum_{i=0}^{n} \binom{n}{i} (1-t)^{n-i} t^i \, \mathbf{P}_i$$

For cubic ($n=3$), the Bernstein coefficients are:

$$B_{0,3} = (1-t)^3, \quad B_{1,3} = 3(1-t)^2 t, \quad B_{2,3} = 3(1-t)t^2, \quad B_{3,3} = t^3$$

These four coefficients always sum to 1, ensuring the curve stays within the **convex hull** of its control points.

### De Casteljau's Algorithm

An elegant, numerically stable recursive method to evaluate a Bézier at parameter $t$:

1. **Given** control points $P_0, P_1, \ldots, P_n$
2. **Lerp** each adjacent pair: $P_i^{(1)} = \text{lerp}(P_i, P_{i+1}, t)$ — this gives $n$ points
3. **Repeat** on the new set until one point remains — that's $\mathbf{B}(t)$

For a cubic curve, this means 3 lerps → 2 lerps → 1 lerp = the point on the curve.

### Tangent at Endpoints

$$\mathbf{B}'(0) = n(\mathbf{P}_1 - \mathbf{P}_0), \quad \mathbf{B}'(1) = n(\mathbf{P}_n - \mathbf{P}_{n-1})$$

The tangent at the start points from $P_0$ toward $P_1$; the tangent at the end points from $P_{n-1}$ toward $P_n$. This is crucial for **continuity** when chaining curves.

### Continuity Between Segments

| Type | Meaning | Requirement |
|---|---|---|
| $C^0$ | Curves connect | Last point of curve A = first point of curve B |
| $C^1$ | Smooth connection | $C^0$ + tangent vectors are equal at the joint |
| $G^1$ | Visually smooth | $C^0$ + tangent *directions* match (magnitudes may differ) |
| $C^2$ | Curvature-smooth | $C^1$ + second derivatives match |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Puppet Strings**
> Imagine $P_0$ and $P_3$ are nails holding a flexible wire in place. $P_1$ and $P_2$ are magnets pulling the wire toward them — the wire bends in their direction but doesn't touch them. Stronger magnets (control points farther from the baseline) create more dramatic bends. Moving a magnet reshapes the curve locally without affecting the nailed endpoints. De Casteljau's algorithm is like recursively finding the midpoint of midpoints — zooming in on exactly where the wire sits at a given position along its length.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Cubic Bézier evaluation, tangent, and path drawing
using UnityEngine;

public class CubicBezier : MonoBehaviour
{
    public Transform p0, p1, p2, p3;
    public int segments = 40;
    public LineRenderer lineRenderer;

    void Update()
    {
        DrawCurve();
    }

    // --- Evaluate cubic Bézier at t using De Casteljau's ---
    public static Vector3 Evaluate(Vector3 a, Vector3 b, Vector3 c, Vector3 d, float t)
    {
        // Level 1
        Vector3 ab  = Vector3.Lerp(a, b, t);
        Vector3 bc  = Vector3.Lerp(b, c, t);
        Vector3 cd  = Vector3.Lerp(c, d, t);
        // Level 2
        Vector3 abc = Vector3.Lerp(ab, bc, t);
        Vector3 bcd = Vector3.Lerp(bc, cd, t);
        // Level 3
        return Vector3.Lerp(abc, bcd, t);
    }

    // --- Tangent (first derivative) ---
    public static Vector3 Tangent(Vector3 a, Vector3 b, Vector3 c, Vector3 d, float t)
    {
        float u = 1f - t;
        return 3f * u * u * (b - a) +
               6f * u * t * (c - b) +
               3f * t * t * (d - c);
    }

    // --- Draw the curve using a LineRenderer ---
    void DrawCurve()
    {
        lineRenderer.positionCount = segments + 1;
        for (int i = 0; i <= segments; i++)
        {
            float t = i / (float)segments;
            lineRenderer.SetPosition(i,
                Evaluate(p0.position, p1.position, p2.position, p3.position, t));
        }
    }

    // --- Sample forward direction at t (useful for orienting objects) ---
    public Vector3 GetForward(float t)
    {
        return Tangent(p0.position, p1.position, p2.position, p3.position, t).normalized;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How many control points define a cubic Bézier? :: 4 control points: $P_0$ (start), $P_1, P_2$ (handles), $P_3$ (end).
- Does a Bézier curve pass through all its control points? :: No — it only passes through the first ($P_0$) and last ($P_n$) points. Interior control points *influence* the shape.
- What is De Casteljau's algorithm? :: A recursive process of lerping adjacent control points at parameter $t$ until one point remains — the point on the curve.
- What is the tangent at $t=0$ for a cubic Bézier? :: $\mathbf{B}'(0) = 3(\mathbf{P}_1 - \mathbf{P}_0)$ — it points from the start toward the first control handle.
- What property do Bernstein basis polynomials guarantee? :: They sum to 1 for all $t \in [0,1]$, ensuring the curve stays within the convex hull of its control points.
- What is $C^1$ continuity between two Bézier segments? :: The curves share an endpoint AND have equal tangent vectors at that point, ensuring a smooth visual join.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Assuming uniform increments of $t$ produce evenly-spaced points along the curve.
> - **The Fix:** Use arc-length parameterization or a lookup table to map desired distances to the correct $t$ values.
> - **Why:** Bézier curves are *parametric*, not arc-length-parameterized. Tight bends compress many $t$ values into a short arc, while straight sections stretch them out. Objects moving with constant $\Delta t$ will speed up on straight parts and slow down in curves.

> [!danger] **Watch Out!**
> - **The Mistake:** Placing control points $P_1$ and $P_2$ far from the curve, creating wild oscillation.
> - **The Fix:** Keep control handles roughly proportional to the segment length (a common rule: handle length ≈ 1/3 of the distance between endpoints).
> - **Why:** Extreme handle lengths create loops or cusps. The convex-hull property means the curve stays bounded, but it can fold back on itself inside that hull.

---

## Related Topics
- [[Math/08_Curves_Interpolation/catmull_rom_splines|Catmull-Rom Splines]]
- [[Math/08_Curves_Interpolation/easing_functions|Easing Functions]]
- [[Math/08_Curves_Interpolation/curve_sampling_speed|Curve Sampling & Constant Speed]]
