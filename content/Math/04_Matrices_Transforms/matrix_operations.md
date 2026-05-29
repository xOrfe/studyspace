---
title: "Matrix Operations: Chaining Transformations"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Matrix Operations: Chaining Transformations

> [!abstract] **The Concept in a Nutshell**
> Matrix operations let you **combine, scale, and chain** transformations. Matrix multiplication is the engine's way of saying "first rotate, then translate, then project" — all baked into a single matrix. The critical insight is that matrix multiplication is **NOT commutative**: the order you multiply matters enormously. Rotating then translating gives a completely different result than translating then rotating.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Spaceship Docking Sequence**
> In *Stellar Drift*, a spacecraft must dock with a rotating space station. The docking sequence applies transformations in order: (1) **Scale** the ship's local mesh to match the docking port size, (2) **Rotate** the ship to align with the port's orientation, (3) **Translate** the ship to the port's position. The final transformation matrix is $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ (read right-to-left: scale first, rotate second, translate last). If you accidentally compute $\mathbf{S} \cdot \mathbf{R} \cdot \mathbf{T}$ instead, the ship would translate first (moving to the wrong position), then rotate around the origin (swinging wildly), then scale — completely wrong!

---

## The Blueprint (Formula & Structure)

### Matrix Addition
Add corresponding elements (matrices must have the same dimensions):

$$\mathbf{A} + \mathbf{B} = \begin{pmatrix} a_{11}+b_{11} & a_{12}+b_{12} \\ a_{21}+b_{21} & a_{22}+b_{22} \end{pmatrix}$$

- Commutative: $\mathbf{A} + \mathbf{B} = \mathbf{B} + \mathbf{A}$
- Rarely used in game transforms (you multiply, not add, transformations)

### Scalar Multiplication
Multiply every element by a scalar:

$$k \cdot \mathbf{A} = \begin{pmatrix} ka_{11} & ka_{12} \\ ka_{21} & ka_{22} \end{pmatrix}$$

### Matrix-Vector Multiplication
A matrix transforms a vector by multiplying it. In column-major convention:

$$\mathbf{M} \cdot \vec{v} = \begin{pmatrix} m_{11} & m_{12} & m_{13} \\ m_{21} & m_{22} & m_{23} \\ m_{31} & m_{32} & m_{33} \end{pmatrix} \begin{pmatrix} v_x \\ v_y \\ v_z \end{pmatrix} = \begin{pmatrix} m_{11}v_x + m_{12}v_y + m_{13}v_z \\ m_{21}v_x + m_{22}v_y + m_{23}v_z \\ m_{31}v_x + m_{32}v_y + m_{33}v_z \end{pmatrix}$$

Each component of the result is a **dot product** of a matrix row with the vector.

### Matrix-Matrix Multiplication
For $\mathbf{A}$ ($m \times n$) and $\mathbf{B}$ ($n \times p$), the result $\mathbf{C} = \mathbf{A} \cdot \mathbf{B}$ is $m \times p$:

$$c_{ij} = \sum_{k=1}^{n} a_{ik} \cdot b_{kj}$$

Each element is the dot product of row $i$ of $\mathbf{A}$ with column $j$ of $\mathbf{B}$.

**Dimension requirement:** Inner dimensions must match: $(m \times \underline{n}) \cdot (\underline{n} \times p) = (m \times p)$

### $2 \times 2$ Example

$$\begin{pmatrix} a & b \\ c & d \end{pmatrix} \cdot \begin{pmatrix} e & f \\ g & h \end{pmatrix} = \begin{pmatrix} ae+bg & af+bh \\ ce+dg & cf+dh \end{pmatrix}$$

### Critical Properties of Matrix Multiplication
| Property | Status | Implication |
|---|---|---|
| Commutative | ❌ **NO** | $\mathbf{A}\mathbf{B} \neq \mathbf{B}\mathbf{A}$ in general |
| Associative | ✅ Yes | $(\mathbf{A}\mathbf{B})\mathbf{C} = \mathbf{A}(\mathbf{B}\mathbf{C})$ |
| Distributive | ✅ Yes | $\mathbf{A}(\mathbf{B} + \mathbf{C}) = \mathbf{A}\mathbf{B} + \mathbf{A}\mathbf{C}$ |
| Identity | ✅ Yes | $\mathbf{A}\mathbf{I} = \mathbf{I}\mathbf{A} = \mathbf{A}$ |

### Transform Concatenation (The Power of Multiplication)
Chaining transforms = multiplying matrices. In column-major convention (Unity/OpenGL), transforms apply **right-to-left**:

$$\vec{v}' = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S} \cdot \vec{v}$$

Read as: first **S**cale, then **R**otate, then **T**ranslate.

