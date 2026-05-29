---
title: "Planes & Half-Spaces: Collision Boundaries and Signed Distances"
tags:
  - math
  - level/Lv2
  - category/geometric_primitives
---

# Planes & Half-Spaces: Collision Boundaries and Signed Distances

> [!abstract] **The Concept in a Nutshell**
> A plane is a flat, infinite 2D surface extending through 3D space. While vectors represent directions and positions, a plane represents a boundary. Mathematically, a plane divides all of 3D space into two halves, called **half-spaces**: the positive half-space (the side the plane's normal points toward) and the negative half-space. This makes planes extremely useful for collision detection, view frustum culling, and determining which side of a wall an object is on.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Determining Which Side of a Mirror a Player is On**
> In a stealth game, you have a mirror surface (a plane). If the player is in front of the mirror, you render their reflection. If they walk behind the mirror wall, you must disable the reflection to save performance and prevent rendering bugs.
>
> By representing the mirror as a plane, you can calculate the **signed distance** from the player's position to the plane.
> - If distance is **positive** ($> 0$), the player is in front (positive half-space).
> - If distance is **zero** ($= 0$), the player is exactly on the mirror boundary.
> - If distance is **negative** ($< 0$), the player is behind it (negative half-space).

---

## The Blueprint (Formula & Structure)

### The Implicit Plane Equation
A plane in 3D space is defined by the equation:
$$ax + by + cz + d = 0$$

Where:
- $\vec{n} = (a, b, c)$: The **normal vector** perpendicular to the plane's surface.
- $(x, y, z)$: Any point on the plane.
- $d$: The offset constant representing the signed distance from the origin $(0, 0, 0)$ to the plane along the normal.
  $$d = -(\vec{n} \cdot \vec{P}_0)$$
  Where $\vec{P}_0$ is a known point on the plane.

### Normal-Distance Form
If the normal vector $\vec{n}$ is normalized ($|\vec{n}| = 1$), the equation is:
$$\vec{n} \cdot \vec{P} + d = 0$$
For any arbitrary point $\vec{Q}$ in space:
$$\text{Signed Distance} = \vec{n} \cdot \vec{Q} + d$$

### Classifying Points Relative to a Plane

| Signed Distance Value | Point Classification | Half-Space |
|-----------------------|----------------------|------------|
| $\vec{n} \cdot \vec{Q} + d > 0$ | Point is in front | Positive half-space |
| $\vec{n} \cdot \vec{Q} + d = 0$ | Point is on the plane | Boundary |
| $\vec{n} \cdot \vec{Q} + d < 0$ | Point is behind | Negative half-space |

### Building a Plane from Three Points
Given three non-collinear points in winding order $\vec{A}$, $\vec{B}$, and $\vec{C}$:
1. Find two vectors along the plane: $\vec{v}_1 = \vec{B} - \vec{A}$ and $\vec{v}_2 = \vec{C} - \vec{A}$.
2. Calculate the normal vector using the cross product:
   $$\vec{n} = \frac{\vec{v}_1 \times \vec{v}_2}{|\vec{v}_1 \times \vec{v}_2|}$$
3. Calculate the distance constant $d$:
   $$d = -(\vec{n} \cdot \vec{A})$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Spatial Knife**
> Think of a plane as a sheet of paper slicing space. The normal vector is a perpendicular arrow stuck through the paper pointing up. 
> 
> Anything in the direction the arrow is pointing is in the "positive zone." Anything on the back side of the paper is in the "negative zone." This division turns complex 3D checks into a simple greater-than-zero math check.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Signed distance to plane and half-space classification
using UnityEngine;

public class PlaneCollisionCheck : MonoBehaviour
{
    public Transform targetObject; // The object to check
    public Vector3 planeNormal = Vector3.up;
    public Vector3 pointOnPlane = Vector3.zero;

    private float d;

    void Start()
    {
        // Normalize normal vector
        planeNormal.Normalize();
        // Calculate plane offset d = -(n . P0)
        d = -Vector3.Dot(planeNormal, pointOnPlane);
    }

    void Update()
    {
        Vector3 q = targetObject.position;

        // Calculate signed distance: (n . q) + d
        float signedDistance = Vector3.Dot(planeNormal, q) + d;

        if (signedDistance > 0)
        {
            Debug.Log($"Target is in FRONT of plane. Distance: {signedDistance:F2}");
        }
        else if (signedDistance < 0)
        {
            Debug.Log($"Target is BEHIND plane. Distance: {signedDistance:F2}");
        }
        else
        {
            Debug.Log("Target is EXACTLY on the plane.");
        }
        
        // Visual representation of normal
        Debug.DrawRay(pointOnPlane, planeNormal * 3f, Color.cyan);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the implicit equation of a plane? :: $ax + by + cz + d = 0$.
- What does the sign of the distance calculation $\vec{n} \cdot \vec{Q} + d$ tell you? :: It tells you which half-space the point $\vec{Q}$ occupies: positive (front), negative (behind), or zero (on the plane).
- How do you compute the plane constant $d$ from a normal vector $\vec{n}$ and a point on the plane $\vec{P}_0$? :: $d = -(\vec{n} \cdot \vec{P}_0)$.
- In which direction does the plane's normal vector point by definition? :: Towards the positive half-space.
- What mathematical tool is used to calculate the plane normal from three points? :: The **cross product** of the vectors between the points.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using an un-normalized normal vector $(a, b, c)$ when calculating signed distance.
> - **The Fix:** Always ensure the normal vector is normalized to unit length ($1.0$).
> - **Why:** If the normal vector is not normalized, the result of $\vec{n} \cdot \vec{Q} + d$ will be scaled by the normal's magnitude, returning an incorrect distance value (though the *sign* will still be correct).

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/05_Coordinate_Spaces/view_frustum|The View Frustum]]
- [[Math/07_Geometric_Primitives/parametric_lines_rays|Parametric Lines & Rays]]
