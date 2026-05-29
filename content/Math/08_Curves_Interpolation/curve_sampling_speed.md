---
title: "Curve Sampling & Constant Speed — Taming the Parameter"
tags:
  - math
  - level/Lv3
  - category/curves_interpolation
---

# Curve Sampling & Constant Speed: Taming the Parameter

> [!abstract] **The Concept in a Nutshell**
> When you traverse a parametric curve by incrementing $t$ uniformly, your speed along the curve is *not* constant — you accelerate through straight sections and slow down around tight bends. Arc-length parameterization solves this by building a mapping from distance $s$ to parameter $t$, so that equal increments of $s$ produce equal distances traveled. This is essential for any game object that must move along a path at a steady pace.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Moving Platform on a Spline**
> Your 3D platformer has a floating platform that follows a looping Bézier spline around a tower. If you advance $t$ by a fixed $\Delta t$ each frame, the platform crawls through the tight spiral section and zooms through the gentle straight-away — disorienting the player. With arc-length parameterization, you specify the platform's speed in world units per second (say 3 m/s). Each frame, you compute $\Delta s = 3 \cdot \Delta t_{\text{frame}}$, look up the corresponding $t$ from your arc-length table, and evaluate the curve. The platform now glides at a perfectly constant speed regardless of curve shape.

---

## The Blueprint (Formula & Structure)

### Why Uniform $t$ ≠ Uniform Speed

For a parametric curve $\mathbf{C}(t)$, the instantaneous speed at parameter $t$ is:

$$v(t) = \left\| \frac{d\mathbf{C}}{dt} \right\| = \| \mathbf{C}'(t) \|$$

This magnitude changes as $t$ varies — it depends on the geometry of the curve. Only when $\|\mathbf{C}'(t)\|$ is constant for all $t$ (which almost never happens in practice) does uniform $t$ give uniform speed.

### Arc Length

The total arc length from $t = 0$ to some parameter $t = T$ is:

$$s(T) = \int_0^T \| \mathbf{C}'(t) \| \, dt$$

The full curve length is $L = s(1)$.

### The Goal: Find $t$ for a Given $s$

We want the inverse mapping $t = s^{-1}(d)$: "What value of $t$ corresponds to having traveled distance $d$ along the curve?"

This integral generally has no closed-form solution for Bézier/Catmull-Rom curves, so we use a **numerical lookup table**.

### Step-by-Step: Building an Arc-Length Table

1. **Sample the curve** at $N$ uniform $t$ values: $t_0 = 0,\; t_1 = \frac{1}{N},\; \ldots,\; t_N = 1$
2. **Compute cumulative distances** between consecutive samples:

$$s_0 = 0, \quad s_k = s_{k-1} + \| \mathbf{C}(t_k) - \mathbf{C}(t_{k-1}) \|$$

3. **Store** the table of pairs $(s_k, t_k)$

The total arc length is $L = s_N$.

### Step-by-Step: Querying the Table (Constant-Speed Traversal)

Given a desired distance $d$ along the curve:

1. **Binary search** the arc-length table to find interval $[s_k, s_{k+1}]$ containing $d$
2. **Linearly interpolate** to get the corresponding $t$:

$$\alpha = \frac{d - s_k}{s_{k+1} - s_k}, \quad t_d = \text{lerp}(t_k, t_{k+1}, \alpha)$$

3. **Evaluate** the curve at $t_d$: $\mathbf{C}(t_d)$

### Accuracy vs Performance

| Parameter | Low Quality | Medium | High Quality |
|---|---|---|---|
| Samples $N$ | 32 | 128 | 512+ |
| Build cost | Cheap | Moderate | Expensive |
| Query cost | Always $O(\log N)$ via binary search | | |
| Accuracy | Visible speed variation | Good for gameplay | Smooth cinematics |

For most game paths, $N = 100$–$200$ is more than sufficient. Build the table once when the curve is created, then query it every frame.

### Alternative: Newton-Raphson Refinement

For extra precision without a huge table, after the binary search step, refine with Newton's method:

