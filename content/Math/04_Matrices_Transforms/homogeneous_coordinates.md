---
title: "Homogeneous Coordinates: The Fourth Dimension of Game Math"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Homogeneous Coordinates: The Fourth Dimension of Game Math

> [!abstract] **The Concept in a Nutshell**
> **Homogeneous coordinates** add a fourth component $w$ to 3D vectors, turning $(x, y, z)$ into $(x, y, z, w)$. This seemingly simple extension is what makes $4 \times 4$ matrices work: $w = 1$ marks a **point** (affected by translation), $w = 0$ marks a **direction** (immune to translation), and values of $w \neq 1$ enable **perspective projection** through the perspective divide. Every vertex the GPU processes uses homogeneous coordinates — it's the mathematical foundation of the entire rendering pipeline.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The Camera's Perspective Projection**
> In *Void Runner*, the player flies through a tunnel at high speed. Objects far away must appear smaller (perspective). The GPU receives each vertex as $(x, y, z, 1)$, multiplies by the view-projection matrix, and gets back $(x', y', z', w')$ where $w' \neq 1$. The **perspective divide** — dividing by $w'$ — produces $(x'/w', y'/w', z'/w', 1)$, which maps to screen coordinates. Far-away objects have large $w'$, so their screen positions compress toward the center — creating the illusion of depth. Meanwhile, when the engine computes lighting, it uses surface normals as $(n_x, n_y, n_z, 0)$ — the $w=0$ ensures normals aren't affected by the camera's translation, only its rotation.

---

## The Blueprint (Formula & Structure)

### The w Component
A 3D vector $(x, y, z)$ is extended to homogeneous coordinates as $(x, y, z, w)$:

| w Value | Represents | Translation Effect | Use Case |
|---|---|---|---|
| $w = 1$ | **Point** (position) | ✅ Affected | Vertex positions, world coordinates |
| $w = 0$ | **Direction** (vector) | ❌ Immune | Normals, light directions, velocities |
| $w \neq 0,1$ | Projective point | Divides out | Perspective projection intermediate |

### Points vs Directions — The Key Distinction
**Point** $(x, y, z, 1)$ multiplied by a translation matrix:

$$\begin{pmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix} \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} x + t_x \\ y + t_y \\ z + t_z \\ 1 \end{pmatrix}$$

Translation is applied! ✅

**Direction** $(x, y, z, 0)$ multiplied by the same translation matrix:

$$\begin{pmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix} \begin{pmatrix} x \\ y \\ z \\ 0 \end{pmatrix} = \begin{pmatrix} x \\ y \\ z \\ 0 \end{pmatrix}$$

Translation has no effect! ✅ Directions shouldn't change when you move the object — "up" is still "up" regardless of position.

### The Perspective Divide
After the projection matrix multiplies a vertex, the result has $w \neq 1$:

$$(x', y', z', w') \xrightarrow{\text{perspective divide}} \left(\frac{x'}{w'}, \frac{y'}{w'}, \frac{z'}{w'}, 1\right)$$

This division by $w'$ is what creates perspective foreshortening:
- Near objects: small $w'$ → coordinates stay large → appear big on screen
- Far objects: large $w'$ → coordinates shrink → appear small on screen

### Why GPUs Use 4×4 Matrices
The full vertex transformation pipeline:

$$\vec{v}_{\text{clip}} = \mathbf{P} \cdot \mathbf{V} \cdot \mathbf{M} \cdot \vec{v}_{\text{local}}$$

| Matrix | Name | Size | Purpose |
|---|---|---|---|
| $\mathbf{M}$ | Model (World) | $4 \times 4$ | Local space → World space |
| $\mathbf{V}$ | View | $4 \times 4$ | World space → Camera space |
| $\mathbf{P}$ | Projection | $4 \times 4$ | Camera space → Clip space |

All three are $4 \times 4$ matrices that chain via multiplication. The GPU multiplies them once ($\mathbf{MVP} = \mathbf{P} \cdot \mathbf{V} \cdot \mathbf{M}$) and then applies the single result to every vertex — one matrix multiply per vertex instead of three.

### Equivalence Class
In homogeneous coordinates, any scalar multiple represents the same point:

$$(x, y, z, w) \equiv (kx, ky, kz, kw) \quad \text{for } k \neq 0$$

So $(2, 4, 6, 2)$ and $(1, 2, 3, 1)$ represent the same 3D point $(1, 2, 3)$. You recover the 3D point by dividing by $w$.

### Converting Between Representations
**3D → Homogeneous:**
- Point: $(x, y, z) \rightarrow (x, y, z, 1)$
- Direction: $(x, y, z) \rightarrow (x, y, z, 0)$

