---
title: "Cross Product: The Perpendicular Generator"
tags:
  - math
  - level/Lv2
  - category/vectors
---

# Cross Product: The Perpendicular Generator

> [!abstract] **The Concept in a Nutshell**
> The **cross product** takes two vectors and produces a third vector that is **perpendicular** to both inputs. Its magnitude equals the area of the parallelogram formed by the two vectors. In game development, the cross product is essential for computing surface normals, determining left/right turns in navigation, calculating face orientations for meshes, and building coordinate frames.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Mesh Normal Calculation for a Terrain System**
> In *Voxel Lands*, the terrain is built from triangles. For each triangle with vertices $A = (0,0,0)$, $B = (1,0,0)$, $C = (0,0,1)$, we compute two edge vectors: $\vec{AB} = B - A = (1,0,0)$ and $\vec{AC} = C - A = (0,0,1)$. The cross product $\vec{AB} \times \vec{AC} = (0,1,0)$ gives the **face normal** — a vector pointing straight up, telling the lighting system this triangle faces the sky. Without cross products, the renderer wouldn't know which direction any surface faces, and lighting, shadows, and backface culling would all break.

---

## The Blueprint (Formula & Structure)

### Cross Product Formula
For $\vec{a} = (a_x, a_y, a_z)$ and $\vec{b} = (b_x, b_y, b_z)$:

$$\vec{a} \times \vec{b} = \begin{pmatrix} a_y b_z - a_z b_y \\ a_z b_x - a_x b_z \\ a_x b_y - a_y b_x \end{pmatrix}$$

**Memory aid** (determinant form):

$$\vec{a} \times \vec{b} = \begin{vmatrix} \hat{i} & \hat{j} & \hat{k} \\ a_x & a_y & a_z \\ b_x & b_y & b_z \end{vmatrix}$$

### Geometric Interpretation
$$\|\vec{a} \times \vec{b}\| = \|\vec{a}\| \, \|\vec{b}\| \sin\theta$$

- **Direction:** Perpendicular to both $\vec{a}$ and $\vec{b}$ (follows the right-hand rule)
- **Magnitude:** Area of the parallelogram spanned by $\vec{a}$ and $\vec{b}$
- Triangle area = $\frac{1}{2}\|\vec{a} \times \vec{b}\|$

### The Right-Hand Rule
1. Point your right-hand fingers along $\vec{a}$
2. Curl them toward $\vec{b}$ (through the smaller angle)
3. Your thumb points in the direction of $\vec{a} \times \vec{b}$

> **Note:** Unity uses a **left-handed** coordinate system, so you'd use the left-hand rule instead. The cross product formula itself is identical — only the coordinate system's handedness changes the visual result.

### Key Properties
- **Anticommutative:** $\vec{a} \times \vec{b} = -(\vec{b} \times \vec{a})$ — swapping order flips direction!
- **NOT associative:** $\vec{a} \times (\vec{b} \times \vec{c}) \neq (\vec{a} \times \vec{b}) \times \vec{c}$
- **Distributive:** $\vec{a} \times (\vec{b} + \vec{c}) = \vec{a} \times \vec{b} + \vec{a} \times \vec{c}$
- **Parallel vectors:** $\vec{a} \times \vec{b} = \vec{0}$ if $\vec{a} \parallel \vec{b}$ (or either is zero)
- **Self-cross:** $\vec{a} \times \vec{a} = \vec{0}$

### Winding Order and Normals
The **winding order** of triangle vertices (clockwise vs counter-clockwise) determines which way the normal points:
- **Counter-clockwise (CCW):** Normal points "outward" (standard in OpenGL)
- **Clockwise (CW):** Normal points "inward"

Swapping the order of the cross product operands flips the normal — this is how backface culling works.

### 2D Pseudo Cross Product
In 2D, the cross product returns a scalar (the Z-component of the 3D cross product with $z=0$):

$$\vec{a} \times \vec{b} = a_x b_y - a_y b_x$$

