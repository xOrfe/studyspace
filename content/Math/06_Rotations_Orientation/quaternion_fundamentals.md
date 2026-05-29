---
title: "Quaternion Fundamentals: The 4D Key to 3D Rotation"
tags:
  - math
  - level/Lv3
  - category/rotations_orientation
---

# Quaternion Fundamentals: The 4D Key to 3D Rotation

> [!abstract] **The Concept in a Nutshell**
> A **quaternion** is a 4-component number $\mathbf{q} = (w, x, y, z) = w + x\mathbf{i} + y\mathbf{j} + z\mathbf{k}$ that extends complex numbers to 4 dimensions. **Unit quaternions** ($\|\mathbf{q}\| = 1$) represent 3D rotations compactly and without gimbal lock. They relate directly to axis-angle: a rotation of $\theta$ around axis $\hat{\mathbf{k}}$ becomes $\mathbf{q} = (\cos\frac{\theta}{2},\; \hat{\mathbf{k}}\sin\frac{\theta}{2})$. Unity stores all rotations as quaternions internally, and understanding them unlocks smooth interpolation, composition, and robust orientation math.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Character Controller in a 3D Platformer**
> Your character can run up walls, flip upside down, and orient freely in a zero-gravity section. You try using Euler angles: `transform.eulerAngles = new Vector3(pitch, yaw, roll)`. It works on flat ground but goes haywire when the character is upside down — angles wrap, gimbal lock strikes, and the character spasms.
>
> You switch to quaternions. `Transform.rotation` is already a `Quaternion` in Unity. You build rotations with `Quaternion.LookRotation(forwardDir, upDir)`, compose them with the `*` operator, and interpolate smoothly with `Quaternion.Slerp`. No gimbal lock. No wrapping angles. No axis alignment issues.
>
> When the inspector shows `Rotation: (0, 0, 0, 1)`, that's `Quaternion.identity` — no rotation. When it shows `(0.707, 0, 0.707, 0)`, that's a $180°$ rotation around the $Y$-axis. Learning to read these numbers is like learning to read sheet music — essential for any serious 3D game developer.

---

## The Blueprint (Formula & Structure)

### Quaternion Definition

A quaternion has a **scalar** (real) part and a **vector** (imaginary) part:

$$\mathbf{q} = w + x\mathbf{i} + y\mathbf{j} + z\mathbf{k} = (w, \vec{\mathbf{v}})$$

Where $\vec{\mathbf{v}} = (x, y, z)$ and the basis elements obey:

$$\mathbf{i}^2 = \mathbf{j}^2 = \mathbf{k}^2 = \mathbf{ijk} = -1$$

From this, the cross-products follow:

$$\mathbf{ij} = \mathbf{k}, \quad \mathbf{jk} = \mathbf{i}, \quad \mathbf{ki} = \mathbf{j}$$
$$\mathbf{ji} = -\mathbf{k}, \quad \mathbf{kj} = -\mathbf{i}, \quad \mathbf{ik} = -\mathbf{j}$$

### Unit Quaternions and Rotation

A **unit quaternion** has $\|\mathbf{q}\| = 1$ and represents a rotation of angle $\theta$ around unit axis $\hat{\mathbf{k}}$:

$$\mathbf{q} = \left(\cos\frac{\theta}{2},\; \hat{\mathbf{k}}\sin\frac{\theta}{2}\right)$$

> The **half-angle** is crucial: quaternions use $\theta/2$ because they apply the rotation via the sandwich product $\mathbf{qvq}^*$, which doubles the angle.

### Identity Quaternion

No rotation:

$$\mathbf{q}_\text{identity} = (1, 0, 0, 0)$$

This corresponds to $\theta = 0$: $\cos(0) = 1$, $\sin(0) = 0$.

### Quaternion Norm (Magnitude)

$$\|\mathbf{q}\| = \sqrt{w^2 + x^2 + y^2 + z^2}$$

For unit quaternions: $\|\mathbf{q}\| = 1$ always. If numerical drift causes $\|\mathbf{q}\| \neq 1$, **renormalize**.

### Conjugate

$$\mathbf{q}^* = (w, -x, -y, -z) = (w, -\vec{\mathbf{v}})$$

The conjugate negates the vector part, which reverses the rotation axis (same angle, opposite direction).

### Inverse

$$\mathbf{q}^{-1} = \frac{\mathbf{q}^*}{\|\mathbf{q}\|^2}$$

For **unit quaternions**, the inverse equals the conjugate:

