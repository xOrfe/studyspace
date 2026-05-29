---
title: "Affine Transformations: Adding Translation to the Mix"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Affine Transformations: Adding Translation to the Mix

> [!abstract] **The Concept in a Nutshell**
> An **affine transformation** extends linear transformations by adding **translation** — the ability to move objects, not just rotate, scale, or shear them. Translation can't be expressed as a $3 \times 3$ matrix multiply, so we extend to $4 \times 4$ matrices using homogeneous coordinates. This is why every game engine uses $4 \times 4$ matrices: they pack scaling, rotation, AND translation into one unified matrix that can be multiplied, chained, and inverted just like any other.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Character Controller — Rotate to Face, Then Move**
> In *Knight's Odyssey*, the player character Sir Aldric receives a movement command. The system must: (1) **Scale** his mesh to match the armor's proportions $(1.0, 1.1, 1.0)$, (2) **Rotate** him to face the movement direction (45° around Y), and (3) **Translate** him to his current world position $(12, 0, -8)$. With a $3 \times 3$ matrix, we can handle steps 1 and 2 — but step 3 is impossible! Translation adds a constant offset, not a multiplication. The solution: the TRS matrix, a $4 \times 4$ matrix $\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ that does all three in one operation. This is exactly what Unity's `Transform` component builds every frame.

---

## The Blueprint (Formula & Structure)

### Why 3×3 Matrices Can't Translate

A $3 \times 3$ matrix multiplied by a vector always maps the origin to the origin:

$$\mathbf{M} \cdot \vec{0} = \vec{0}$$

But translation moves the origin! If we want to move a point by $(t_x, t_y, t_z)$:

$$\vec{v}' = \mathbf{M}\vec{v} + \vec{t}$$

This is NOT a matrix multiplication — it's a multiply plus an add. We can't chain these cleanly because $(\mathbf{M}_2(\mathbf{M}_1\vec{v} + \vec{t}_1) + \vec{t}_2)$ doesn't simplify into a single matrix operation.

### The 4×4 Solution: Affine Matrix

By extending to 4D (homogeneous coordinates), we embed translation into the matrix:

$$\mathbf{M}_{4\times4} = \begin{pmatrix} r_{11} & r_{12} & r_{13} & t_x \\ r_{21} & r_{22} & r_{23} & t_y \\ r_{31} & r_{32} & r_{33} & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix}$$

The top-left $3 \times 3$ block handles linear transforms (rotation, scale, shear). The right column holds translation. The bottom row is always $(0, 0, 0, 1)$ for affine transforms.

### Translation Matrix

$$\mathbf{T}(t_x, t_y, t_z) = \begin{pmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix}$$

Multiplying by a point $(x, y, z, 1)$:

$$\mathbf{T} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} x + t_x \\ y + t_y \\ z + t_z \\ 1 \end{pmatrix}$$

### The TRS Composite Matrix

The standard game engine transform order (in column-major, right-to-left):

$$\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$$

1. **S**cale the mesh in local space
2. **R**otate to the desired orientation
3. **T**ranslate to the world position

Expanded as a $4 \times 4$ matrix:

$$\mathbf{TRS} = \begin{pmatrix} s_x r_{11} & s_y r_{12} & s_z r_{13} & t_x \\ s_x r_{21} & s_y r_{22} & s_z r_{23} & t_y \\ s_x r_{31} & s_y r_{32} & s_z r_{33} & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix}$$

Where $r_{ij}$ are the rotation matrix elements and $s_x, s_y, s_z$ are scale factors.

### Affine vs Linear
| Property | Linear Transform | Affine Transform |
|---|---|---|
| Matrix size (3D) | $3 \times 3$ | $4 \times 4$ |
| Origin moves? | ❌ No | ✅ Yes |
| Includes translation? | ❌ No | ✅ Yes |
| Lines stay straight? | ✅ Yes | ✅ Yes |
| Parallel lines stay parallel? | ✅ Yes | ✅ Yes |
| Preserves ratios on lines? | ✅ Yes | ✅ Yes |

### Decomposing a TRS Matrix
Given a composite $4 \times 4$ matrix, you can extract:
- **Translation:** Column 3 (the last column): $(m_{03}, m_{13}, m_{23})$
- **Scale:** Magnitude of each column in the $3 \times 3$ block: $s_x = \|\text{col}_0\|$, $s_y = \|\text{col}_1\|$, $s_z = \|\text{col}_2\|$
- **Rotation:** Divide each column by its scale to get the pure rotation matrix

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Moving Stage**
> Imagine a theater stage where actors are posed using linear transforms — you can rotate and scale them, but the stage itself is bolted to the floor (origin fixed). An **affine transformation** adds wheels to the stage. Now you can pose the actors (linear transforms) AND roll the entire stage to any position in the theater (translation). The $4 \times 4$ matrix is the blueprint that says "here's how the actors are posed, AND here's where the stage is parked." The beauty is that you can photocopy and stack blueprints (multiply matrices) to describe nested stages — a puppet on a table on a stage on a truck — all as one single $4 \times 4$ matrix.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Building and decomposing affine transformation matrices
using UnityEngine;

