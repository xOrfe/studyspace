---
title: "Matrix Fundamentals: The Grid That Transforms Everything"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Matrix Fundamentals: The Grid That Transforms Everything

> [!abstract] **The Concept in a Nutshell**
> A **matrix** is a rectangular grid of numbers arranged in rows and columns. In game development, matrices are the machinery behind every transformation — rotation, scaling, translation, and projection. Every `Transform` component in Unity, every vertex sent to the GPU, and every camera view is powered by matrix math. Understanding matrices unlocks the entire transformation pipeline that makes 3D rendering possible.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Unity's Transform Component Under the Hood**
> Every time you move, rotate, or scale a GameObject in Unity, the engine updates a **4×4 matrix** called the `localToWorldMatrix`. When you parent a sword to a character's hand bone, the engine **multiplies** the sword's local matrix by the hand's matrix, by the arm's matrix, by the character's matrix — a chain of 4×4 matrices that places the sword exactly in world space. The GPU receives these matrices every frame to transform thousands of vertices in parallel. A single `Transform.position = ...` assignment triggers a full matrix rebuild behind the scenes.

---

## The Blueprint (Formula & Structure)

### Matrix Notation
A matrix $\mathbf{M}$ with $m$ rows and $n$ columns is an $m \times n$ matrix:

$$\mathbf{M}_{3 \times 3} = \begin{pmatrix} m_{11} & m_{12} & m_{13} \\ m_{21} & m_{22} & m_{23} \\ m_{31} & m_{32} & m_{33} \end{pmatrix}$$

Element $m_{ij}$ is at row $i$, column $j$.

### Matrix Dimensions
- **$2 \times 2$:** 2D linear transforms (rotation, scale, shear)
- **$3 \times 3$:** 3D linear transforms (rotation, scale, shear)
- **$4 \times 4$:** Full 3D affine transforms (adds translation) — the standard in game engines

### Types of Matrices
| Type | Description | Example |
|---|---|---|
| **Square** | Rows = Columns ($n \times n$) | All transform matrices |
| **Identity** | 1s on diagonal, 0s elsewhere | "Do nothing" transform |
| **Diagonal** | Non-zero only on diagonal | Uniform/non-uniform scale |
| **Symmetric** | $m_{ij} = m_{ji}$ (mirror across diagonal) | Inertia tensors |
| **Orthogonal** | $\mathbf{M}^T \mathbf{M} = \mathbf{I}$ | Pure rotation matrices |
| **Zero** | All elements are 0 | Collapse to origin |

### The Identity Matrix
The identity matrix $\mathbf{I}$ is the matrix equivalent of multiplying by 1:

$$\mathbf{I}_{3} = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{pmatrix} \qquad \mathbf{M} \cdot \mathbf{I} = \mathbf{I} \cdot \mathbf{M} = \mathbf{M}$$

### Row-Major vs Column-Major Storage

This is one of the most confusing aspects of matrix math in games:

| Convention | Used By | Matrix-Vector Multiply | Memory Layout |
|---|---|---|---|
| **Row-major** | DirectX, HLSL | $\vec{v}' = \vec{v} \cdot \mathbf{M}$ (row vector × matrix) | Row elements contiguous |
| **Column-major** | OpenGL, GLSL, Math textbooks | $\vec{v}' = \mathbf{M} \cdot \vec{v}$ (matrix × column vector) | Column elements contiguous |

**Unity** uses column-major convention internally (matching OpenGL), but the `Matrix4x4` API uses `m[row, col]` indexing for readability.

**Why this matters for GPU:** Shaders need to know the memory layout to correctly access matrix elements. Transposing a matrix swaps between conventions. Sending a row-major matrix to a column-major shader (or vice versa) produces completely wrong transforms.

### Column Vectors as Basis Vectors
In column-major convention, the columns of a 3×3 matrix represent the transformed basis vectors:

$$\mathbf{M} = \begin{pmatrix} | & | & | \\ \vec{x}' & \vec{y}' & \vec{z}' \\ | & | & | \end{pmatrix}$$

