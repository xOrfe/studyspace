---
title: "Normal Vectors: Surfaces, Lighting directions, and Transforming Normals"
tags:
  - math
  - level/Lv2
  - category/geometric_primitives
---

# Normal Vectors: Surfaces, Lighting directions, and Transforming Normals

> [!abstract] **The Concept in a Nutshell**
> A normal vector (or simply a **normal**) is a unit vector that points directly perpendicular to a surface. In game development, normals are the mathematical core of rendering, lighting, and physics. They tell the graphics card which direction a polygon is facing, allowing it to calculate how light bounces off the surface. Without normal vectors, all 3D objects would look completely flat and un-shaded.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Flat vs. Smooth Shading on a Sphere**
> Imagine two 3D spheres. One looks like a low-poly disco ball (flat shading), and the other looks perfectly round and smooth (smooth shading). Both models have the exact same number of triangles!
>
> The difference lies in how their normals are computed:
> - **Flat Shading:** The engine uses **Face Normals**. Every point on a single triangle shares the same normal vector. The lighting changes abruptly at the edges, highlighting individual flat triangles.
> - **Smooth Shading:** The engine uses **Vertex Normals**. Normals are defined at the vertices (pointing outward from the sphere center) and are smoothly interpolated across the triangle using barycentric coordinates. The GPU calculates lighting based on these blended directions, hiding the triangle edges and making the surface look perfectly round.

---

## The Blueprint (Formula & Structure)

### 1. Computing Face Normals
For a triangle with vertices $\vec{A}$, $\vec{B}$, and $\vec{C}$ in clockwise order:
1. Find two edge vectors: $\vec{v}_1 = \vec{B} - \vec{A}$ and $\vec{v}_2 = \vec{C} - \vec{A}$.
2. Take the cross product to find a perpendicular vector:
   $$\vec{n}_{\text{unnormalized}} = \vec{v}_1 \times \vec{v}_2$$
3. Normalize to get the unit normal:
   $$\vec{n} = \frac{\vec{n}_{\text{unnormalized}}}{|\vec{n}_{\text{unnormalized}}|}$$

### 2. Computing Vertex Normals (Average Normals)
To calculate a vertex normal for a shared vertex:
$$\vec{n}_{\text{vertex}} = \text{normalize}\left(\sum_{i=1}^{k} \vec{n}_{\text{face}, i}\right)$$
Where $\vec{n}_{\text{face}, i}$ are the normals of all triangles sharing that vertex.

### 3. Transforming Normals (The Normal Matrix)
When an object is rotated or scaled, its normal vectors must also be updated. However, you **cannot** simply multiply normals by the same model-view matrix used for positions.
- If the object is stretched unevenly (non-uniform scale, e.g., scaled by $2$ on X and $1$ on Y), multiplying the normal directly makes it *no longer perpendicular* to the surface.
- **The Solution:** Multiply normal vectors by the **transpose of the inverse** of the rotation-scale matrix:
  $$\vec{n}_{\text{transformed}} = \text{normalize}\left( (M^{-1})^T \vec{n} \right)$$
  If the matrix only contains rotation and uniform scaling, you can multiply normals directly by the matrix and re-normalize them.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Surface Needles**
> Imagine a hedgehog. Its needles point straight out, perpendicular to its skin.
> - The needles represent the normal vectors.
> - If the hedgehog moves, rotates, or gets squished, the needles must adjust to remain perpendicular to the surface.
> - If a light shines on the hedgehog, the needles pointing directly at the light reflect the most brightness, while needles pointing sideways receive less.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Computing face normal from three vertices and transforming it
using UnityEngine;

public class NormalComputation : MonoBehaviour
{
    // Compute face normal in local space
    public static Vector3 ComputeFaceNormal(Vector3 a, Vector3 b, Vector3 c)
    {
        Vector3 side1 = b - a;
        Vector3 side2 = c - a;
        
        // Perpendicular vector via cross product
        Vector3 normal = Vector3.Cross(side1, side2);
        
        return normal.normalized;
    }

    void Update()
    {
        // Example: Getting the normal of a game object's plane and transforming it
        Vector3 localNormal = Vector3.up;

        // Correctly transforming normal to world space
        // Unity handles the normal matrix automatically under transform.TransformDirection
        Vector3 worldNormal = transform.TransformDirection(localNormal);

        // Visual representation in scene
        Debug.DrawRay(transform.position, worldNormal * 2f, Color.blue);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is a normal vector? :: A unit vector ($1.0$ length) perpendicular to a surface.
- Why can't we use the standard model matrix to transform normals when non-uniform scaling is present? :: Non-uniform scaling distorts the angles, causing the transformed normal to lose its perpendicular orientation to the surface.
- What matrix must be used to transform normal vectors? :: The **transpose of the inverse** of the top-left 3x3 model matrix: $(M^{-1})^T$.
- How is a face normal computed? :: Take the cross product of two edge vectors of the triangle and normalize the result: $\text{normalize}((\vec{B} - \vec{A}) \times (\vec{C} - \vec{A}))$.
- What is the difference between face normals and vertex normals? :: Face normals are uniform across a polygon (producing flat shading), while vertex normals are defined at the corners and interpolated across the face (producing smooth shading).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to re-normalize normal vectors after transforming them in shaders.
> - **The Fix:** Always call `normalize()` on transformed normal vectors in the fragment/vertex shader.
> - **Why:** Matrix transformations (even just rotation or scaling) can change the length of the normal vector. If length differs from $1.0$, lighting equations (which rely on unit vectors) produce blown-out or overly dark artifacts.

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/07_Geometric_Primitives/triangles_meshes|Triangles & Meshes]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