public class AffineTransformations : MonoBehaviour
{
    void Start()
    {
        // === BUILD A TRS MATRIX MANUALLY ===
        Vector3 position = new Vector3(12f, 0f, -8f);
        Quaternion rotation = Quaternion.Euler(0f, 45f, 0f);
        Vector3 scale = new Vector3(1f, 1.1f, 1f);

        // Unity's built-in TRS constructor
        Matrix4x4 trs = Matrix4x4.TRS(position, rotation, scale);
        Debug.Log($"TRS Matrix:\n{trs}");

        // === TRANSFORM A POINT ===
        Vector3 localVertex = new Vector3(1f, 0f, 0f);
        Vector3 worldVertex = trs.MultiplyPoint3x4(localVertex);
        Debug.Log($"Local {localVertex} → World {worldVertex}");

        // === DECOMPOSE THE MATRIX ===
        // Extract translation (column 3)
        Vector3 extractedPos = trs.GetColumn(3);

        // Extract scale (magnitude of each column)
        float sx = trs.GetColumn(0).magnitude;
        float sy = trs.GetColumn(1).magnitude;
        float sz = trs.GetColumn(2).magnitude;
        Vector3 extractedScale = new Vector3(sx, sy, sz);

        // Extract rotation (divide columns by scale)
        Matrix4x4 rotMatrix = trs;
        rotMatrix.SetColumn(0, trs.GetColumn(0) / sx);
        rotMatrix.SetColumn(1, trs.GetColumn(1) / sy);
        rotMatrix.SetColumn(2, trs.GetColumn(2) / sz);
        rotMatrix.SetColumn(3, new Vector4(0, 0, 0, 1));
        Quaternion extractedRot = rotMatrix.rotation;

        Debug.Log($"Extracted — Pos: {extractedPos}, " +
                  $"Rot: {extractedRot.eulerAngles}, Scale: {extractedScale}");

        // === WHY 3×3 CAN'T TRANSLATE ===
        Matrix4x4 linearOnly = Matrix4x4.Rotate(rotation) *
                                Matrix4x4.Scale(scale);
        Vector3 result = linearOnly.MultiplyPoint3x4(Vector3.zero);
        Debug.Log($"Linear transform of origin: {result}");
        // Always (0,0,0) — can't move the origin with linear transforms!
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Why can't a 3×3 matrix represent translation? :: Because $\mathbf{M} \cdot \vec{0} = \vec{0}$ — a linear transformation always maps the origin to the origin. Translation needs to move the origin, which requires extending to 4×4 with homogeneous coordinates.
- What is the standard transform order in a TRS matrix? :: $\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$. Read right-to-left (column-major): **S**cale first (in local space), then **R**otate, then **T**ranslate to world position.
- Where is the translation stored in a 4×4 matrix? :: In the last column (column-major): elements $(m_{03}, m_{13}, m_{23})$, or equivalently `matrix.GetColumn(3)` in Unity.
- How do you extract the scale from a TRS matrix? :: Take the magnitude (length) of each of the first three columns: $s_x = \|\text{col}_0\|$, $s_y = \|\text{col}_1\|$, $s_z = \|\text{col}_2\|$.
- What's the difference between affine and linear transforms? :: Affine = linear + translation. Both keep lines straight and parallel lines parallel. The key difference is that affine transforms CAN move the origin (translate), while linear transforms cannot.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Applying transforms in the wrong TRS order. A common error is building $\mathbf{S} \cdot \mathbf{R} \cdot \mathbf{T}$ instead of $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$, which translates first (in local space), then rotates around the origin (causing an orbital motion), then scales.
> - **The Fix:** Always use TRS order: $\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$. In Unity, use `Matrix4x4.TRS(pos, rot, scale)` which handles the order correctly. Remember: "the rightmost transform is applied first."
> - **Why:** Translating before rotating moves the object away from the origin, so the subsequent rotation swings it in an arc around the origin instead of spinning in place. The TRS order ensures objects are built up in local space (scale → rotate) then placed in the world (translate).

---

## Related Topics
- [[Math/04_Matrices_Transforms/linear_transformations|Linear Transformations]]
- [[Math/04_Matrices_Transforms/homogeneous_coordinates|Homogeneous Coordinates]]
- [[Math/05_Coordinate_Spaces/object_world_space|Object & World Space]]
