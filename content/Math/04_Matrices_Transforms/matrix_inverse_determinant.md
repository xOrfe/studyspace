---
title: "Inverse & Determinant: Undoing Transformations"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Inverse & Determinant: Undoing Transformations

> [!abstract] **The Concept in a Nutshell**
> The **inverse** of a matrix undoes its transformation — if $\mathbf{M}$ rotates an object 30° clockwise, $\mathbf{M}^{-1}$ rotates it 30° counter-clockwise. The **determinant** is a single number that tells you whether a matrix is invertible and how it affects area/volume. Together, these concepts are essential for converting between coordinate spaces, reversing transforms, and detecting degenerate (collapsed) geometry.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: World-to-Local Space Conversion for a Targeting System**
> In *Mech Assault*, the player's mech has a targeting computer that needs to determine if an enemy is within its local attack cone. The enemy is at world position $\vec{P}_w = (50, 10, 30)$. The mech's localToWorldMatrix $\mathbf{M}$ transforms from local to world space. To get the enemy's position in the **mech's local space**, we need the inverse: $\vec{P}_{\text{local}} = \mathbf{M}^{-1} \cdot \vec{P}_w$. Now the targeting system can simply check if `localPos.z > 0` (in front) and `localPos.x` and `localPos.y` are within bounds. Without the inverse matrix, we'd have no clean way to reverse the chain of transforms from world back to local space.

---

## The Blueprint (Formula & Structure)

### Inverse Matrix Definition
The inverse $\mathbf{M}^{-1}$ satisfies:

$$\mathbf{M} \cdot \mathbf{M}^{-1} = \mathbf{M}^{-1} \cdot \mathbf{M} = \mathbf{I}$$

Applying $\mathbf{M}$ then $\mathbf{M}^{-1}$ (or vice versa) returns to the original state.

### 2×2 Inverse Formula
For $\mathbf{A} = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$:

$$\mathbf{A}^{-1} = \frac{1}{ad - bc}\begin{pmatrix} d & -b \\ -c & a \end{pmatrix}$$

The value $ad - bc$ is the **determinant**. If it's zero, no inverse exists.

### Determinant
The determinant $\det(\mathbf{M})$ encodes how the transformation scales area (2D) or volume (3D).

**2×2 Determinant:**
$$\det\begin{pmatrix} a & b \\ c & d \end{pmatrix} = ad - bc$$

**3×3 Determinant (cofactor expansion along row 1):**
$$\det\begin{pmatrix} a & b & c \\ d & e & f \\ g & h & i \end{pmatrix} = a(ei - fh) - b(di - fg) + c(dh - eg)$$

### What the Determinant Tells You
| Determinant Value | Meaning |
|---|---|
| $\det = 1$ | Volume preserved, orientation preserved (pure rotation) |
| $\det = -1$ | Volume preserved, orientation **flipped** (reflection) |
| $\|\det\| > 1$ | Volume expanded (scaling up) |
| $0 < \|\det\| < 1$ | Volume shrunk (scaling down) |
| $\det = 0$ | Volume collapsed to 0 — **singular, not invertible!** |

### 3×3 Inverse Formula
$$\mathbf{A}^{-1} = \frac{1}{\det(\mathbf{A})} \text{adj}(\mathbf{A})$$

Where $\text{adj}(\mathbf{A})$ is the **adjugate** (transpose of the cofactor matrix). This is computationally expensive for large matrices — in practice, game engines use optimized algorithms.

### Orthogonal Matrix Shortcut
A matrix is **orthogonal** if its columns (and rows) are mutually perpendicular unit vectors. Pure rotation matrices are orthogonal. For orthogonal matrices:

$$\mathbf{M}^{-1} = \mathbf{M}^T \quad \text{(inverse = transpose!)}$$

This is massively cheaper to compute — just swap rows and columns. No determinant calculation needed!

### Singular Matrices (Non-Invertible)
A matrix is **singular** (non-invertible) when $\det(\mathbf{M}) = 0$. This happens when:
- A scale factor is 0 (collapsed dimension)
- Two or more columns are linearly dependent (parallel)
- The matrix maps 3D space to a plane, line, or point