The combined matrix $\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ can be precomputed once and applied to every vertex — this is why matrix multiplication is so powerful for performance.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Instruction Stack**
> Think of each matrix as an instruction card: "rotate 45°," "scale by 2," "move right 5 units." Matrix multiplication is **stacking** these cards into a single combined instruction. The stack reads from bottom to top (right-to-left in the formula): the bottom card executes first. Crucially, "rotate then move" is completely different from "move then rotate" — just like "put on socks, then shoes" versus "put on shoes, then socks." The order of the stack IS the result. Associativity means you can pre-combine any adjacent cards without changing the outcome, but you can never rearrange the stack.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Matrix multiplication and transform chaining
using UnityEngine;

public class MatrixOperations : MonoBehaviour
{
    void Start()
    {
        // === BUILD INDIVIDUAL TRANSFORM MATRICES ===
        // Scale by (2, 2, 2)
        Matrix4x4 S = Matrix4x4.Scale(new Vector3(2f, 2f, 2f));

        // Rotate 45° around Y-axis
        Matrix4x4 R = Matrix4x4.Rotate(Quaternion.Euler(0f, 45f, 0f));

        // Translate by (5, 0, 3)
        Matrix4x4 T = Matrix4x4.Translate(new Vector3(5f, 0f, 3f));

        // === CHAIN THEM: T * R * S (scale first, rotate second, translate last) ===
        Matrix4x4 TRS = T * R * S;

        // === TRANSFORM A POINT ===
        Vector3 localPoint = new Vector3(1f, 0f, 0f);
        Vector3 worldPoint = TRS.MultiplyPoint3x4(localPoint);
        Debug.Log($"Local {localPoint} → World {worldPoint}");

        // === DEMONSTRATE NON-COMMUTATIVITY ===
        Matrix4x4 TR = T * R;
        Matrix4x4 RT = R * T;
        Debug.Log($"T*R == R*T? {TR == RT}"); // FALSE!

        Vector3 testPoint = new Vector3(1f, 0f, 0f);
        Vector3 result_TR = TR.MultiplyPoint3x4(testPoint);
        Vector3 result_RT = RT.MultiplyPoint3x4(testPoint);
        Debug.Log($"T*R result: {result_TR}");
        Debug.Log($"R*T result: {result_RT}"); // Different!

        // === UNITY'S BUILT-IN TRS ===
        Matrix4x4 builtIn = Matrix4x4.TRS(
            new Vector3(5f, 0f, 3f),        // Translation
            Quaternion.Euler(0f, 45f, 0f),   // Rotation
            new Vector3(2f, 2f, 2f)          // Scale
        );
        // This equals T * R * S
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Is matrix multiplication commutative? :: **No!** $\mathbf{A}\mathbf{B} \neq \mathbf{B}\mathbf{A}$ in general. "Rotate then translate" ≠ "translate then rotate." This is the single most important property to remember.
- In column-major convention, what does $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S} \cdot \vec{v}$ mean? :: Apply transformations right-to-left: first **S**cale the vector, then **R**otate, then **T**ranslate. The rightmost matrix acts first.
- What must be true about matrix dimensions for multiplication? :: The inner dimensions must match: $(m \times \underline{n}) \cdot (\underline{n} \times p) = (m \times p)$. A $3 \times 3$ can multiply a $3 \times 1$ vector, producing a $3 \times 1$ result.
- Why is matrix concatenation important for performance? :: Instead of applying 3 separate transforms to each of 10,000 vertices (30,000 operations), you precompute $\mathbf{M} = \mathbf{T}\mathbf{R}\mathbf{S}$ once, then apply the single combined matrix to each vertex (10,000 operations).
- What does each element $c_{ij}$ of the product matrix equal? :: The dot product of row $i$ of the first matrix with column $j$ of the second matrix: $c_{ij} = \sum_k a_{ik} b_{kj}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Applying transforms in the wrong order. Writing `R * T` when you mean "rotate then translate" (in column-major convention, you should write `T * R`).
> - **The Fix:** In Unity (column-major), read multiplication right-to-left. The rightmost operation happens first. Use the TRS mnemonic: $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ — Scale, Rotate, Translate is the standard game engine order.
> - **Why:** "Rotate then translate" means the object rotates in place, then moves. "Translate then rotate" means the object moves first, then rotates around the **origin** (not its own center), causing it to orbit wildly.

---

## Related Topics
- [[Math/04_Matrices_Transforms/matrix_fundamentals|Matrix Fundamentals]]
- [[Math/04_Matrices_Transforms/matrix_inverse_determinant|Inverse & Determinant]]
- [[Math/04_Matrices_Transforms/linear_transformations|Linear Transformations]]