$$\mathbf{q}^{-1} = \mathbf{q}^* \quad \text{(when } \|\mathbf{q}\| = 1\text{)}$$

This is the quaternion analog of orthogonal matrices having $\mathbf{R}^{-1} = \mathbf{R}^T$.

### Common Rotations as Quaternions

| Rotation | Quaternion $(w, x, y, z)$ |
|---|---|
| No rotation (identity) | $(1, 0, 0, 0)$ |
| $90°$ around Y-axis | $(\frac{\sqrt{2}}{2}, 0, \frac{\sqrt{2}}{2}, 0) \approx (0.707, 0, 0.707, 0)$ |
| $180°$ around Y-axis | $(0, 0, 1, 0)$ |
| $90°$ around X-axis | $(0.707, 0.707, 0, 0)$ |
| $180°$ around Z-axis | $(0, 0, 0, 1)$ |

### Why Half-Angles?

Two quaternions $\mathbf{q}$ and $-\mathbf{q}$ represent the **same rotation** (double cover of SO(3)). This means the quaternion space covers each rotation **twice**. The half-angle ensures that composing rotations works correctly through the sandwich product.

### Quaternion vs Other Representations

| Representation | Values | Gimbal Lock? | Interpolation | Composition |
|---|---|---|---|---|
| Euler Angles | 3 floats | Yes | Poor | Order-dependent |
| Rotation Matrix | 9 floats (6 constraints) | No | Expensive | Matrix multiply |
| Axis-Angle | 4 floats | No | Difficult | No simple formula |
| **Quaternion** | **4 floats (1 constraint)** | **No** | **Slerp** | **Quaternion multiply** |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Spinning Top's Instructions**
> Think of a quaternion as a compact instruction card:
>
> **"Spin $\theta$ degrees around this direction."**
>
> - The **vector part** $(x, y, z)$ points along the rotation axis — like the skewer through a spinning top.
> - The **scalar part** $w$ encodes how much rotation to apply.
> - Both are scaled by the half-angle: $w = \cos(\theta/2)$, vector $= \hat{\mathbf{k}} \sin(\theta/2)$.
>
> **No rotation?** The instruction is "spin 0° around... doesn't matter." That's $w = 1$, vector $= (0,0,0)$ — the identity.
>
> **Full 180° flip around Y?** "Spin 180° around Y." Half-angle = 90°, so $w = \cos(90°) = 0$, vector $= (0, 1, 0)$. Quaternion: $(0, 0, 1, 0)$.
>
> The beauty is that **any** orientation, no matter how twisted, has exactly one axis and angle — and thus one quaternion (well, two: $\mathbf{q}$ and $-\mathbf{q}$, but they mean the same thing).

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Working with quaternion fundamentals
using UnityEngine;

public class QuaternionBasics : MonoBehaviour
{
    void Start()
    {
        // === Identity quaternion ===
        Quaternion id = Quaternion.identity; // (0, 0, 0, 1) in Unity's xyzw order
        Debug.Log($"Identity: {id}"); // Note: Unity prints (x, y, z, w)

        // === From Axis-Angle ===
        // 90° rotation around the Y-axis
        Vector3 axis = Vector3.up;
        float angle = 90f;
        Quaternion q = Quaternion.AngleAxis(angle, axis);
        Debug.Log($"90° around Y: {q}");
        // Expected: (0, 0.707, 0, 0.707) → (x, y, z, w)

        // === Verify the half-angle encoding ===
        float halfAngle = angle * Mathf.Deg2Rad * 0.5f;
        float expectedW = Mathf.Cos(halfAngle); // cos(45°) ≈ 0.707
        float expectedY = Mathf.Sin(halfAngle); // sin(45°) ≈ 0.707
        Debug.Log($"Manual: w={expectedW:F3}, y={expectedY:F3}");

        // === Quaternion properties ===
        // Norm (should be 1 for unit quaternion)
        float norm = Mathf.Sqrt(q.x*q.x + q.y*q.y + q.z*q.z + q.w*q.w);
        Debug.Log($"Norm: {norm}"); // 1.0

        // === Conjugate (inverse for unit quaternions) ===
        // Reverses the rotation
        Quaternion conjugate = Quaternion.Inverse(q);
        // q * inverse(q) should give identity
        Quaternion shouldBeIdentity = q * conjugate;
        Debug.Log($"q * q⁻¹ = {shouldBeIdentity}"); // ≈ identity

        // === Extracting axis and angle back ===
        float extractedAngle;
        Vector3 extractedAxis;
        q.ToAngleAxis(out extractedAngle, out extractedAxis);
        Debug.Log($"Extracted: {extractedAngle}° around {extractedAxis}");

        // === Creating from direction ===
        // "What rotation makes +Z point toward the target?"
        Vector3 targetDir = new Vector3(1f, 0f, 1f).normalized;
        Quaternion lookRot = Quaternion.LookRotation(targetDir, Vector3.up);
        Debug.Log($"Look rotation: {lookRot}");

        // === Double cover: q and -q are the same rotation ===
        Quaternion negQ = new Quaternion(-q.x, -q.y, -q.z, -q.w);
        Vector3 testPoint = new Vector3(1f, 2f, 3f);
        Vector3 result1 = q * testPoint;
        Vector3 result2 = negQ * testPoint;
        Debug.Log($"q * p = {result1}");
        Debug.Log($"-q * p = {result2}"); // Same result!
    }