- Column 0: Where the X-axis $(1,0,0)$ ends up after transformation
- Column 1: Where the Y-axis $(0,1,0)$ ends up
- Column 2: Where the Z-axis $(0,0,1)$ ends up

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Recipe Card Rack**
> Think of a matrix as a recipe card holder with numbered slots. Each row is a shelf, each column is a slot position. A $3 \times 3$ matrix is a 3-shelf rack with 3 slots each — 9 numbers total. The **identity matrix** is the "default recipe" where nothing changes. When you fill the slots with different numbers, you create a new recipe that stretches, rotates, or skews anything that passes through it. The beauty is that you can **stack recipe cards** (multiply matrices) to combine transformations — first rotate, then scale, then translate — all in one combined recipe.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Exploring matrix fundamentals
using UnityEngine;

public class MatrixFundamentals : MonoBehaviour
{
    void Start()
    {
        // === IDENTITY MATRIX ===
        Matrix4x4 identity = Matrix4x4.identity;
        Debug.Log($"Identity:\n{identity}");
        // 1 0 0 0
        // 0 1 0 0
        // 0 0 1 0
        // 0 0 0 1

        // === ACCESSING THE TRANSFORM MATRIX ===
        Matrix4x4 localToWorld = transform.localToWorldMatrix;
        Debug.Log($"Local-to-World:\n{localToWorld}");

        // === READING BASIS VECTORS (columns in Unity) ===
        Vector3 right   = localToWorld.GetColumn(0); // X-axis
        Vector3 up      = localToWorld.GetColumn(1); // Y-axis
        Vector3 forward = localToWorld.GetColumn(2); // Z-axis
        Vector3 pos     = localToWorld.GetColumn(3); // Translation

        Debug.Log($"Right: {right}, Up: {up}, Forward: {forward}");
        Debug.Log($"Position: {pos}");

        // === MATRIX ELEMENT ACCESS ===
        float m00 = localToWorld[0, 0]; // Row 0, Col 0
        float m12 = localToWorld[1, 2]; // Row 1, Col 2

        // === CHECKING MATRIX DIMENSIONS ===
        // Unity's Matrix4x4 is always 4×4 (16 elements)
        // Individual element access: matrix[row, col]
        for (int row = 0; row < 4; row++)
        {
            string line = "";
            for (int col = 0; col < 4; col++)
                line += $"{localToWorld[row, col]:F2}\t";
            Debug.Log(line);
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is a $4 \times 4$ matrix and why is it used in games? :: A grid of 16 numbers (4 rows × 4 columns) that can represent any affine transformation: rotation + scale + translation. It's the standard in game engines because it encodes all transformations in one structure.
- What does the identity matrix do when you multiply a vector by it? :: Nothing — the vector is unchanged. The identity matrix is the "no-op" transformation, like multiplying by 1.
- What is the difference between row-major and column-major? :: They describe memory layout and multiplication order. Row-major (DirectX): `v' = v × M`. Column-major (OpenGL/Unity): `v' = M × v`. The transposed matrix of one equals the other.
- In Unity's column-major convention, what do the columns of a transform matrix represent? :: Column 0 = transformed X-axis (right), Column 1 = transformed Y-axis (up), Column 2 = transformed Z-axis (forward), Column 3 = translation (position).
- What is a diagonal matrix? :: A square matrix where only the diagonal elements ($m_{ii}$) are non-zero. It represents non-uniform scaling along each axis.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Confusing row-major and column-major conventions when writing shader code or porting between APIs. A DirectX matrix sent to an OpenGL shader without transposing will produce completely wrong results.
> - **The Fix:** Always check which convention your API uses. Unity uses column-major (OpenGL style). When writing custom shaders, use `mul(UNITY_MATRIX_MVP, vertex)` which handles the convention correctly. If you build matrices manually, transpose when crossing API boundaries.
> - **Why:** The same 16 numbers arranged row-major vs column-major represent different transformations. What looks like a translation in one convention could be a shear in the other.

---

## Related Topics
- [[Math/04_Matrices_Transforms/matrix_operations|Matrix Operations]]
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
