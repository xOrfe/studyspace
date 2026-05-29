---
title: "Linear Transformations: Reshaping Space with Matrices"
tags:
  - math
  - level/Lv2
  - category/matrices_transforms
---

# Linear Transformations: Reshaping Space with Matrices

> [!abstract] **The Concept in a Nutshell**
> A **linear transformation** is a function that maps vectors to vectors while preserving addition and scalar multiplication — geometrically, it keeps the origin fixed, lines remain lines, and parallel lines stay parallel. Scaling, rotation, reflection, and shear are all linear transformations that can be represented as matrix multiplications. Understanding them means understanding how game engines manipulate every vertex, normal, and direction in your scene.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Juice Effects in a Platformer**
> In *Bounce Kingdom*, when the slime character Blobbert lands after a jump, the game applies a **squash** effect: scale $(1.3, 0.7, 1.3)$ — wider and shorter. During the jump's peak, a **stretch** effect applies: $(0.8, 1.4, 0.8)$ — taller and thinner. When Blobbert enters a mirror dimension, a **reflection** matrix flips the X-axis, making everything appear reversed. Wind blowing through a forest applies a subtle **shear** to tree meshes, tilting them diagonally. Each of these effects is a linear transformation encoded in a matrix that modifies the mesh vertices before rendering.

---

## The Blueprint (Formula & Structure)

### What Makes a Transformation "Linear"?
A transformation $T$ is linear if:
1. **Additivity:** $T(\vec{u} + \vec{v}) = T(\vec{u}) + T(\vec{v})$
2. **Homogeneity:** $T(k\vec{v}) = kT(\vec{v})$

Consequence: $T(\vec{0}) = \vec{0}$ — the origin never moves. This is why translation is NOT linear.

### Scaling Matrix
Non-uniform scaling along each axis:

$$\mathbf{S} = \begin{pmatrix} s_x & 0 & 0 \\ 0 & s_y & 0 \\ 0 & 0 & s_z \end{pmatrix}$$

- $s_x = s_y = s_z$: **Uniform** scaling (proportional)
- Different values: **Non-uniform** scaling (stretching/squashing)
- $s = -1$ on one axis: **Reflection** across that axis's plane

### 2D Rotation Matrix
Rotate by angle $\theta$ counter-clockwise:

