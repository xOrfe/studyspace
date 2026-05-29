---
title: "Intersection Testing: Raycasting, Spheres, Planes, and Möller-Trumbore"
tags:
  - math
  - level/Lv3
  - category/geometric_primitives
---

# Intersection Testing: Raycasting, Spheres, Planes, and Möller-Trumbore

> [!abstract] **The Concept in a Nutshell**
> Intersection testing is the mathematical process of determining if and where two geometric shapes overlap in space. It is the core algorithm behind raycasting, camera visibility checks, mouse picking (clicking on a 3D object), and physics collision resolution. Running these tests involves solving algebraic equations for the parametric distance parameter $t$ to locate exact points of contact.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Selecting a 3D Unit by Clicking the Mouse**
> In a real-time strategy (RTS) game, the player clicks on a soldier mesh in the 3D world to select them. The monitor is a flat 2D screen, but the game world is 3D.
>
> To resolve this click:
> 1. The engine generates a **ray** from the camera's lens passing through the 2D cursor coordinate on the screen.
> 2. The engine tests this ray against the bounding volumes (AABBs or Spheres) of all units in the scene.
> 3. If a unit's bounding sphere is hit, the engine runs a highly precise **Ray-Triangle intersection test** (using the **Möller-Trumbore algorithm**) against the unit's mesh triangles to confirm if the player clicked the unit or just clicked the empty space near them.

---

## The Blueprint (Formula & Structure)

### 1. Ray-Plane Intersection
Given a ray $\vec{P}(t) = \vec{O} + t\vec{d}$ and a plane $\vec{n} \cdot \vec{P} + d = 0$:
1. Substitute the ray equation into the plane equation:
   $$\vec{n} \cdot (\vec{O} + t\vec{d}) + d = 0$$
2. Distribute and solve for $t$:
   $$t = \frac{-(\vec{n} \cdot \vec{O} + d)}{\vec{n} \cdot \vec{d}}$$
- **Edge Cases:**
  - If $\vec{n} \cdot \vec{d} = 0$, the ray is parallel to the plane (no intersection, or infinite if on it).
  - If $t < 0$, the plane is behind the ray's origin.

### 2. Ray-Sphere Intersection
Given a ray $\vec{P}(t) = \vec{O} + t\vec{d}$ and a sphere centered at $\vec{C}$ with radius $r$:
1. Set the distance from a point on the ray to the center equal to $r$:
   $$|\vec{O} + t\vec{d} - \vec{C}|^2 = r^2$$
2. This expands into a quadratic equation: $At^2 + Bt + C = 0$, where:
   - $A = \vec{d} \cdot \vec{d} = 1$ (if normalized)
   - $B = 2(\vec{d} \cdot (\vec{O} - \vec{C}))$
   - $C = (\vec{O} - \vec{C}) \cdot (\vec{O} - \vec{C}) - r^2$
3. Solve using the quadratic formula:
   $$t = \frac{-B \pm \sqrt{B^2 - 4AC}}{2A}$$
- **Discriminant ($D = B^2 - 4AC$):**
  - $D < 0$: No intersection (ray misses sphere).
  - $D = 0$: Ray is tangent to sphere (one contact point).
  - $D > 0$: Ray passes through sphere (two points; the smaller positive $t$ is the entry point).

### 3. Ray-Triangle (Möller-Trumbore Algorithm)
The standard industry algorithm. Instead of intersecting a plane first and checking if the point lies inside the triangle, Möller-Trumbore uses barycentric coordinates directly to solve for $t$, $u$, and $v$ simultaneously using linear algebra:
$$\vec{O} + t\vec{d} = (1 - u - v)\vec{V}_0 + u\vec{V}_1 + v\vec{V}_2$$
The system is solved using Cramer's rule. If $u \ge 0, v \ge 0$, and $u + v \le 1.0$, the intersection lies inside the triangle.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Projecting a Thread**
> Imagine holding a laser pointer (ray) and shining it at a target.
> - For a **plane**, you are checking where the light spot hits the infinite wall.
> - For a **sphere**, you check if the beam pierces the surface and exits the other side.
> - For a **triangle**, you check if the beam lands inside the boundary lines of the three vertices.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Implementing Ray-Plane and Ray-Sphere Intersection tests
using UnityEngine;

public class IntersectionTests : MonoBehaviour
{
    // Ray-Plane Intersection
    // Returns true and sets t if intersection occurs in front of ray
    public static bool IntersectRayPlane(Vector3 origin, Vector3 direction, Vector3 planeNormal, Vector3 pointOnPlane, out float t)
    {
        t = 0f;
        float denom = Vector3.Dot(planeNormal, direction);
        
        // Check if ray is parallel to plane
        if (Mathf.Abs(denom) < 0.0001f)
            return false;

        float d = -Vector3.Dot(planeNormal, pointOnPlane);
        t = -(Vector3.Dot(planeNormal, origin) + d) / denom;
        
        return t >= 0f;
    }

    // Ray-Sphere Intersection
    public static bool IntersectRaySphere(Vector3 origin, Vector3 direction, Vector3 sphereCenter, float sphereRadius, out float t)
    {
        t = 0f;
        Vector3 oc = origin - sphereCenter;
        
        // Quadratic coefficients
        float a = Vector3.Dot(direction, direction); // 1.0f if normalized
        float b = 2f * Vector3.Dot(direction, oc);
        float c = Vector3.Dot(oc, oc) - (sphereRadius * sphereRadius);
        
        float discriminant = (b * b) - (4f * a * c);
        
        if (discriminant < 0)
        {
            return false; // Ray misses sphere
        }
        else
        {
            // Find closest positive intersection point
            float t1 = (-b - Mathf.Sqrt(discriminant)) / (2f * a);
            float t2 = (-b + Mathf.Sqrt(discriminant)) / (2f * a);

            if (t1 >= 0)
            {
                t = t1;
                return true;
            }
            if (t2 >= 0)
            {
                t = t2;
                return true;
            }
            return false; // Both points are behind origin
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does a negative discriminant ($B^2 - 4AC < 0$) mean in a ray-sphere intersection test? :: The ray misses the sphere (no real roots/intersection points exist).
- What formula is used to check if a ray is parallel to a plane? :: Take the dot product: $\vec{n} \cdot \vec{d} = 0$, where $\vec{n}$ is the plane normal and $\vec{d}$ is the ray direction.
- Why is the Möller-Trumbore algorithm highly preferred for ray-triangle intersection? :: It computes barycentric coordinates directly without needing to calculate the plane equation first, saving memory and instruction cycles.
- In ray-plane intersection, what does $t < 0$ indicate? :: The plane lies behind the ray's origin point.
- How many potential intersection points can a ray have with a sphere? :: Zero (miss), one (tangent), or two (entry and exit points).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Neglecting to check if the denominator ($\vec{n} \cdot \vec{d}$) is close to zero in ray-plane checks.
> - **The Fix:** Wrap the division in an epsilon check: `if (Mathf.Abs(denom) < 0.0001f) return false;`.
> - **Why:** If the ray is parallel to the plane, division by zero occurs, generating a `NaN` (Not a Number) coordinate value that crashes simulation logic.

---

## Related Topics
- [[Math/07_Geometric_Primitives/parametric_lines_rays|Parametric Lines & Rays]]
- [[Math/07_Geometric_Primitives/planes_implicit|Planes & Half-Spaces]]
- [[Math/07_Geometric_Primitives/bounding_volumes|Bounding Volumes]]
