---
title: "Quaternion Rotations: The Sandwich Product & Beyond"
tags:
  - math
  - level/Lv3
  - category/rotations_orientation
---

# Quaternion Rotations: The Sandwich Product & Beyond

> [!abstract] **The Concept in a Nutshell**
> To rotate a vector $\mathbf{v}$ by quaternion $\mathbf{q}$, you use the **sandwich product**: $\mathbf{v}' = \mathbf{q}\mathbf{v}\mathbf{q}^*$. To compose two rotations, simply multiply their quaternions: $\mathbf{q}_\text{total} = \mathbf{q}_2 \cdot \mathbf{q}_1$ (apply $\mathbf{q}_1$ first, then $\mathbf{q}_2$). Quaternions can be converted to/from rotation matrices and Euler angles, making them the universal language connecting all rotation representations.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Tank Turret Tracking a Moving Target**
> An enemy helicopter is circling your tank at $(200, 50, 150)$ while you're parked at $(100, 0, 80)$. The turret needs to rotate to aim at the helicopter, smoothly transitioning from its current facing direction.
>
> Each frame:
> 1. Calculate the desired direction: `targetDir = (helicopter.pos - turret.pos).normalized`
> 2. Build the target quaternion: `Quaternion.LookRotation(targetDir)`
> 3. Smoothly rotate: `turret.rotation = Quaternion.RotateTowards(current, target, speed * dt)`
>
> The turret's barrel rotates in the turret's local space (yaw only, pitch separate), while the turret base rotates in the tank's local space. These rotations **compose**: $\mathbf{q}_\text{barrel\_world} = \mathbf{q}_\text{tank} \cdot \mathbf{q}_\text{turret} \cdot \mathbf{q}_\text{barrel}$. This is quaternion multiplication in action.

---

## The Blueprint (Formula & Structure)

### The Sandwich Product (Rotating a Vector)

To rotate vector $\mathbf{v}$ by unit quaternion $\mathbf{q}$:

