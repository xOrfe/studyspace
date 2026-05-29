---
title: "2D Rotation Matrices: Spinning Things Right"
tags:
  - math
  - level/Lv2
  - category/rotations_orientation
---

# 2D Rotation Matrices: Spinning Things Right

> [!abstract] **The Concept in a Nutshell**
> A 2D rotation matrix rotates a point around the origin by angle $\theta$ using sine and cosine. It is a $2 \times 2$ orthogonal matrix that preserves distances and angles — the shape doesn't stretch or skew, it only spins. To rotate around an **arbitrary point**, you translate to the origin, rotate, then translate back. This is the foundation for all rotation math in games.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Top-Down Space Shooter**
> You're piloting a spaceship in a top-down 2D shooter. The ship needs to face wherever the mouse cursor points. The mouse is at screen position $(800, 350)$, and your ship is at $(400, 300)$.
>
> The direction vector is $(800 - 400, 350 - 300) = (400, 50)$. Using `atan2`, the angle is $\theta \approx 7.1°$. You build a rotation matrix with this angle and apply it to the ship's sprite vertices, the thrust particle emitter direction, and the bullet spawn direction — all using the same rotation matrix.
>
> The turrets on your ship also need to rotate independently — each turret pivots around its own mount point, not the ship's center. That's rotation around an arbitrary point.

---

## The Blueprint (Formula & Structure)

### The 2D Rotation Matrix

To rotate a point $(x, y)$ by angle $\theta$ **counterclockwise** around the origin:

$$\mathbf{R}(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$$

Applied to a point:

$$\begin{bmatrix} x' \\ y' \end{bmatrix} = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix}$$

Expanded:

$$x' = x\cos\theta - y\sin\theta$$
$$y' = x\sin\theta + y\cos\theta$$

### Derivation from Trigonometry

A point at $(r, 0)$ — i.e., on the positive $x$-axis at distance $r$ from the origin — rotated by $\theta$ lands at:

$$(r\cos\theta,\; r\sin\theta)$$

For a general point at angle $\phi$ from the $x$-axis:
- Original: $(r\cos\phi,\; r\sin\phi)$
- After rotation by $\theta$: $(r\cos(\phi + \theta),\; r\sin(\phi + \theta))$

Expanding with the angle addition formulas:

$$r\cos(\phi + \theta) = r\cos\phi\cos\theta - r\sin\phi\sin\theta = x\cos\theta - y\sin\theta$$
$$r\sin(\phi + \theta) = r\cos\phi\sin\theta + r\sin\phi\cos\theta = x\sin\theta + y\cos\theta$$

This is exactly the matrix multiplication — the derivation is complete.

### Properties of the Rotation Matrix

| Property | Formula | Meaning |
|---|---|---|
| Orthogonal | $\mathbf{R}^T = \mathbf{R}^{-1}$ | Transpose is the inverse |
| Determinant | $\det(\mathbf{R}) = 1$ | No scaling, preserves area |
| Inverse rotation | $\mathbf{R}(-\theta) = \mathbf{R}^T(\theta)$ | Negate angle = transpose |
| Composition | $\mathbf{R}(\alpha) \cdot \mathbf{R}(\beta) = \mathbf{R}(\alpha + \beta)$ | Rotations add |
| Distance preserving | $\|\mathbf{R}\mathbf{v}\| = \|\mathbf{v}\|$ | Lengths unchanged |

### Rotation Around an Arbitrary Point

To rotate around point $\mathbf{c} = (c_x, c_y)$ instead of the origin:

$$\mathbf{p}' = \mathbf{R}(\theta) \cdot (\mathbf{p} - \mathbf{c}) + \mathbf{c}$$

In steps:
1. **Translate** so $\mathbf{c}$ is at the origin: $\mathbf{p}_\text{shifted} = \mathbf{p} - \mathbf{c}$
2. **Rotate** around the origin: $\mathbf{p}_\text{rotated} = \mathbf{R}(\theta) \cdot \mathbf{p}_\text{shifted}$
3. **Translate back**: $\mathbf{p}' = \mathbf{p}_\text{rotated} + \mathbf{c}$

As a single $3 \times 3$ homogeneous matrix:

$$\begin{bmatrix} \cos\theta & -\sin\theta & c_x(1-\cos\theta) + c_y\sin\theta \\ \sin\theta & \cos\theta & c_y(1-\cos\theta) - c_x\sin\theta \\ 0 & 0 & 1 \end{bmatrix}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Clock Hand**
> Think of a clock hand pinned at the center. The rotation matrix is the instruction "move the tip of the hand $\theta$ degrees around the circle." The hand's length (distance from center) never changes — that's the distance-preserving property.
>
> - Each column of the rotation matrix tells you where the basis vectors $(1,0)$ and $(0,1)$ end up after rotation.
> - Column 1: $(\cos\theta, \sin\theta)$ — the new X-axis direction.
> - Column 2: $(-\sin\theta, \cos\theta)$ — the new Y-axis direction (always perpendicular to column 1).
>
> Rotating around an arbitrary point? Imagine moving the clock so its center is at the origin, spinning the hand, then sliding the clock back. Same idea.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Rotating a 2D spaceship to face the mouse cursor
using UnityEngine;