- Positive → $\vec{b}$ is to the **left** of $\vec{a}$ (CCW turn)
- Negative → $\vec{b}$ is to the **right** of $\vec{a}$ (CW turn)
- Zero → vectors are parallel (no turn)

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Screwdriver**
> Imagine holding a screwdriver aligned with vector $\vec{a}$. Now turn it toward vector $\vec{b}$. The direction the screw moves (in or out) is the direction of $\vec{a} \times \vec{b}$. The harder you have to twist (greater angle between vectors, longer vectors), the faster the screw moves (larger magnitude). If the two vectors are parallel — no twist possible, screw doesn't move — the cross product is zero. Reverse the twist direction ($\vec{b} \times \vec{a}$), and the screw goes the opposite way. That's anticommutativity!

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Computing mesh normals and determining turn direction
using UnityEngine;

public class CrossProductDemo : MonoBehaviour
{
    void Start()
    {
        // === FACE NORMAL CALCULATION ===
        Vector3 A = new Vector3(0, 0, 0);
        Vector3 B = new Vector3(1, 0, 0);
        Vector3 C = new Vector3(0, 0, 1);

        Vector3 edgeAB = B - A; // (1, 0, 0)
        Vector3 edgeAC = C - A; // (0, 0, 1)

        Vector3 faceNormal = Vector3.Cross(edgeAB, edgeAC).normalized;
        // In Unity (left-handed): (0, 1, 0) — pointing up
        Debug.Log($"Face normal: {faceNormal}");

        // Triangle area = half the cross product magnitude
        float triangleArea = Vector3.Cross(edgeAB, edgeAC).magnitude * 0.5f;
        Debug.Log($"Triangle area: {triangleArea}");
    }

    void Update()
    {
        // === LEFT/RIGHT TURN DETECTION (2D Navigation) ===
        Vector3 forward = transform.forward;
        Vector3 toTarget = (FindAnyObjectByType<Transform>().position
                           - transform.position).normalized;

        Vector3 cross = Vector3.Cross(forward, toTarget);

        if (cross.y > 0.01f)
            Debug.Log("Target is to the LEFT — turn left!");
        else if (cross.y < -0.01f)
            Debug.Log("Target is to the RIGHT — turn right!");
        else
            Debug.Log("Target is straight ahead (or behind).");

        // === BUILDING A COORDINATE FRAME ===
        Vector3 up = Vector3.up;
        Vector3 fwd = transform.forward;
        Vector3 right = Vector3.Cross(up, fwd).normalized;
        Vector3 correctedUp = Vector3.Cross(fwd, right).normalized;
        // Now (right, correctedUp, fwd) form an orthonormal basis
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the cross product $\vec{a} \times \vec{b}$ produce? :: A new vector that is **perpendicular to both** $\vec{a}$ and $\vec{b}$, with magnitude equal to $\|\vec{a}\|\|\vec{b}\|\sin\theta$.
- Is the cross product commutative? :: No! It is **anticommutative**: $\vec{a} \times \vec{b} = -(\vec{b} \times \vec{a})$. Swapping the order reverses the direction.
- What does a cross product of zero mean? :: The two vectors are **parallel** (or one is the zero vector). There is no unique perpendicular direction when vectors lie on the same line.
- How do you determine if a target is to the left or right using the cross product? :: Compute `Vector3.Cross(forward, toTarget)`. If the Y-component is positive, the target is to the left. If negative, it's to the right. (In Unity's left-handed system.)
- How do you calculate the area of a triangle from its vertices? :: Area $= \frac{1}{2} \|\vec{AB} \times \vec{AC}\|$ where $\vec{AB} = B - A$ and $\vec{AC} = C - A$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Assuming cross product order doesn't matter (like dot product). Writing `Cross(B, A)` when you meant `Cross(A, B)` produces a normal pointing the wrong way.
> - **The Fix:** Always be intentional about operand order. For triangle normals, use consistent winding order (e.g., always counter-clockwise vertices). Test by visualizing the resulting normal with `Debug.DrawRay`.
> - **Why:** The cross product is anticommutative — swapping inputs flips the result by 180°. A flipped normal causes inverted lighting, broken backface culling, and inside-out meshes.

---

## Related Topics
- [[Math/03_Vectors/dot_product|Dot Product]]
- [[Math/07_Geometric_Primitives/normal_vectors|Normal Vectors]]
- [[Math/07_Geometric_Primitives/triangles_meshes|Triangles & Meshes]]