    void Update()
    {
        // Renormalization example (combat numerical drift)
        Quaternion rot = transform.rotation;
        float sqrMag = rot.x*rot.x + rot.y*rot.y + rot.z*rot.z + rot.w*rot.w;

        if (Mathf.Abs(sqrMag - 1f) > 0.001f)
        {
            // Quaternion has drifted from unit length — renormalize
            float invMag = 1f / Mathf.Sqrt(sqrMag);
            transform.rotation = new Quaternion(
                rot.x * invMag, rot.y * invMag,
                rot.z * invMag, rot.w * invMag);
            Debug.LogWarning("Quaternion renormalized!");
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How do you convert an axis-angle rotation to a quaternion? :: $\mathbf{q} = (\cos\frac{\theta}{2},\; \hat{\mathbf{k}}_x\sin\frac{\theta}{2},\; \hat{\mathbf{k}}_y\sin\frac{\theta}{2},\; \hat{\mathbf{k}}_z\sin\frac{\theta}{2})$. The scalar part gets the cosine of the **half-angle**, the vector part gets the axis scaled by the sine of the half-angle.
- What is the quaternion identity and what rotation does it represent? :: $\mathbf{q}_\text{id} = (1, 0, 0, 0)$ — it represents **no rotation**. $\cos(0/2) = 1$, $\sin(0/2) = 0$.
- Why does a quaternion use the half-angle $\theta/2$ instead of $\theta$? :: Because the rotation is applied via the **sandwich product** $\mathbf{qvq}^*$, which effectively applies the angle twice. Using $\theta/2$ ensures the net rotation is exactly $\theta$.
- How do you compute the inverse of a unit quaternion? :: For unit quaternions, $\mathbf{q}^{-1} = \mathbf{q}^* = (w, -x, -y, -z)$ — just negate the vector part. This gives the reverse rotation.
- What does it mean that $\mathbf{q}$ and $-\mathbf{q}$ represent the same rotation? :: Quaternions provide a **double cover** of 3D rotations. Both $\mathbf{q}$ and $-\mathbf{q}$ produce identical results when applied to a vector via the sandwich product. This matters for interpolation (always choose the shorter path).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Trying to manually set quaternion components like `transform.rotation = new Quaternion(0.5f, 0.5f, 0.5f, 0.5f)` without ensuring it's a unit quaternion.
> - **The Fix:** Always use factory methods: `Quaternion.AngleAxis()`, `Quaternion.Euler()`, `Quaternion.LookRotation()`. If you must set components manually, verify $\|\mathbf{q}\| = 1$ or normalize afterward.
> - **Why:** Non-unit quaternions don't represent pure rotations. They introduce scaling and shearing. Unity won't automatically normalize for you.

> [!danger] **Watch Out!**
> - **The Mistake:** Confusing Unity's quaternion component order `(x, y, z, w)` with the mathematical convention `(w, x, y, z)`.
> - **The Fix:** Unity's `Quaternion` struct stores and prints as `(x, y, z, w)`. The **scalar** part is `.w`, not the first component. When reading math textbooks, mentally reorder.
> - **Why:** This is a historical convention difference. Math and physics literature puts $w$ first; Unity (and many engines) put it last. Getting them mixed means your "identity" quaternion becomes a $180°$ rotation.

---

## Related Topics
- [[Math/06_Rotations_Orientation/quaternion_rotations|Quaternion Rotations]]
- [[Math/06_Rotations_Orientation/euler_angles|Euler Angles]]
- [[Math/06_Rotations_Orientation/rotation_arbitrary_axis|Rotation Around Arbitrary Axis]]