public class ShipRotation2D : MonoBehaviour
{
    public float rotationSpeed = 720f; // degrees per second

    void Update()
    {
        // Get mouse position in world space
        Vector3 mouseWorld = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        mouseWorld.z = 0f;

        // Direction from ship to mouse
        Vector2 direction = (mouseWorld - transform.position).normalized;

        // Calculate target angle using atan2
        // atan2 gives the angle from +X axis to the direction vector
        float targetAngle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;

        // In Unity 2D, sprite "forward" is often +Y (up), so offset by -90°
        targetAngle -= 90f;

        // Smooth rotation toward target
        float currentAngle = transform.eulerAngles.z;
        float newAngle = Mathf.MoveTowardsAngle(
            currentAngle, targetAngle, rotationSpeed * Time.deltaTime);
        transform.rotation = Quaternion.Euler(0f, 0f, newAngle);

        // === Manual rotation matrix (educational) ===
        float theta = targetAngle * Mathf.Deg2Rad;
        float cos = Mathf.Cos(theta);
        float sin = Mathf.Sin(theta);

        // Rotate a point manually (e.g., bullet spawn offset)
        Vector2 localOffset = new Vector2(0f, 1.5f); // 1.5 units "ahead"
        Vector2 rotatedOffset = new Vector2(
            localOffset.x * cos - localOffset.y * sin,
            localOffset.x * sin + localOffset.y * cos
        );
        Vector2 bulletSpawn = (Vector2)transform.position + rotatedOffset;

        // Rotate around an arbitrary pivot (e.g., turret mount)
        Vector2 turretMount = (Vector2)transform.position + new Vector2(0.5f, 0f);
        Vector2 turretTip = new Vector2(0.5f, 1f);  // relative to ship center
        Vector2 rotatedTurretTip = RotateAroundPoint(
            turretTip, turretMount, 45f * Mathf.Deg2Rad);
    }

    /// <summary>
    /// Rotate point p around center c by angle theta (radians).
    /// </summary>
    Vector2 RotateAroundPoint(Vector2 p, Vector2 c, float theta)
    {
        float cos = Mathf.Cos(theta);
        float sin = Mathf.Sin(theta);

        // Translate to origin
        Vector2 shifted = p - c;

        // Apply rotation
        Vector2 rotated = new Vector2(
            shifted.x * cos - shifted.y * sin,
            shifted.x * sin + shifted.y * cos
        );

        // Translate back
        return rotated + c;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Write the 2D rotation matrix for angle $\theta$. :: $\mathbf{R}(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$. The first column is where $(1,0)$ maps to, the second column is where $(0,1)$ maps to.
- What does "orthogonal" mean for a rotation matrix? :: Its transpose equals its inverse: $\mathbf{R}^T = \mathbf{R}^{-1}$. This means it **preserves lengths and angles** — no stretching or shearing, just pure rotation.
- How do you rotate around an arbitrary point $\mathbf{c}$ instead of the origin? :: 1) Translate: $\mathbf{p} - \mathbf{c}$. 2) Rotate around origin: $\mathbf{R}(\theta)(\mathbf{p} - \mathbf{c})$. 3) Translate back: result $+ \mathbf{c}$. Formula: $\mathbf{p}' = \mathbf{R}(\theta)(\mathbf{p} - \mathbf{c}) + \mathbf{c}$.
- Why is $\det(\mathbf{R}) = 1$ significant? :: A determinant of 1 means the matrix preserves **area** and **orientation** (no reflection). A determinant of $-1$ would mean a rotation + reflection. This confirms it's a proper rotation.
- What is the relationship between the two columns of the 2D rotation matrix? :: They are **perpendicular unit vectors** (orthonormal). Column 1 = $(\cos\theta, \sin\theta)$, Column 2 = $(-\sin\theta, \cos\theta)$. Their dot product is zero and each has magnitude 1.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Applying the rotation matrix around the origin when you meant to rotate around the object's center, causing the object to orbit wildly around $(0,0)$.
> - **The Fix:** Always translate the object to the origin first, rotate, then translate back. Use the three-step pattern: $\mathbf{R}(\theta)(\mathbf{p} - \mathbf{c}) + \mathbf{c}$.
> - **Why:** The standard rotation matrix is defined for rotation around the origin. If your pivot point isn't at the origin, the object traces a circle around $(0,0)$ — definitely not what you wanted.

> [!danger] **Watch Out!**
> - **The Mistake:** Confusing clockwise and counterclockwise rotation. The standard matrix rotates **counterclockwise** in a right-handed system where $+Y$ is up.
> - **The Fix:** If you need clockwise rotation, negate the angle: use $\mathbf{R}(-\theta)$. In screen coordinates (where $+Y$ is down), the visual rotation direction is reversed.
> - **Why:** Mathematical convention is counterclockwise = positive. Screen space flips $Y$, which reverses the apparent rotation direction.

---

## Related Topics
- [[Math/06_Rotations_Orientation/euler_angles|Euler Angles]]
- [[Math/04_Matrices_Transforms/linear_transformations|Linear Transformations]]
- [[Math/02_Geometry_Trigonometry/trigonometric_functions|Trigonometric Functions]]