$$t_{n+1} = t_n + \frac{d_{\text{target}} - s(t_n)}{\| \mathbf{C}'(t_n) \|}$$

A few iterations typically converge quickly.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Mile Markers on a Winding Road**
> Imagine driving on a mountain road. The highway engineer placed mile markers (arc length $s$) at equal physical intervals. But the road's mathematical parameter $t$ (think: percentage of the total road blueprint) doesn't correspond — a tight switchback might occupy $t \in [0.3, 0.5]$ (20% of the parameter range) but only cover 0.5 miles, while a straight valley stretch might be $t \in [0.5, 0.6]$ (10% of the parameter) but cover 3 miles. The arc-length table is your road atlas — it tells you which mile marker corresponds to which blueprint percentage, so your cruise control (constant speed) works correctly.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Arc-length parameterized Bézier path traversal
using UnityEngine;

public class ArcLengthPath : MonoBehaviour
{
    public Transform p0, p1, p2, p3;
    public float speed = 3f; // world units per second

    private float[] arcLengths;
    private float totalLength;
    private int sampleCount = 200;
    private float distanceTraveled = 0f;

    void Start()
    {
        BuildArcLengthTable();
    }

    void Update()
    {
        // Advance by constant distance
        distanceTraveled += speed * Time.deltaTime;
        distanceTraveled %= totalLength; // Loop

        float t = DistanceToT(distanceTraveled);
        transform.position = EvalBezier(t);

        // Orient along curve
        Vector3 tangent = EvalBezierTangent(t);
        if (tangent.sqrMagnitude > 0.001f)
            transform.forward = tangent.normalized;
    }

    /// <summary>Build a cumulative arc-length lookup table.</summary>
    void BuildArcLengthTable()
    {
        arcLengths = new float[sampleCount + 1];
        arcLengths[0] = 0f;
        Vector3 prev = EvalBezier(0f);

        for (int i = 1; i <= sampleCount; i++)
        {
            float t = i / (float)sampleCount;
            Vector3 curr = EvalBezier(t);
            arcLengths[i] = arcLengths[i - 1] + Vector3.Distance(prev, curr);
            prev = curr;
        }

        totalLength = arcLengths[sampleCount];
    }

    /// <summary>Binary search the table to find t for a given distance.</summary>
    float DistanceToT(float distance)
    {
        int lo = 0, hi = sampleCount;

        while (lo < hi - 1)
        {
            int mid = (lo + hi) / 2;
            if (arcLengths[mid] < distance)
                lo = mid;
            else
                hi = mid;
        }

        // Linear interpolation within the found interval
        float segLen = arcLengths[hi] - arcLengths[lo];
        float alpha = segLen > 0f ? (distance - arcLengths[lo]) / segLen : 0f;
        return Mathf.Lerp(lo / (float)sampleCount, hi / (float)sampleCount, alpha);
    }

    // --- Cubic Bézier helpers ---
    Vector3 EvalBezier(float t)
    {
        float u = 1f - t;
        return u * u * u * p0.position +
               3f * u * u * t * p1.position +
               3f * u * t * t * p2.position +
               t * t * t * p3.position;
    }

    Vector3 EvalBezierTangent(float t)
    {
        float u = 1f - t;
        return 3f * u * u * (p1.position - p0.position) +
               6f * u * t * (p2.position - p1.position) +
               3f * t * t * (p3.position - p2.position);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Why doesn't uniform $\Delta t$ produce constant speed along a curve? :: Because $\|\mathbf{C}'(t)\|$ varies — the distance traveled per unit $t$ depends on the curve geometry at that point.
- What is arc-length parameterization? :: A remapping from distance $s$ to parameter $t$ such that equal increments of $s$ correspond to equal distances along the curve.
- How do you build an arc-length lookup table? :: Sample the curve at $N$ uniform $t$ values, measure the distance between consecutive samples, and store cumulative distances alongside their $t$ values.
- How do you query the arc-length table for a target distance? :: Binary search for the interval containing the target distance, then linearly interpolate between the two bounding $t$ values.
- What is a typical sample count for a game path? :: $N = 100$–$200$ is usually sufficient; cinematics may use $500+$.
- What is Newton-Raphson refinement in this context? :: After finding an approximate $t$ via lookup, iteratively improve it using $t_{n+1} = t_n + \frac{d_{\text{target}} - s(t_n)}{\|\mathbf{C}'(t_n)\|}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Building the arc-length table every frame instead of caching it.
> - **The Fix:** Compute the table once at initialization (or when control points change). Cache it and reuse.
> - **Why:** Building the table requires $N$ curve evaluations — that's fine once, but doing it at 60 fps is a huge waste. Only rebuild when the curve's control points actually move.

> [!danger] **Watch Out!**
> - **The Mistake:** Using too few samples in the lookup table, causing visible speed jitter (especially on tight curves).
> - **The Fix:** Increase sample count or use adaptive sampling (more samples where curvature is high).
> - **Why:** With too few samples, the linear interpolation between entries introduces error proportional to how much the curve bends within each interval. Sharp bends need denser sampling.

---

## Related Topics
- [[Math/08_Curves_Interpolation/bezier_curves|Bézier Curves]]
- [[Math/08_Curves_Interpolation/catmull_rom_splines|Catmull-Rom Splines]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration]]
