---
title: "Bounding Volumes: Broad-Phase Collisions, AABBs, and Spheres"
tags:
  - math
  - level/Lv2
  - category/geometric_primitives
---

# Bounding Volumes: Broad-Phase Collisions, AABBs, and Spheres

> [!abstract] **The Concept in a Nutshell**
> Testing for collisions between complex 3D meshes (which can contain thousands of triangles) is extremely expensive. To solve this, game engines wrap meshes inside simpler geometric containers called **bounding volumes**. By running quick, inexpensive checks on these bounding shapes first (broad-phase collision), we can immediately skip detailed mesh tests for objects that have no chance of overlapping.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Optimizing Physics with Broad-Phase Checks**
> Imagine two high-polygon characters fighting. Each character model is composed of $50{,}000$ triangles.
>
> Checking every single triangle of Character A against every triangle of Character B would require $2.5$ billion checks per frame.
> Instead, the engine surrounds each character with a **Bounding Sphere** or an **Axis-Aligned Bounding Box (AABB)**.
> - Calculating sphere-sphere overlap requires a single distance check (using Pythagorean optimization).
> - If the bounding spheres do not overlap, we know with $100\%$ certainty that the meshes are not touching. We perform $1$ check instead of $2.5$ billion, freeing up the CPU for gameplay.

---

## The Blueprint (Formula & Structure)

### 1. Bounding Sphere
Defined by a center point $\vec{C}$ and radius $r$.
- **Sphere-Sphere Overlap Test:**
  Two spheres $S_1(\vec{C}_1, r_1)$ and $S_2(\vec{C}_2, r_2)$ overlap if the distance between their centers is less than the sum of their radii:
  $$|\vec{C}_1 - \vec{C}_2| < r_1 + r_2$$
  *Optimization:* Use squared distance:
  $$|\vec{C}_1 - \vec{C}_2|^2 < (r_1 + r_2)^2$$

### 2. Axis-Aligned Bounding Box (AABB)
A box whose faces are aligned with the world coordinate axes. Defined by a minimum corner $\vec{P}_{\text{min}}(x_{\text{min}}, y_{\text{min}}, z_{\text{min}})$ and maximum corner $\vec{P}_{\text{max}}(x_{\text{max}}, y_{\text{max}}, z_{\text{max}})$, or center + half-extents.
- **AABB-AABB Overlap Test:**
  Two boxes $A$ and $B$ overlap if they overlap along all three axes simultaneously:
  $$\text{Overlap} = (A_{\text{min}.x} \le B_{\text{max}.x} \land A_{\text{max}.x} \ge B_{\text{min}.x}) \land (A_{\text{min}.y} \le B_{\text{max}.y} \land A_{\text{max}.y} \ge B_{\text{min}.y}) \land (A_{\text{min}.z} \le B_{\text{max}.z} \land A_{\text{max}.z} \ge B_{\text{min}.z})$$

### 3. Oriented Bounding Box (OBB)
A box that can rotate alongside the object. Defined by a center point, three orthogonal basis direction axes, and half-extents.
- OBBs provide much tighter fits for rotated objects, but testing overlap requires the **Separating Axis Theorem (SAT)**, which is significantly more complex and expensive than AABB checks.

### Comparison Table

| Volume Type | Fit Quality | Computational Cost | Invariant to Rotation? |
|-------------|-------------|--------------------|------------------------|
| **Sphere** | Poor (gaps) | Lowest (1 distance check) | Yes |
| **AABB** | Medium | Low (6 comparisons) | No (must resize when object rotates) |
| **OBB** | Tightest | High (15 axes tests via SAT) | Yes |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Nesting Dolls**
> Bounding volumes are like protective wooden crates built around fragile models.
> - The cheapest crate to build and inspect is a sphere (just a bubble).
> - The next is an AABB, which stays locked straight with the compass directions but grows and shrinks as the object inside rotates.
> - The OBB is a custom-fitted crate that spins along with the object inside.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Implementing custom AABB and Sphere overlap tests
using UnityEngine;

public class BoundingVolumeDemo : MonoBehaviour
{
    // Custom Sphere Definition
    public struct BoundingSphere
    {
        public Vector3 center;
        public float radius;

        public bool Overlaps(BoundingSphere other)
        {
            float sqrDistance = (center - other.center).sqrMagnitude;
            float sumRadii = radius + other.radius;
            return sqrDistance < (sumRadii * sumRadii);
        }
    }

    // Custom AABB Definition
    public struct CustomAABB
    {
        public Vector3 min;
        public Vector3 max;

        public bool Overlaps(CustomAABB other)
        {
            return (min.x <= other.max.x && max.x >= other.min.x) &&
                   (min.y <= other.max.y && max.y >= other.min.y) &&
                   (min.z <= other.max.z && max.z >= other.min.z);
        }
    }

    void Start()
    {
        // Simple sphere check
        BoundingSphere s1 = new BoundingSphere { center = Vector3.zero, radius = 2f };
        BoundingSphere s2 = new BoundingSphere { center = new Vector3(3f, 0f, 0f), radius = 2f };
        Debug.Log($"Spheres overlap? {s1.Overlaps(s2)}"); // True (distance 3 < radius sum 4)

        // Simple AABB check
        CustomAABB box1 = new CustomAABB { min = new Vector3(-1, -1, -1), max = new Vector3(1, 1, 1) };
        CustomAABB box2 = new CustomAABB { min = new Vector3(1.5f, 0, 0), max = new Vector3(2.5f, 1, 1) };
        Debug.Log($"Boxes overlap? {box1.Overlaps(box2)}"); // False
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What parameters define a Bounding Sphere? :: A center point position vector $\vec{C}$ and a radius scalar $r$.
- What does AABB stand for? :: **A**xis-**A**ligned **B**ounding **B**ox.
- What is the difference between an AABB and an OBB? :: An AABB's axes are locked parallel to the world coordinate axes, whereas an OBB's axes rotate dynamically alongside the object's orientation.
- Why must an AABB be resized when the object inside rotates? :: Because rotation changes the object's width, height, and depth along the world axes.
- What mathematical test is used to check for OBB-OBB intersections? :: The **Separating Axis Theorem** (SAT).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using an object's local AABB for intersection tests in world space.
> - **The Fix:** Always transform the AABB's local min/max points into world space coordinates using the model transformation matrix before running collision calculations.
> - **Why:** Local coordinates are relative to the object's center. Running a world collision check using local coordinates causes physics to behave as if all objects are clustered together at the world origin $(0, 0, 0)$.

---

## Related Topics
- [[Math/02_Geometry_Trigonometry/distance_and_pythagorean|Distance & the Pythagorean Theorem]]
- [[Math/10_Physics_Math/collision_detection|Collision Detection]]
- [[Math/07_Geometric_Primitives/intersection_testing|Intersection Testing]]
