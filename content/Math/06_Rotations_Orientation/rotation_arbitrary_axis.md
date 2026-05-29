---
title: "Rotation Around an Arbitrary Axis: Rodrigues' Formula"
tags:
  - math
  - level/Lv3
  - category/rotations_orientation
---

# Rotation Around an Arbitrary Axis: Rodrigues' Formula

> [!abstract] **The Concept in a Nutshell**
> Rodrigues' rotation formula rotates a vector $\mathbf{v}$ around any unit axis $\hat{\mathbf{k}}$ by angle $\theta$. It decomposes $\mathbf{v}$ into components parallel and perpendicular to the axis, leaves the parallel part alone, and spins the perpendicular part. This is more general than Euler angles (which only rotate around coordinate axes) and forms the bridge to quaternions and axis-angle representation.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Medieval Castle Gate and Spinning Windmill**
> You're building a medieval sim. The castle's drawbridge must rotate around its **hinge axis** — a horizontal bar along the $x$-axis at the top of the gate. The windmill blades spin around an axis that's tilted $30°$ from vertical. Neither of these is aligned with a coordinate axis.
>
> You can't simply use `Quaternion.Euler(angle, 0, 0)` for the windmill — its rotation axis is $(0.5, 0.866, 0)$, pointing along the tilted axle. Rodrigues' formula (or its quaternion equivalent, `Quaternion.AngleAxis(angle, axisDirection)`) lets you define the rotation axis as **any direction in space**, then rotate objects around it smoothly.
>
> Even a simple door uses this: the door hinge is a vertical axis at the door's edge, not the center. You combine axis-angle rotation with a pivot offset.

---

## The Blueprint (Formula & Structure)

### Rodrigues' Rotation Formula

Given:
- $\mathbf{v}$: the vector to rotate
- $\hat{\mathbf{k}}$: the unit rotation axis ($\|\hat{\mathbf{k}}\| = 1$)
- $\theta$: the rotation angle (counterclockwise when viewed from the tip of $\hat{\mathbf{k}}$)

$$\mathbf{v}' = \mathbf{v}\cos\theta + (\hat{\mathbf{k}} \times \mathbf{v})\sin\theta + \hat{\mathbf{k}}(\hat{\mathbf{k}} \cdot \mathbf{v})(1 - \cos\theta)$$

### Derivation Intuition

Decompose $\mathbf{v}$ into parts **parallel** and **perpendicular** to the axis:

$$\mathbf{v}_\parallel = (\hat{\mathbf{k}} \cdot \mathbf{v})\hat{\mathbf{k}} \quad \text{(projection onto axis)}$$
$$\mathbf{v}_\perp = \mathbf{v} - \mathbf{v}_\parallel \quad \text{(rejection — the part to rotate)}$$

The perpendicular component lives in a 2D plane normal to $\hat{\mathbf{k}}$. In this plane, we need a second basis vector:

$$\mathbf{w} = \hat{\mathbf{k}} \times \mathbf{v} = \hat{\mathbf{k}} \times \mathbf{v}_\perp$$

This is perpendicular to both $\hat{\mathbf{k}}$ and $\mathbf{v}_\perp$, and has the same magnitude as $\mathbf{v}_\perp$.

Now rotate in the $(\mathbf{v}_\perp, \mathbf{w})$ plane by $\theta$:

$$\mathbf{v}_\perp' = \mathbf{v}_\perp \cos\theta + \mathbf{w}\sin\theta$$

The parallel part doesn't change:

$$\mathbf{v}' = \mathbf{v}_\parallel + \mathbf{v}_\perp' = \hat{\mathbf{k}}(\hat{\mathbf{k}} \cdot \mathbf{v}) + (\mathbf{v} - \hat{\mathbf{k}}(\hat{\mathbf{k}} \cdot \mathbf{v}))\cos\theta + (\hat{\mathbf{k}} \times \mathbf{v})\sin\theta$$

Collecting terms gives the final formula.

### Matrix Form

Rodrigues' formula can be written as a rotation matrix:

$$\mathbf{R}(\hat{\mathbf{k}}, \theta) = \mathbf{I}\cos\theta + (1 - \cos\theta)\hat{\mathbf{k}}\hat{\mathbf{k}}^T + \sin\theta[\hat{\mathbf{k}}]_\times$$

Where $[\hat{\mathbf{k}}]_\times$ is the **skew-symmetric cross-product matrix**:

$$[\hat{\mathbf{k}}]_\times = \begin{bmatrix} 0 & -k_z & k_y \\ k_z & 0 & -k_x \\ -k_y & k_x & 0 \end{bmatrix}$$

And $\hat{\mathbf{k}}\hat{\mathbf{k}}^T$ is the outer product:

