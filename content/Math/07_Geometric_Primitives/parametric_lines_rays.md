---
title: "Parametric Lines & Rays: Pathing, Shooting, and Raycasting"
tags:
  - math
  - level/Lv2
  - category/geometric_primitives
---

# Parametric Lines & Rays: Pathing, Shooting, and Raycasting

> [!abstract] **The Concept in a Nutshell**
> In geometry, a line is infinite. In game programming, we represent lines, rays, and line segments using a **parametric equation**. Instead of using slope-intercept equations ($y = mx + b$), we represent a line as a starting point (origin $\vec{O}$) and a direction vector ($\vec{d}$) scaled by a scalar parameter $t$. By changing the range of $t$, we can represent infinite lines, one-directional rays (bullets, lasers), or finite line segments (movement paths).

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Raycasting for a Hitscan Weapon**
> When a player fires a sniper rifle in an FPS game, the engine doesn't spawn a physical projectile. Instead, it performs a **raycast** — instant hit detection.
>
> The gun muzzle is the ray **origin** ($\vec{O}$). The barrel direction is the ray **direction** ($\vec{d}$). The line representing the bullet's path is modeled as:
> $$\vec{P}(t) = \vec{O} + t\vec{d}$$
>
> To check if the bullet hits an enemy collider, the physics engine solves for $t$ where the ray intersects the collider's shape.
> - If $t < 0$, the intersection is *behind* the player (ignored).
> - If $t \ge 0$, it is in front. The smallest positive value of $t$ represents the closest obstacle hit. We multiply $t$ by the length of $\vec{d}$ to find the exact distance of the shot.

---

## The Blueprint (Formula & Structure)

### The Parametric Vector Equation
Any point $\vec{P}$ along the line can be represented as:
$$\vec{P}(t) = \vec{O} + t\vec{d}$$

Where:
- $\vec{O}$: Origin point (vector position).
- $\vec{d}$: Direction vector (often normalized).
- $t$: The scalar parameter.

### Line Types Defined by $t$

| Primitive | Parameter Range | Game Dev Analogy |
|-----------|-----------------|------------------|
| **Infinite Line** | $-\infty < t < \infty$ | Geometric axis alignment |
| **Ray** | $0 \le t < \infty$ | Muzzle flash, sight check, laser beam |
| **Line Segment** | $0 \le t \le t_{\text{max}}$ | Path from waypoint A to B, bullet range |

### Distance from a Point to a Line
To find the shortest distance from an arbitrary point $\vec{Q}$ to a line defined by origin $\vec{O}$ and normalized direction $\vec{d}$:

1. Project the vector from origin to point $(\vec{Q} - \vec{O})$ onto direction $\vec{d}$:
   $$t_{\text{closest}} = (\vec{Q} - \vec{O}) \cdot \vec{d}$$
2. The closest point $\vec{P}_{\text{closest}}$ on the line is:
   $$\vec{P}_{\text{closest}} = \vec{O} + t_{\text{closest}}\vec{d}$$
3. The distance is the magnitude of the offset:
   $$\text{distance} = |\vec{Q} - \vec{P}_{\text{closest}}|$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Timeline Slider**
> Imagine $\vec{O}$ as a train station and $\vec{d}$ as the velocity of a train. The parameter $t$ is a timeline slider.
> - At $t = 0$, you are at the station.
> - At $t = 1$, you are 1 unit of distance away along the track.
> - At $t = -1$, you went backward in time (behind the origin).
> 
> Parametric representations turn a static geometric line into a dynamic path that is easy to navigate using simple numbers.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Parametric ray definition and distance checks
using UnityEngine;

public class ParametricRayDemo : MonoBehaviour
{
    public Transform target; // Point to check distance to

    // Define our ray programmatically
    private Vector3 rayOrigin;
    private Vector3 rayDirection;

    void Update()
    {
        rayOrigin = transform.position;
        rayDirection = transform.forward; // Normalized direction vector

        // Vector from ray origin to the target point
        Vector3 originToTarget = target.position - rayOrigin;

        // Project target onto ray direction to find closest t
        float t = Vector3.Dot(originToTarget, rayDirection);

        // We only want points in front of the origin (Ray, t >= 0)
        t = Mathf.Max(0f, t);

        // Find the coordinates of the closest point on the ray
        Vector3 closestPoint = rayOrigin + t * rayDirection;

        // Draw the ray in the editor
        Debug.DrawRay(rayOrigin, rayDirection * 10f, Color.green);
        // Draw line to closest point
        Debug.DrawLine(target.position, closestPoint, Color.red);

        float distance = Vector3.Distance(target.position, closestPoint);
        Debug.Log($"Distance to ray: {distance:F2} at t={t:F2}");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the parametric equation of a ray? :: $\vec{P}(t) = \vec{O} + t\vec{d}$, where $t \ge 0$.
- What is the difference between a ray and a line segment in terms of the parameter $t$? :: A ray has $t \in [0, \infty)$, whereas a line segment has $t \in [0, t_{\text{max}}]$.
- How do you find the parameter $t$ of the closest point on a line to a point $\vec{Q}$? :: Take the dot product: $t = (\vec{Q} - \vec{O}) \cdot \vec{d}$ (assuming $\vec{d}$ is normalized).
- What does it mean if $t < 0$ in a raycast intersection? :: The intersection point lies behind the ray's origin point.
- Why is parametric representation better than $y = mx + b$ in games? :: It works in any number of dimensions (2D, 3D, etc.) and naturally handles vertical lines where slope $m$ would be infinite.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using an un-normalized direction vector $\vec{d}$ in the parametric equation without adjusting calculations.
> - **The Fix:** Always normalize the direction vector $\vec{d}$ before performing projections or distance calculations.
> - **Why:** If $\vec{d}$ is not normalized, the parameter $t$ does not represent actual distance units, which corrupts dot product projections and distance checks.

---

## Related Topics
- [[Math/03_Vectors/dot_product|Dot Product]]
- [[Math/07_Geometric_Primitives/planes_implicit|Planes & Half-Spaces]]
- [[Math/07_Geometric_Primitives/intersection_testing|Intersection Testing]]