**Homogeneous → 3D (when $w \neq 0$):**
$$(x, y, z, w) \rightarrow \left(\frac{x}{w}, \frac{y}{w}, \frac{z}{w}\right)$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Postal Address vs the Compass Bearing**
> Think of $w = 1$ (a point) as a **postal address** — "123 Main Street." If you move the entire city 5 miles east (translation), the address becomes "123 Main Street, 5 miles east." The location changes with the coordinate system.
>
> Think of $w = 0$ (a direction) as a **compass bearing** — "north." If you move the city 5 miles east, "north" is still "north." Directions don't care about where you are, only which way you're pointing.
>
> The fourth coordinate $w$ is the magic switch that tells the matrix "treat this as an address (translate it) or a compass bearing (leave it alone)."

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Homogeneous coordinates in practice
using UnityEngine;

public class HomogeneousCoordinates : MonoBehaviour
{
    public Transform target;

    void Start()
    {
        // === POINTS VS DIRECTIONS ===
        Matrix4x4 worldMatrix = transform.localToWorldMatrix;

        // POINT (w=1): Position is affected by translation
        Vector3 localPos = new Vector3(0f, 1f, 0f);
        Vector3 worldPos = worldMatrix.MultiplyPoint3x4(localPos);
        // MultiplyPoint3x4 treats input as (x, y, z, 1) — a POINT

        // DIRECTION (w=0): Normal is NOT affected by translation
        Vector3 localNormal = Vector3.up;
        Vector3 worldNormal = worldMatrix.MultiplyVector(localNormal);
        // MultiplyVector treats input as (x, y, z, 0) — a DIRECTION

        Debug.Log($"World position: {worldPos}");
        Debug.Log($"World normal: {worldNormal}");

        // === MANUAL HOMOGENEOUS COORDINATES ===
        Vector4 pointH = new Vector4(1f, 2f, 3f, 1f);   // Point
        Vector4 dirH   = new Vector4(0f, 1f, 0f, 0f);    // Direction

        // Transform both through the same matrix
        Vector4 transformedPoint = worldMatrix * pointH;
        Vector4 transformedDir   = worldMatrix * dirH;

        Debug.Log($"Point after transform: {transformedPoint}");
        Debug.Log($"Dir after transform: {transformedDir}");
        // Notice: direction's xyz changed (rotated) but w stayed 0

        // === PERSPECTIVE DIVIDE SIMULATION ===
        // After projection, a vertex might be (10, 5, 3, 2)
        Vector4 clipSpace = new Vector4(10f, 5f, 3f, 2f);
        Vector3 ndc = new Vector3(
            clipSpace.x / clipSpace.w,  // 5
            clipSpace.y / clipSpace.w,  // 2.5
            clipSpace.z / clipSpace.w   // 1.5
        );
        Debug.Log($"After perspective divide: {ndc}");
    }

    void Update()
    {
        // === WHY THIS MATTERS: Correct normal transformation ===
        // When transforming normals, ALWAYS use MultiplyVector (w=0)
        // NEVER use MultiplyPoint (w=1) — that would translate the normal,
        // which is meaningless and produces wrong lighting
        Vector3 surfaceNormal = Vector3.up;
        Vector3 correctWorldNormal = transform.localToWorldMatrix
                                             .MultiplyVector(surfaceNormal);
        Debug.DrawRay(transform.position, correctWorldNormal, Color.green);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does $w = 1$ represent in homogeneous coordinates? :: A **point** (position in space). It IS affected by translation. Vertex positions use $w = 1$.
- What does $w = 0$ represent in homogeneous coordinates? :: A **direction** (vector). It is NOT affected by translation, only rotation/scale. Normals, light directions, and velocity vectors use $w = 0$.
- What is the perspective divide? :: Dividing the clip-space coordinates $(x, y, z, w)$ by $w$ to get normalized device coordinates: $(x/w, y/w, z/w)$. This creates perspective foreshortening — farther objects appear smaller.
- In Unity, what's the difference between `MultiplyPoint3x4` and `MultiplyVector`? :: `MultiplyPoint3x4` treats the input as $w = 1$ (a point — affected by translation). `MultiplyVector` treats it as $w = 0$ (a direction — immune to translation).
- Why do GPUs use 4×4 matrices instead of 3×3? :: To handle translation alongside rotation and scale in a single matrix multiplication. The 4th dimension (homogeneous coordinates) enables the full affine transform pipeline and perspective projection.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `MultiplyPoint3x4` (w=1) to transform normals and directions. This applies the matrix's translation to the normal, producing a completely wrong direction that depends on the object's position.
> - **The Fix:** Always use `MultiplyVector` (w=0) for normals, light directions, and any pure direction. Use `MultiplyPoint3x4` (or `MultiplyPoint`) only for positions.
> - **Why:** A surface normal of $(0, 1, 0)$ means "pointing up." If the object is at position $(100, 50, 200)$, using `MultiplyPoint` would turn the normal into $(100, 51, 200)$ — completely meaningless as a direction. The $w = 0$ trick ensures translation is ignored.

---

## Related Topics
- [[Math/04_Matrices_Transforms/affine_transformations|Affine Transformations]]
- [[Math/05_Coordinate_Spaces/perspective_projection|Perspective Projection]]
- [[Math/05_Coordinate_Spaces/view_projection_space|View & Projection Space]]