$$\hat{\mathbf{k}}\hat{\mathbf{k}}^T = \begin{bmatrix} k_x^2 & k_xk_y & k_xk_z \\ k_xk_y & k_y^2 & k_yk_z \\ k_xk_z & k_yk_z & k_z^2 \end{bmatrix}$$

### Expanded Matrix

$$\mathbf{R} = \begin{bmatrix} \cos\theta + k_x^2(1-\cos\theta) & k_xk_y(1-\cos\theta) - k_z\sin\theta & k_xk_z(1-\cos\theta) + k_y\sin\theta \\ k_yk_x(1-\cos\theta) + k_z\sin\theta & \cos\theta + k_y^2(1-\cos\theta) & k_yk_z(1-\cos\theta) - k_x\sin\theta \\ k_zk_x(1-\cos\theta) - k_y\sin\theta & k_zk_y(1-\cos\theta) + k_x\sin\theta & \cos\theta + k_z^2(1-\cos\theta) \end{bmatrix}$$

### Axis-Angle Representation

An orientation can be compactly stored as:
- **Axis:** unit vector $\hat{\mathbf{k}} = (k_x, k_y, k_z)$ — 3 values (2 DOF since it's unit length)
- **Angle:** scalar $\theta$ — 1 value

Total: **4 numbers** for 3 DOF, same as a quaternion. In fact, a unit quaternion $\mathbf{q} = (\cos\frac{\theta}{2},\; \hat{\mathbf{k}}\sin\frac{\theta}{2})$ encodes exactly this axis-angle pair.

### Special Cases

| Axis $\hat{\mathbf{k}}$ | Result |
|---|---|
| $(1, 0, 0)$ | $\mathbf{R}_x(\theta)$ — standard pitch rotation |
| $(0, 1, 0)$ | $\mathbf{R}_y(\theta)$ — standard yaw rotation |
| $(0, 0, 1)$ | $\mathbf{R}_z(\theta)$ — standard roll rotation |
| $\theta = 0$ | $\mathbf{R} = \mathbf{I}$ — identity, no rotation |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Kebab Skewer**
> Imagine sticking a skewer through a piece of meat at an arbitrary angle. The skewer is your **rotation axis** $\hat{\mathbf{k}}$. Now spin the meat around the skewer.
>
> - The part of the meat directly *on* the skewer (the parallel component) doesn't move.
> - Everything else (the perpendicular component) traces circles around the skewer.
> - The radius of each circle depends on how far the point is from the skewer.
>
> Rodrigues' formula says: "Split the vector into the part along the skewer (keep it) and the part across the skewer (spin it in the perpendicular plane)."
>
> The cross product $\hat{\mathbf{k}} \times \mathbf{v}$ gives you the "sideways" direction in that perpendicular plane — it's the second axis you need to define the circular motion.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Rotating objects around arbitrary axes
using UnityEngine;

public class ArbitraryAxisRotation : MonoBehaviour
{
    [Header("Windmill Settings")]
    public Transform windmillBlades;
    public float spinSpeed = 90f; // degrees per second

    // The windmill axle is tilted 30° from vertical
    private Vector3 axleDirection;

    [Header("Door Settings")]
    public Transform door;
    public Transform hingePoint; // Empty GameObject at the hinge position
    public float doorAngle = 0f;
    public float doorSpeed = 120f;

    void Start()
    {
        // Windmill axle: tilted 30° from Y toward Z
        axleDirection = Quaternion.Euler(30f, 0f, 0f) * Vector3.up;
        axleDirection.Normalize();
    }

    void Update()
    {
        // === Windmill: Continuous rotation around tilted axis ===
        // Unity's AngleAxis uses Rodrigues' formula internally
        windmillBlades.Rotate(axleDirection, spinSpeed * Time.deltaTime,
                              Space.World);

        // === Door: Open/close around hinge axis ===
        bool openDoor = Input.GetKey(KeyCode.E);
        float targetAngle = openDoor ? 90f : 0f;
        doorAngle = Mathf.MoveTowards(doorAngle, targetAngle,
                                       doorSpeed * Time.deltaTime);

        // Door hinge is vertical (Y-axis) at the hinge point
        Vector3 hingeAxis = Vector3.up;

        // Rotate the door around the hinge point
        door.position = hingePoint.position;
        door.rotation = Quaternion.AngleAxis(doorAngle, hingeAxis)
                      * Quaternion.identity; // base rotation
        // Offset the door center from the hinge
        Vector3 doorOffset = new Vector3(0.5f, 0f, 0f); // half door width
        door.position += door.rotation * doorOffset;

        // === Manual Rodrigues' formula (educational) ===
        Vector3 v = new Vector3(1f, 0f, 0f);       // vector to rotate
        Vector3 k = Vector3.up;                     // axis (Y-up)
        float theta = 45f * Mathf.Deg2Rad;          // angle

        Vector3 rotated = RodriguesRotate(v, k, theta);
        Debug.Log($"Rodrigues result: {rotated}");
        // Expected: (cos45, 0, -sin45) ≈ (0.707, 0, -0.707)
    }

    /// <summary>
    /// Rodrigues' rotation formula: rotate v around unit axis k by theta radians.
    /// </summary>
    Vector3 RodriguesRotate(Vector3 v, Vector3 k, float theta)
    {
        float cosT = Mathf.Cos(theta);
        float sinT = Mathf.Sin(theta);

        // v' = v*cos(θ) + (k × v)*sin(θ) + k*(k·v)*(1-cos(θ))
        return v * cosT
             + Vector3.Cross(k, v) * sinT
             + k * Vector3.Dot(k, v) * (1f - cosT);
    }

    /// <summary>
    /// Build the 3x3 rotation matrix from Rodrigues' formula.
    /// </summary>
    Matrix4x4 RodriguesMatrix(Vector3 k, float theta)
    {
        float c = Mathf.Cos(theta);
        float s = Mathf.Sin(theta);
        float t = 1f - c;

        // R = I*cos(θ) + (1-cos(θ))*k*kT + sin(θ)*[k]×
        Matrix4x4 m = Matrix4x4.identity;
        m[0, 0] = c + k.x * k.x * t;
        m[0, 1] = k.x * k.y * t - k.z * s;
        m[0, 2] = k.x * k.z * t + k.y * s;
        m[1, 0] = k.y * k.x * t + k.z * s;
        m[1, 1] = c + k.y * k.y * t;
        m[1, 2] = k.y * k.z * t - k.x * s;
        m[2, 0] = k.z * k.x * t - k.y * s;
        m[2, 1] = k.z * k.y * t + k.x * s;
        m[2, 2] = c + k.z * k.z * t;

        return m;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- State Rodrigues' rotation formula. :: $\mathbf{v}' = \mathbf{v}\cos\theta + (\hat{\mathbf{k}} \times \mathbf{v})\sin\theta + \hat{\mathbf{k}}(\hat{\mathbf{k}} \cdot \mathbf{v})(1 - \cos\theta)$. It rotates vector $\mathbf{v}$ around unit axis $\hat{\mathbf{k}}$ by angle $\theta$.
- What are the three components in Rodrigues' formula and what do they represent? :: 1) $\mathbf{v}\cos\theta$ — the original vector scaled down. 2) $(\hat{\mathbf{k}} \times \mathbf{v})\sin\theta$ — the perpendicular "swing" component. 3) $\hat{\mathbf{k}}(\hat{\mathbf{k}} \cdot \mathbf{v})(1-\cos\theta)$ — the parallel component restored after the cosine scaling removed it.
- What is the skew-symmetric cross-product matrix $[\hat{\mathbf{k}}]_\times$? :: A $3 \times 3$ matrix such that $[\hat{\mathbf{k}}]_\times \mathbf{v} = \hat{\mathbf{k}} \times \mathbf{v}$. It's $\begin{bmatrix} 0 & -k_z & k_y \\ k_z & 0 & -k_x \\ -k_y & k_x & 0 \end{bmatrix}$.
- How does axis-angle relate to quaternions? :: A quaternion $\mathbf{q} = (\cos\frac{\theta}{2},\; \hat{\mathbf{k}}\sin\frac{\theta}{2})$ encodes exactly the same axis-angle rotation. The half-angle arises from the quaternion double-cover of 3D rotations.
- In Unity, what function performs axis-angle rotation? :: `Quaternion.AngleAxis(angleDegrees, axis)` creates a quaternion from an axis-angle pair, using Rodrigues' formula internally.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to normalize the axis vector $\hat{\mathbf{k}}$ before applying Rodrigues' formula, leading to incorrect rotation magnitudes and shearing.
> - **The Fix:** Always ensure $\|\hat{\mathbf{k}}\| = 1$. Call `.normalized` or `Vector3.Normalize()` before passing the axis to the formula.
> - **Why:** The formula assumes $\hat{\mathbf{k}}$ is a unit vector. A non-unit axis will scale the parallel component incorrectly, producing a transform that isn't a pure rotation.

> [!danger] **Watch Out!**
> - **The Mistake:** Confusing the rotation direction. Is positive $\theta$ clockwise or counterclockwise?
> - **The Fix:** Rodrigues' formula uses the **right-hand rule**: curl the fingers of your right hand in the direction of positive rotation, and your thumb points along $\hat{\mathbf{k}}$. Unity's `AngleAxis` follows this convention.
> - **Why:** If you negate the axis, the rotation reverses. If you accidentally flip the axis direction, your door opens the wrong way.

---

## Related Topics
- [[Math/06_Rotations_Orientation/euler_angles|Euler Angles]]
- [[Math/06_Rotations_Orientation/quaternion_fundamentals|Quaternion Fundamentals]]
- [[Math/04_Matrices_Transforms/linear_transformations|Linear Transformations]]
