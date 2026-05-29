---
title: "Euler Angles: The Intuitive Trap"
tags:
  - math
  - level/Lv2
  - category/rotations_orientation
---

# Euler Angles: The Intuitive Trap

> [!abstract] **The Concept in a Nutshell**
> Euler angles describe a 3D orientation as three sequential rotations around coordinate axes — commonly **pitch** (X), **yaw** (Y), and **roll** (Z). They're human-readable and intuitive, which is why every game engine exposes them in the inspector. However, they suffer from **rotation order dependency** (different orders give different results) and **gimbal lock** (a configuration where one degree of freedom is lost). Understanding their strengths and weaknesses is critical for choosing the right rotation representation.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: An FPS Camera Controller**
> You're implementing first-person camera controls. The player moves the mouse left/right to **yaw** (look around horizontally) and up/down to **pitch** (look up/down). The camera should never **roll** (tilt sideways) — that would be disorienting.
>
> ```
> Mouse ΔX → Yaw (rotate around world Y-axis)
> Mouse ΔY → Pitch (rotate around local X-axis)
> ```
>
> This works perfectly until the player looks straight up ($90°$ pitch). Suddenly, yaw and roll become the same axis — moving the mouse left/right either does nothing or causes weird twisting. That's **gimbal lock**. You clamp pitch to $±89°$ and it *mostly* works, but this is a fundamental limitation of Euler angles. For smooth 3D rotations (flight sims, space games), you'll need quaternions.

---

## The Blueprint (Formula & Structure)

### The Three Rotations

Euler angles decompose a 3D rotation into three elemental rotations around coordinate axes:

| Common Name | Axis | Symbol | Example |
|---|---|---|---|
| **Pitch** | X-axis | $\alpha$ | Looking up/down (nodding) |
| **Yaw** | Y-axis | $\beta$ | Looking left/right (shaking head) |
| **Roll** | Z-axis | $\gamma$ | Tilting sideways (ear to shoulder) |

### Individual Rotation Matrices

**Rotation around X (Pitch):**
$$\mathbf{R}_x(\alpha) = \begin{bmatrix} 1 & 0 & 0 \\ 0 & \cos\alpha & -\sin\alpha \\ 0 & \sin\alpha & \cos\alpha \end{bmatrix}$$

**Rotation around Y (Yaw):**
$$\mathbf{R}_y(\beta) = \begin{bmatrix} \cos\beta & 0 & \sin\beta \\ 0 & 1 & 0 \\ -\sin\beta & 0 & \cos\beta \end{bmatrix}$$

**Rotation around Z (Roll):**
$$\mathbf{R}_z(\gamma) = \begin{bmatrix} \cos\gamma & -\sin\gamma & 0 \\ \sin\gamma & \cos\gamma & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

### Rotation Order Matters!

The **combined** rotation matrix depends on the order of multiplication. Two common conventions:

**Intrinsic rotations** (rotate around the object's own axes — each rotation changes subsequent axes):
$$\mathbf{R} = \mathbf{R}_z(\gamma) \cdot \mathbf{R}_x(\alpha) \cdot \mathbf{R}_y(\beta)$$

**Extrinsic rotations** (rotate around fixed world axes):
$$\mathbf{R} = \mathbf{R}_y(\beta) \cdot \mathbf{R}_x(\alpha) \cdot \mathbf{R}_z(\gamma)$$

> **Unity uses ZXY intrinsic order** (applied as $\mathbf{R}_y \cdot \mathbf{R}_x \cdot \mathbf{R}_z$), which is equivalent to **YXZ extrinsic order**.

There are **12 possible Euler angle conventions** (XYZ, XZY, YXZ, YZX, ZXY, ZYX, plus 6 with repeated axes). This is a perpetual source of bugs when porting between engines.

### Gimbal Lock

Gimbal lock occurs when the middle rotation brings two axes into alignment, collapsing 3 degrees of freedom into 2.

For **YXZ order**, gimbal lock occurs at pitch $= \pm 90°$:

When $\alpha = 90°$: $\sin\alpha = 1$, $\cos\alpha = 0$

$$\mathbf{R}_y(\beta) \cdot \mathbf{R}_x(90°) \cdot \mathbf{R}_z(\gamma) = \begin{bmatrix} 0 & \sin(\gamma - \beta) & \cos(\gamma - \beta) \\ 0 & \cos(\gamma - \beta) & -\sin(\gamma - \beta) \\ -1 & 0 & 0 \end{bmatrix}$$

Notice: yaw $\beta$ and roll $\gamma$ now only appear as $(\gamma - \beta)$ — a single combined angle. Changing yaw has the same effect as changing roll. **One degree of freedom is lost.**

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Nested Turntables (Physical Gimbals)**
> Imagine three nested turntables, one inside the other — like a gyroscope:
>
> - **Outer ring** rotates around the Y-axis (yaw).
> - **Middle ring** (mounted inside the outer) rotates around X (pitch).
> - **Inner ring** (mounted inside the middle) rotates around Z (roll).
>
> When all three are at neutral, they spin around independent axes — 3 full degrees of freedom.
>
> Now pitch the middle ring to $90°$. The outer ring's Y-axis and the inner ring's Z-axis are now **parallel** — they both rotate around the same direction. Push the middle ring and two rings move together. You've "locked" two axes — that's gimbal lock.
>
> The takeaway: Euler angles describe orientation via *sequential* rotations around *nested* axes. When axes align, the nesting collapses.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — FPS camera with Euler angles + gimbal lock clamping
using UnityEngine;

public class FPSCamera : MonoBehaviour
{
    public float mouseSensitivity = 2f;
    public float pitchClamp = 89f; // Clamp to avoid gimbal lock

    private float yaw = 0f;
    private float pitch = 0f;

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;

        // Initialize from current rotation
        Vector3 euler = transform.eulerAngles;
        yaw = euler.y;
        pitch = euler.x;
    }

    void Update()
    {
        // Mouse input
        float mouseX = Input.GetAxis("Mouse X") * mouseSensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * mouseSensitivity;

        // Accumulate yaw and pitch
        yaw += mouseX;
        pitch -= mouseY;  // Inverted: mouse up → look up → negative pitch

        // CRITICAL: Clamp pitch to avoid gimbal lock zone
        // At ±90° pitch, yaw and roll become indistinguishable
        pitch = Mathf.Clamp(pitch, -pitchClamp, pitchClamp);

        // Apply as Euler angles
        // Unity applies these in ZXY order internally
        transform.rotation = Quaternion.Euler(pitch, yaw, 0f);

        // === Demonstrating rotation order dependency ===
        Quaternion orderA = Quaternion.Euler(45f, 30f, 0f); // Unity's ZXY
        // Manual: Y first, then X (equivalent to Unity's convention)
        Quaternion orderB = Quaternion.AngleAxis(30f, Vector3.up)
                          * Quaternion.AngleAxis(45f, Vector3.right);
        // These match because Unity uses YXZ extrinsic = ZXY intrinsic

        // If we reverse the order:
        Quaternion orderC = Quaternion.AngleAxis(45f, Vector3.right)
                          * Quaternion.AngleAxis(30f, Vector3.up);
        // orderC ≠ orderB — rotation order matters!

        Debug.Log($"Same order: {Quaternion.Angle(orderA, orderB):F2}°");
        Debug.Log($"Diff order: {Quaternion.Angle(orderA, orderC):F2}°");
    }

    // Demonstrate gimbal lock
    void ShowGimbalLock()
    {
        // At pitch = 90°, try changing yaw...
        Quaternion q1 = Quaternion.Euler(90f, 0f, 0f);
        Quaternion q2 = Quaternion.Euler(90f, 45f, 0f);
        Quaternion q3 = Quaternion.Euler(90f, 0f, 45f);

        // q2 and q3 describe the SAME orientation when pitch = 90!
        // Yaw and roll become interchangeable
        float diff = Quaternion.Angle(q2, q3);
        Debug.Log($"Gimbal lock demo — angle between q2 & q3: {diff:F2}°");
        // This will be 0 (or very small), proving the axes collapsed
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What are the three Euler angles commonly called in game dev? :: **Pitch** (X-axis, looking up/down), **Yaw** (Y-axis, looking left/right), **Roll** (Z-axis, tilting sideways).
- Why does rotation order matter with Euler angles? :: 3D rotations are **not commutative**: $\mathbf{R}_x \cdot \mathbf{R}_y \neq \mathbf{R}_y \cdot \mathbf{R}_x$. Different orders produce different final orientations. There are 12 possible Euler conventions.
- What is gimbal lock and when does it occur? :: Gimbal lock occurs when the **middle rotation** in the Euler sequence aligns two axes, reducing 3 degrees of freedom to 2. For YXZ order, it happens at pitch $= \pm 90°$ — yaw and roll become indistinguishable.
- What Euler angle order does Unity use? :: Unity uses **ZXY intrinsic** order, equivalent to **YXZ extrinsic**. When you set `transform.eulerAngles = (pitch, yaw, roll)`, it applies yaw (Y), then pitch (X), then roll (Z) in world coordinates.
- How do FPS games typically handle gimbal lock? :: They **clamp pitch** to approximately $\pm 89°$, never allowing it to reach the singularity at $\pm 90°$. This is practical because most FPS games don't need the camera to look perfectly straight up or down.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Reading `transform.eulerAngles` to accumulate rotation deltas like `euler.y += mouseX` directly, then writing it back.
> - **The Fix:** Track your own `yaw` and `pitch` float variables. Rebuild the rotation from scratch each frame: `Quaternion.Euler(pitch, yaw, 0)`.
> - **Why:** Unity's `eulerAngles` can return unexpected values (e.g., 270° instead of -90°) because Euler angles are not unique — many different angle triplets can represent the same rotation. Accumulated read-modify-write causes drift and jumps.

> [!danger] **Watch Out!**
> - **The Mistake:** Assuming Euler angles from one engine/tool work directly in another (e.g., exporting from Blender to Unity).
> - **The Fix:** Always verify the **rotation order** and **axis conventions** (Y-up vs Z-up, left-handed vs right-handed) when transferring Euler angles between systems.
> - **Why:** Blender uses XYZ Euler by default in a Z-up right-handed system. Unity uses ZXY in a Y-up left-handed system. The same numbers $(30, 45, 60)$ will produce completely different orientations.

---

## Related Topics
- [[Math/06_Rotations_Orientation/rotation_matrices_2d|2D Rotation Matrices]]
- [[Math/06_Rotations_Orientation/quaternion_fundamentals|Quaternion Fundamentals]]
- [[Math/06_Rotations_Orientation/rotation_arbitrary_axis|Rotation Around Arbitrary Axis]]