1. Embed $\mathbf{v}$ as a pure quaternion: $\mathbf{p} = (0, v_x, v_y, v_z)$
2. Apply: $\mathbf{p}' = \mathbf{q} \cdot \mathbf{p} \cdot \mathbf{q}^*$
3. Extract the vector part of $\mathbf{p}'$: $\mathbf{v}' = (p'_x, p'_y, p'_z)$

> The result $\mathbf{p}'$ is always a pure quaternion (scalar part = 0), so you can safely extract just the vector part.

### Quaternion Multiplication

For $\mathbf{q}_1 = (w_1, \vec{\mathbf{v}}_1)$ and $\mathbf{q}_2 = (w_2, \vec{\mathbf{v}}_2)$:

$$\mathbf{q}_1 \cdot \mathbf{q}_2 = (w_1w_2 - \vec{\mathbf{v}}_1 \cdot \vec{\mathbf{v}}_2,\; w_1\vec{\mathbf{v}}_2 + w_2\vec{\mathbf{v}}_1 + \vec{\mathbf{v}}_1 \times \vec{\mathbf{v}}_2)$$

In component form:

$$w = w_1w_2 - x_1x_2 - y_1y_2 - z_1z_2$$
$$x = w_1x_2 + x_1w_2 + y_1z_2 - z_1y_2$$
$$y = w_1y_2 - x_1z_2 + y_1w_2 + z_1x_2$$
$$z = w_1z_2 + x_1y_2 - y_1x_2 + z_1w_2$$

> **Key:** Quaternion multiplication is **not commutative**: $\mathbf{q}_1 \mathbf{q}_2 \neq \mathbf{q}_2 \mathbf{q}_1$ (in general).

### Composing Rotations

To apply rotation $\mathbf{q}_1$ first, then $\mathbf{q}_2$:

$$\mathbf{q}_\text{total} = \mathbf{q}_2 \cdot \mathbf{q}_1$$

Read right-to-left, same as matrix composition: $\mathbf{M}_\text{total} = \mathbf{M}_2 \cdot \mathbf{M}_1$.

### Quaternion → Rotation Matrix

For unit quaternion $\mathbf{q} = (w, x, y, z)$:

$$\mathbf{R} = \begin{bmatrix} 1 - 2(y^2 + z^2) & 2(xy - wz) & 2(xz + wy) \\ 2(xy + wz) & 1 - 2(x^2 + z^2) & 2(yz - wx) \\ 2(xz - wy) & 2(yz + wx) & 1 - 2(x^2 + y^2) \end{bmatrix}$$

### Rotation Matrix → Quaternion

Given rotation matrix $\mathbf{R}$, using **Shepperd's method** for numerical stability:

$$w = \frac{1}{2}\sqrt{1 + R_{00} + R_{11} + R_{22}}$$
$$x = \frac{R_{21} - R_{12}}{4w}, \quad y = \frac{R_{02} - R_{20}}{4w}, \quad z = \frac{R_{10} - R_{01}}{4w}$$

*(When $w \approx 0$, use alternative formulas based on the largest diagonal element to avoid division by zero.)*

### Euler Angles → Quaternion

For Euler angles $(\alpha, \beta, \gamma)$ in YXZ order (Unity's convention):

$$\mathbf{q} = \mathbf{q}_y(\beta) \cdot \mathbf{q}_x(\alpha) \cdot \mathbf{q}_z(\gamma)$$

Where each elemental quaternion is:

$$\mathbf{q}_x(\alpha) = (\cos\frac{\alpha}{2}, \sin\frac{\alpha}{2}, 0, 0)$$
$$\mathbf{q}_y(\beta) = (\cos\frac{\beta}{2}, 0, \sin\frac{\beta}{2}, 0)$$
$$\mathbf{q}_z(\gamma) = (\cos\frac{\gamma}{2}, 0, 0, \sin\frac{\gamma}{2})$$

### Quaternion → Euler Angles

Extract from the rotation matrix (or directly from quaternion components):

$$\text{pitch} = \arcsin(2(wy - xz))$$
$$\text{yaw} = \text{atan2}(2(wx + yz),\; 1 - 2(x^2 + y^2))$$
$$\text{roll} = \text{atan2}(2(wz + xy),\; 1 - 2(y^2 + z^2))$$

*(Exact formulas depend on the Euler order convention.)*

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Two-Handed Rotation**
> Think of the sandwich product $\mathbf{qvq}^*$ as physically grabbing a vector with **two hands**:
>
> - $\mathbf{q}$ is your right hand applying a twist.
> - $\mathbf{q}^*$ is your left hand applying the opposite twist.
> - The vector $\mathbf{v}$ is in between.
>
> Together, the two twists combine to produce a clean rotation (each contributes half the angle — hence the $\theta/2$ in the quaternion). Without both hands, you'd get a rotation in 4D space that isn't a pure 3D rotation.
>
> **Composing** rotations is like stacking instructions: "First face north, then turn 30° right" = multiply the two quaternions. The order matters — "turn right then face north" gives a different result.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Turret tracking with quaternion composition
using UnityEngine;

public class TurretTracking : MonoBehaviour
{
    public Transform turretBase;    // Yaw (rotates on Y)
    public Transform turretBarrel;  // Pitch (rotates on X)
    public Transform target;        // The thing to aim at
    public float trackingSpeed = 90f; // degrees/sec

    void Update()
    {
        if (target == null) return;

        // Direction to target in world space
        Vector3 toTarget = target.position - turretBase.position;

        // === YAW: Rotate base around Y to face target horizontally ===
        Vector3 flatDir = new Vector3(toTarget.x, 0f, toTarget.z).normalized;
        if (flatDir.sqrMagnitude > 0.001f)
        {
            Quaternion targetYaw = Quaternion.LookRotation(flatDir, Vector3.up);
            turretBase.rotation = Quaternion.RotateTowards(
                turretBase.rotation, targetYaw,
                trackingSpeed * Time.deltaTime);
        }

        // === PITCH: Rotate barrel around local X to elevate/depress ===
        // Calculate pitch angle
        Vector3 localToTarget = turretBase.InverseTransformPoint(target.position);
        float pitchAngle = -Mathf.Atan2(localToTarget.y,
            localToTarget.z) * Mathf.Rad2Deg;
        pitchAngle = Mathf.Clamp(pitchAngle, -30f, 60f); // Barrel limits

        Quaternion targetPitch = Quaternion.Euler(pitchAngle, 0f, 0f);
        turretBarrel.localRotation = Quaternion.RotateTowards(
            turretBarrel.localRotation, targetPitch,
            trackingSpeed * Time.deltaTime);

        // === MANUAL COMPOSITION (educational) ===
        // The barrel's world rotation is the composition:
        // q_world = q_tank * q_base * q_barrel
        Quaternion composedWorld = transform.rotation  // tank body
                                * turretBase.localRotation
                                * turretBarrel.localRotation;

        // This should match turretBarrel.rotation (Unity computes it)
        float error = Quaternion.Angle(composedWorld, turretBarrel.rotation);
        // error should be ~0

        // === SANDWICH PRODUCT (manual vector rotation) ===
        // Rotate a point by a quaternion manually
        Vector3 forward = QuatRotateVector(turretBarrel.rotation,
                                            Vector3.forward);
        Debug.DrawRay(turretBarrel.position, forward * 50f, Color.red);
    }

    /// <summary>
    /// Manual sandwich product: v' = q * v * q*
    /// </summary>
    Vector3 QuatRotateVector(Quaternion q, Vector3 v)
    {
        // Embed v as pure quaternion (0, vx, vy, vz)
        Quaternion p = new Quaternion(v.x, v.y, v.z, 0f);

        // Conjugate: negate vector part
        Quaternion qConj = new Quaternion(-q.x, -q.y, -q.z, q.w);

        // Sandwich: q * p * q*
        Quaternion result = QuatMultiply(QuatMultiply(q, p), qConj);

        return new Vector3(result.x, result.y, result.z);
    }

    /// <summary>
    /// Manual quaternion multiplication.
    /// </summary>
    Quaternion QuatMultiply(Quaternion a, Quaternion b)
    {
        return new Quaternion(
            a.w * b.x + a.x * b.w + a.y * b.z - a.z * b.y,  // x
            a.w * b.y - a.x * b.z + a.y * b.w + a.z * b.x,  // y
            a.w * b.z + a.x * b.y - a.y * b.x + a.z * b.w,  // z
            a.w * b.w - a.x * b.x - a.y * b.y - a.z * b.z   // w
        );
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How do you rotate a vector $\mathbf{v}$ by quaternion $\mathbf{q}$? :: Use the **sandwich product**: embed $\mathbf{v}$ as a pure quaternion $\mathbf{p} = (0, \mathbf{v})$, then compute $\mathbf{p}' = \mathbf{q}\mathbf{p}\mathbf{q}^*$. Extract the vector part of $\mathbf{p}'$.
- How do you compose two quaternion rotations? :: Multiply them: $\mathbf{q}_\text{total} = \mathbf{q}_2 \cdot \mathbf{q}_1$ applies $\mathbf{q}_1$ **first**, then $\mathbf{q}_2$. Order matters (right-to-left, same as matrices).
- Write the quaternion multiplication formula in terms of scalar and vector parts. :: $(w_1, \vec{v}_1)(w_2, \vec{v}_2) = (w_1w_2 - \vec{v}_1 \cdot \vec{v}_2,\; w_1\vec{v}_2 + w_2\vec{v}_1 + \vec{v}_1 \times \vec{v}_2)$.
- Is quaternion multiplication commutative? :: **No.** $\mathbf{q}_1\mathbf{q}_2 \neq \mathbf{q}_2\mathbf{q}_1$ in general. This reflects the fact that 3D rotations themselves are non-commutative.
- What Unity function applies rotation $\mathbf{q}$ to vector $\mathbf{v}$? :: The `*` operator: `Vector3 rotated = q * v;`. This performs the sandwich product internally ($\mathbf{qvq}^*$).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Composing rotations in the wrong order. Writing `q1 * q2` when you meant to apply `q2` first, then `q1`.
> - **The Fix:** Remember: $\mathbf{q}_\text{total} = \mathbf{q}_\text{second} \cdot \mathbf{q}_\text{first}$. The **rightmost** quaternion is applied **first** — same convention as matrix multiplication.
> - **Why:** Quaternion multiplication follows matrix conventions. If you read left-to-right but compose left-to-right, you get the reverse of what you expected.

> [!danger] **Watch Out!**
> - **The Mistake:** Converting to Euler angles for "easier" manipulation (adding angles), then converting back — and losing precision or getting gimbal lock artifacts.
> - **The Fix:** Stay in quaternion space. Use `Quaternion.AngleAxis()` to create incremental rotations and multiply them. Use `Quaternion.RotateTowards()` or `Quaternion.Slerp()` for smooth transitions.
> - **Why:** Quaternion → Euler → Quaternion round-trips are lossy. Euler angles have degenerate regions (gimbal lock) and aren't unique (multiple angle sets = same rotation).

---

## Related Topics
- [[Math/06_Rotations_Orientation/quaternion_fundamentals|Quaternion Fundamentals]]
- [[Math/06_Rotations_Orientation/slerp_interpolation|Slerp Interpolation]]
- [[Math/06_Rotations_Orientation/rotation_arbitrary_axis|Rotation Around Arbitrary Axis]]
