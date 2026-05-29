---
title: "Eigenvalues & Eigenvectors: Principal Axis alignment, PCA, and OBBs"
tags:
  - math
  - level/Lv4
  - category/advanced_topics
---

# Eigenvalues & Eigenvectors: Principal Axis alignment, PCA, and OBBs

> [!abstract] **The Concept in a Nutshell**
> When a matrix multiplies a vector, it typically rotates and scales it. However, for any square matrix, there exist special directions called **eigenvectors** that are *not rotated* by the transformation — they are only scaled. The factor by which the eigenvector is stretched or shrunk is its corresponding **eigenvalue**. In game development, eigenvalues and eigenvectors are the core components of **Principal Component Analysis (PCA)**, which is used to calculate tight-fitting Oriented Bounding Boxes (OBBs) around complex meshes and analyze character motion data.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Generating a Tight-Fitting Oriented Bounding Box (OBB)**
> You have a 3D model of a long, diagonal sword. If you wrap it in an Axis-Aligned Bounding Box (AABB), the box will be a massive, empty cube because the sword is rotated at $45^\circ$. This creates a lot of false positive collisions.
>
> To fit a tight OBB, you must find the sword's "principal directions" of distribution.
> 1. Compute the **covariance matrix** of the sword's vertex coordinates. This matrix represents how the vertices spread out in space.
> 2. Calculate the **eigenvectors** of this covariance matrix. 
>    - The eigenvector with the largest eigenvalue is the axis along which the vertices spread out the most (the blade of the sword).
>    - The second eigenvector points across the width of the guard.
>    - The third points across the thickness.
> 3. These three eigenvectors form the orthogonal axes of your OBB, fitting the sword like a custom sheath.

---

## The Blueprint (Formula & Structure)

### The Eigenvalue Equation
For a square matrix $M$ and a non-zero vector $\vec{v}$:
$$M \vec{v} = \lambda \vec{v}$$

Where:
- $M$: The 3x3 or NxN transformation matrix.
- $\vec{v}$: The **eigenvector**.
- $\lambda$: The **eigenvalue** (a scalar factor).

### Finding Eigenvalues (The Characteristic Equation)
To solve for $\lambda$, we rewrite the equation as:
$$(M - \lambda I) \vec{v} = \vec{0}$$
Since $\vec{v}$ must be non-zero, the matrix $(M - \lambda I)$ must be singular (non-invertible), meaning its determinant is zero:
$$\det(M - \lambda I) = 0$$
This determinant expands into a polynomial (the **characteristic polynomial**). The roots of this polynomial are the eigenvalues of $M$.

### Principal Component Analysis (PCA) for OBBs
PCA is an algorithm that finds the directions of maximum variance in a dataset:
1. Shift the vertex positions so their mean is at the origin $(0,0,0)$.
2. Calculate the 3x3 covariance matrix $C$:
   $$C_{jk} = \frac{1}{N} \sum_{i=1}^{N} x_{ij} x_{ik}$$
3. Diagonalize the covariance matrix to find its eigenvectors $\vec{v}_1, \vec{v}_2, \vec{v}_3$.
4. Use these eigenvectors as the rotation matrix columns to orient the OBB.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Stretched Painting**
> Imagine printing a painting on a sheet of rubber and stretching it: you pull it twice as hard horizontally and three times as hard vertically.
> - Almost every brushstroke in the painting shifts diagonally (rotates and stretches).
> - However, a line painted exactly horizontal stays horizontal — it only stretches by $2\times$ (eigenvalue $2$, horizontal eigenvector).
> - A line painted exactly vertical stays vertical — it only stretches by $3\times$ (eigenvalue $3$, vertical eigenvector).
> 
> Eigenvectors identify the "pure axes" of a transformation or shape distribution.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Concept outline of PCA axis extraction for OBB calculation
using UnityEngine;

public class OBBGeneratorPCA : MonoBehaviour
{
    public MeshFilter meshFilter;

    public void CalculateOBBAxes(out Vector3 axisX, out Vector3 axisY, out Vector3 axisZ)
    {
        Vector3[] vertices = meshFilter.sharedMesh.vertices;
        int n = vertices.Length;

        // 1. Calculate Center of Mass (Mean Position)
        Vector3 mean = Vector3.zero;
        for (int i = 0; i < n; i++) mean += vertices[i];
        mean /= n;

        // 2. Compute 3x3 Covariance Matrix
        float cxx = 0, cxy = 0, cxz = 0;
        float cyy = 0, cyz = 0, czz = 0;

        for (int i = 0; i < n; i++)
        {
            Vector3 diff = vertices[i] - mean;
            cxx += diff.x * diff.x;
            cxy += diff.x * diff.y;
            cxz += diff.x * diff.z;
            cyy += diff.y * diff.y;
            cyz += diff.y * diff.z;
            czz += diff.z * diff.z;
        }

        // Covariance matrix is symmetric (cxy = cyx, cxz = czx, cyz = czy)
        float[,] covMatrix = new float[3, 3] {
            { cxx / n, cxy / n, cxz / n },
            { cxy / n, cyy / n, cyz / n },
            { cxz / n, cyz / n, czz / n }
        };

        // 3. Extract Eigenvalues & Eigenvectors
        // In a production engine, this uses a numerical solver like Jacobi Rotation or QR decomposition.
        SolveEigenSystem(covMatrix, out axisX, out axisY, out axisZ);
    }

    // Dummy representation of numerical diagonalization
    private void SolveEigenSystem(float[,] matrix, out Vector3 ev1, out Vector3 ev2, out Vector3 ev3)
    {
        // Numerical approximation algorithms return the three orthogonal eigenvectors.
        // Here we default to local basis coordinates as a fallback.
        ev1 = Vector3.right;
        ev2 = Vector3.up;
        ev3 = Vector3.forward;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Define eigenvectors and eigenvalues mathematically. :: $M\vec{v} = \lambda\vec{v}$, where $M$ is a square matrix, $\vec{v}$ is the non-zero eigenvector, and $\lambda$ is the eigenvalue scalar.
- What equation is solved to find the eigenvalues of a matrix $M$? :: The characteristic equation: $\det(M - \lambda I) = 0$.
- In game dev, how are eigenvalues and eigenvectors used in bounding box generation? :: PCA extracts the eigenvectors of a mesh's covariance matrix, utilizing them as the rotation axes to construct a tightly fitted Oriented Bounding Box (OBB).
- Why is a covariance matrix symmetric? :: Because the covariance of X and Y is identical to the covariance of Y and X ($C_{xy} = C_{yx}$).
- What does the eigenvector with the largest eigenvalue represent in a dataset? :: The direction of maximum variance (the principal axis of distribution).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Neglecting to subtract the center of mass (mean position) from vertex positions before computing the covariance matrix.
> - **The Fix:** Always center the dataset at $(0,0,0)$ first.
> - **Why:** If the data is offset from the origin, the covariance calculation will reflect the offset distance instead of the shape's distribution, generating incorrect eigenvectors that skew the OBB axes.

---

## Related Topics
- [[Math/04_Matrices_Transforms/matrix_inverse_determinant|Inverse & Determinant]]
- [[Math/07_Geometric_Primitives/bounding_volumes|Bounding Volumes]]
- [[Math/10_Physics_Math/rigid_body_dynamics|Rigid Body Dynamics]]
