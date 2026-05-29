---
title: "Triangles & Meshes: Vertices, Indices, and Barycentric Space"
tags:
  - math
  - level/Lv2
  - category/geometric_primitives
---

# Triangles & Meshes: Vertices, Indices, and Barycentric Space

> [!abstract] **The Concept in a Nutshell**
> Almost every 3D object in modern video games is composed of **triangle meshes**. A triangle is the simplest possible polygon that can define a flat surface in 3D space. It is guaranteed to be planar (flat) and convex, which simplifies rendering and physics math. A mesh is a collection of vertices, connected by indices, that form these triangles. Inside a triangle, any point can be represented using **barycentric coordinates**, which act as a local coordinate system for interpolating values like colors, normals, and texture coordinates.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Calculating Texture Coordinates (UVs) at Ray Hit Point**
> When a player shoots a wall with a paintball gun, the physics engine performs a raycast and finds that the ray hit a specific triangle on the wall's mesh. To paint a splat texture at the exact spot, you need to know the texture coordinates (UVs) at the point of impact.
>
> A mesh vertex only has UV coordinates at its corners: $V_0(u_0, v_0)$, $V_1(u_1, v_1)$, and $V_2(u_2, v_2)$. 
> By converting the 3D ray hit position into **barycentric coordinates** $(w_0, w_1, w_2)$, we get three weight factors that sum to $1.0$.
> We then interpolate the UV coordinate at the hit point:
> $$\text{Hit UV} = w_0(u_0, v_0) + w_1(u_1, v_1) + w_2(u_2, v_2)$$
> This allows the game to accurately map details to any point on a mesh surface.

---

## The Blueprint (Formula & Structure)

### Mesh Structure: Vertices and Indices
Instead of storing three coordinate points for every triangle, meshes use two separate buffers to optimize memory and performance:
1. **Vertex Buffer (Positions):** A list of unique coordinate points in space.
   $$[\vec{V}_0, \vec{V}_1, \vec{V}_2, \vec{V}_3, \dots]$$
2. **Index Buffer (Topology):** A list of integers grouping vertices into triangles. For example, indices $[0, 1, 2, 0, 2, 3]$ represent two triangles sharing vertices $\vec{V}_0$ and $\vec{V}_2$.

### Winding Order & Backface Culling
The order in which vertices are listed in the index buffer determines the triangle's facing direction:
- **Clockwise (CW):** Facing the camera (rendered).
- **Counter-Clockwise (CCW):** Facing away from the camera (culled/discarded).
This optimization (**backface culling**) cuts the number of rendered triangles in half by ignoring the interior faces of closed objects.

### Barycentric Coordinates $(w_0, w_1, w_2)$
Any point $\vec{P}$ inside a triangle defined by vertices $\vec{A}$, $\vec{B}$, and $\vec{C}$ can be uniquely written as:
$$\vec{P} = w_0\vec{A} + w_1\vec{B} + w_2\vec{C}$$
Where:
- $w_0 + w_1 + w_2 = 1.0$
- $w_i \ge 0$ for all $i$ (if the point lies inside the triangle).

```
         A
        / \
       / P \
      /_____\
     B       C
```

These weights represent the relative area of the sub-triangles opposite to each vertex:
- If $w_0 = 1, w_1 = 0, w_2 = 0$, the point is exactly at vertex $\vec{A}$.
- If $w_0 = \frac{1}{3}, w_1 = \frac{1}{3}, w_2 = \frac{1}{3}$, the point is at the triangle's geometric center (centroid).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Three-Way Tug-of-War**
> Imagine three people standing at the corners of a triangle $\vec{A}$, $\vec{B}$, and $\vec{C}$, holding ropes connected to a ring at point $\vec{P}$. 
> 
> The barycentric coordinates $(w_0, w_1, w_2)$ represent the percentage of force each person is pulling with to keep the ring stationary.
> - If Person $\vec{A}$ pulls with $100\%$ force ($w_0 = 1$), the ring moves to $\vec{A}$.
> - If all pull equally ($w_0 = w_1 = w_2 = 0.33$), the ring rests in the center.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Working with Mesh vertices/indices and interpolating values
using UnityEngine;

public class MeshDataDemo : MonoBehaviour
{
    void Start()
    {
        MeshFilter meshFilter = GetComponent<MeshFilter>();
        if (meshFilter == null) return;

        Mesh mesh = meshFilter.sharedMesh;
        Vector3[] vertices = mesh.vertices;
        int[] triangles = mesh.triangles; // Index buffer

        Debug.Log($"Mesh has {vertices.Length} unique vertices.");
        Debug.Log($"Mesh has {triangles.Length / 3} triangles.");

        // Example: Print the vertices of the first triangle
        if (triangles.Length >= 3)
        {
            Vector3 v0 = vertices[triangles[0]];
            Vector3 v1 = vertices[triangles[1]];
            Vector3 v2 = vertices[triangles[2]];
            Debug.Log($"Triangle 0 vertices: {v0}, {v1}, {v2}");
        }
    }

    // Helper: Compute barycentric coordinates for a point P on triangle ABC
    public static Vector3 GetBarycentricWeights(Vector3 p, Vector3 a, Vector3 b, Vector3 c)
    {
        Vector3 v0 = b - a, v1 = c - a, v2 = p - a;
        float d00 = Vector3.Dot(v0, v0);
        float d01 = Vector3.Dot(v0, v1);
        float d11 = Vector3.Dot(v1, v1);
        float d20 = Vector3.Dot(v2, v0);
        float d21 = Vector3.Dot(v2, v1);
        float denom = d00 * d11 - d01 * d01;

        float v = (d11 * d20 - d01 * d21) / denom;
        float w = (d00 * d21 - d01 * d20) / denom;
        float u = 1.0f - v - w;

        return new Vector3(u, v, w); // weights correspond to (a, b, c)
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the sum of the three barycentric coordinates $(w_0, w_1, w_2)$ for any point? :: $1.0$.
- What is winding order, and why does it matter? :: The index sequence direction (clockwise or counter-clockwise) defining the "front" side of a triangle for culling.
- Why do game engines separate vertex positions from the index buffer? :: It avoids duplicate storage for shared vertices, saving memory and allowing fast vertex transformations.
- What does it mean if a point's barycentric weights include a negative number? :: The point lies outside the boundaries of the triangle.
- What is backface culling? :: The process of discarding triangles that face away from the camera, reducing rendering workloads.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Reversing the winding order when generating indices procedurally.
> - **The Fix:** Ensure your index trios are listed consistently (typically Clockwise when looking at the front face of the mesh).
> - **Why:** If the order is reversed, the engine applies backface culling to the *front* face, making the geometry invisible from the outside and visible only from the inside.

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/07_Geometric_Primitives/normal_vectors|Normal Vectors]]
- [[Math/11_Graphics_Math/rendering_pipeline|The Rendering Pipeline]]