### Properties of Inverse
- $(\mathbf{A}\mathbf{B})^{-1} = \mathbf{B}^{-1}\mathbf{A}^{-1}$ (reverse order!)
- $(\mathbf{A}^{-1})^{-1} = \mathbf{A}$
- $(\mathbf{A}^T)^{-1} = (\mathbf{A}^{-1})^T$
- $\det(\mathbf{A}^{-1}) = 1 / \det(\mathbf{A})$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Undo Button**
> The inverse matrix is the **Ctrl+Z** of linear algebra. If a matrix transforms your character from local space into world space, the inverse transforms them back. If a matrix rotates, scales, and translates an object, the inverse un-translates, un-scales, and un-rotates (in reverse order — like undoing a sequence of actions). The determinant is like a "health check" for the transformation: if it's zero, the transformation has crushed space flat (like compressing a 3D object into a 2D shadow), and there's no way to undo that — the information is lost forever.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Using inverse matrices for space conversion
using UnityEngine;

public class MatrixInverseDemo : MonoBehaviour
{
    public Transform targetEnemy;

    void Update()
    {
        // === WORLD-TO-LOCAL CONVERSION ===
        // Get the mech's local-to-world matrix
        Matrix4x4 localToWorld = transform.localToWorldMatrix;

        // Invert it to get world-to-local
        Matrix4x4 worldToLocal = localToWorld.inverse;

        // Transform enemy position from world to mech's local space
        Vector3 enemyWorld = targetEnemy.position;
        Vector3 enemyLocal = worldToLocal.MultiplyPoint3x4(enemyWorld);

        // Now we can check local-space conditions easily
        bool isInFront = enemyLocal.z > 0f;
        bool isAbove   = enemyLocal.y > 0f;
        Debug.Log($"Enemy in local space: {enemyLocal}, In front: {isInFront}");

        // === DETERMINANT CHECK ===
        float det = localToWorld.determinant;
        Debug.Log($"Determinant: {det:F4}");

        if (Mathf.Approximately(det, 0f))
            Debug.LogWarning("Singular matrix! Transform is degenerate.");

        // === ORTHOGONAL SHORTCUT: Rotation-only inverse = transpose ===
        Matrix4x4 rotOnly = Matrix4x4.Rotate(transform.rotation);
        Matrix4x4 rotInverse = rotOnly.transpose; // Cheap! No full inverse needed.

        // Verify: rotOnly * rotInverse ≈ identity
        Matrix4x4 shouldBeIdentity = rotOnly * rotInverse;
        Debug.Log($"Is identity? m00={shouldBeIdentity[0,0]:F3}, " +
                  $"m01={shouldBeIdentity[0,1]:F3}");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the inverse of a matrix do? :: It **undoes** the transformation. If $\mathbf{M}$ transforms from local to world space, $\mathbf{M}^{-1}$ transforms from world back to local space. $\mathbf{M}\mathbf{M}^{-1} = \mathbf{I}$.
- When is a matrix NOT invertible? :: When its determinant is zero ($\det(\mathbf{M}) = 0$). This means the transformation collapses a dimension (e.g., scaling by 0 on one axis), and the lost information cannot be recovered.
- What is the shortcut for inverting an orthogonal matrix? :: $\mathbf{M}^{-1} = \mathbf{M}^T$ (the inverse equals the transpose). Pure rotation matrices are orthogonal, so this is a huge performance win.
- What does a negative determinant indicate? :: The transformation **flips orientation** — it includes a reflection. A mirror or odd number of axis flips produces a negative determinant.
- What is the determinant of the 2×2 matrix $\begin{pmatrix} 3 & 1 \\ 2 & 4 \end{pmatrix}$? :: $\det = 3 \cdot 4 - 1 \cdot 2 = 12 - 2 = 10$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Computing a full matrix inverse for rotation-only matrices. The general inverse is expensive ($O(n^3)$), but rotation matrices are orthogonal.
> - **The Fix:** If the matrix is a pure rotation (no scale, no shear), use the **transpose** instead: `Matrix4x4 inv = rotMatrix.transpose;`. This is essentially free.
> - **Why:** For a pure rotation, $\mathbf{R}^T = \mathbf{R}^{-1}$ because the columns are orthonormal. Computing the full inverse with cofactors and determinants wastes cycles and introduces floating-point error.

---

## Related Topics
- [[Math/04_Matrices_Transforms/matrix_operations|Matrix Operations]]
- [[Math/04_Matrices_Transforms/linear_transformations|Linear Transformations]]
- [[Math/05_Coordinate_Spaces/object_world_space|Object & World Space]]