$$\mathbf{R}(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$$

### 3D Rotation Matrices (Around Principal Axes)
**Around X-axis:**
$$\mathbf{R}_x(\theta) = \begin{pmatrix} 1 & 0 & 0 \\ 0 & \cos\theta & -\sin\theta \\ 0 & \sin\theta & \cos\theta \end{pmatrix}$$

**Around Y-axis:**
$$\mathbf{R}_y(\theta) = \begin{pmatrix} \cos\theta & 0 & \sin\theta \\ 0 & 1 & 0 \\ -\sin\theta & 0 & \cos\theta \end{pmatrix}$$

**Around Z-axis:**
$$\mathbf{R}_z(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta & 0 \\ \sin\theta & \cos\theta & 0 \\ 0 & 0 & 1 \end{pmatrix}$$

### Reflection Matrix
Reflect across a plane defined by its normal. For reflection across the XZ-plane (flip Y):

$$\mathbf{F}_y = \begin{pmatrix} 1 & 0 & 0 \\ 0 & -1 & 0 \\ 0 & 0 & 1 \end{pmatrix}$$

General reflection across a plane with unit normal $\hat{n} = (n_x, n_y, n_z)$:

$$\mathbf{F} = \mathbf{I} - 2\hat{n}\hat{n}^T$$

### Shear Matrix
Shear along X proportional to Y (skewing/tilting):

$$\mathbf{H}_{xy} = \begin{pmatrix} 1 & s & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{pmatrix}$$

Where $s$ controls the shear amount. The X-coordinate shifts by $s \cdot y$.

### Combining Transforms
Multiple linear transforms combine via multiplication. The standard order for a single object:

$$\mathbf{M} = \mathbf{R} \cdot \mathbf{S}$$

(Scale first, then rotate — remember right-to-left in column-major)

Adding more transforms:
$$\mathbf{M} = \mathbf{R}_2 \cdot \mathbf{R}_1 \cdot \mathbf{S}$$

### Transform Order Matters!
$$\mathbf{R} \cdot \mathbf{S} \neq \mathbf{S} \cdot \mathbf{R} \quad \text{(in general)}$$

- $\mathbf{R} \cdot \mathbf{S}$: Scale in local space, then rotate → axes stay aligned during scaling
- $\mathbf{S} \cdot \mathbf{R}$: Rotate first, then scale along world axes → can produce shear if non-uniform

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Rubber Sheet**
> Imagine the entire game world printed on a rubber sheet pinned at the origin. **Scaling** stretches or squishes the sheet (pulling corners outward or pushing them in). **Rotation** spins the sheet around the pin. **Reflection** flips the sheet over like a pancake. **Shear** is like pushing the top edge of the sheet sideways while the bottom stays fixed — everything tilts. The key constraint: the pin (origin) never moves, and straight lines on the sheet remain straight. That's linearity. Translation — sliding the entire sheet — would move the pin, which is why it's NOT a linear transform.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Building transform matrices from scratch
using UnityEngine;

public class LinearTransformations : MonoBehaviour
{
    [Header("Squash & Stretch")]
    public float squashAmount = 0.7f;
    public float stretchAmount = 1.4f;

    void Start()
    {
        // === SCALE MATRIX (Squash effect) ===
        Matrix4x4 squash = Matrix4x4.Scale(
            new Vector3(1f / squashAmount, squashAmount, 1f / squashAmount));

        // === ROTATION MATRIX (45° around Y) ===
        float angle = 45f * Mathf.Deg2Rad;
        Matrix4x4 rotY = new Matrix4x4(
            new Vector4(Mathf.Cos(angle), 0, -Mathf.Sin(angle), 0),  // col 0
            new Vector4(0, 1, 0, 0),                                   // col 1
            new Vector4(Mathf.Sin(angle), 0, Mathf.Cos(angle), 0),    // col 2
            new Vector4(0, 0, 0, 1)                                    // col 3
        );

        // === REFLECTION MATRIX (Mirror across YZ plane, flip X) ===
        Matrix4x4 mirror = Matrix4x4.Scale(new Vector3(-1f, 1f, 1f));

        // === COMBINE: Mirror, then rotate, then squash ===
        Matrix4x4 combined = squash * rotY * mirror;

        // Transform a point
        Vector3 original = new Vector3(1f, 2f, 3f);
        Vector3 transformed = combined.MultiplyPoint3x4(original);
        Debug.Log($"Original: {original} → Transformed: {transformed}");
    }

    // === ANIMATED SQUASH & STRETCH ON LANDING ===
    public void ApplySquashStretch(Transform target, float t)
    {
        // t = 0: normal, t = 1: full squash
        float yScale = Mathf.Lerp(1f, squashAmount, t);
        float xzScale = Mathf.Lerp(1f, 1f / squashAmount, t); // Preserve volume

        target.localScale = new Vector3(xzScale, yScale, xzScale);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What two properties define a linear transformation? :: **Additivity:** $T(\vec{u}+\vec{v}) = T(\vec{u})+T(\vec{v})$ and **Homogeneity:** $T(k\vec{v}) = kT(\vec{v})$. The origin must stay fixed and lines remain lines.
- Why is translation NOT a linear transformation? :: Because $T(\vec{0}) \neq \vec{0}$. Translation moves the origin, violating the fundamental property that linear transforms keep the origin fixed.
- What does the 2D rotation matrix look like for angle $\theta$? :: $\begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$. The columns are the rotated basis vectors.
- How do you create a volume-preserving squash effect? :: Scale Y by factor $s$ (e.g., 0.7) and scale X and Z by $1/\sqrt{s}$ (or $1/s$ for a simpler approximation). This keeps the determinant close to 1, preserving volume.
- What happens if you scale non-uniformly and then rotate? :: The result is correct: scale in local space, then rotate. But if you rotate first and then scale non-uniformly along world axes, you can introduce **shear** (skewing), which is usually undesirable.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Applying non-uniform scale to an already-rotated object. If the object is rotated 45° and you scale X by 2, you're scaling along the world X-axis, not the object's local X-axis — producing an unintended shear effect.
> - **The Fix:** Always apply scale in local space (before rotation): $\mathbf{M} = \mathbf{R} \cdot \mathbf{S}$. In Unity, set `localScale` which applies before rotation. Or decompose the transform and recompose in the correct order.
> - **Why:** Non-uniform scaling is NOT rotation-invariant. $\mathbf{S} \cdot \mathbf{R} \neq \mathbf{R} \cdot \mathbf{S}$ when $s_x \neq s_y \neq s_z$. The scale axes must align with the object's own axes to avoid shear.

---

## Related Topics
- [[Math/04_Matrices_Transforms/affine_transformations|Affine Transformations]]
- [[Math/06_Rotations_Orientation/rotation_matrices_2d|2D Rotation Matrices]]
- [[Math/04_Matrices_Transforms/matrix_operations|Matrix Operations]]
