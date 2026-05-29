---
title: "Slerp: The Smooth Spin Between Orientations"
tags:
  - math
  - level/Lv3
  - category/rotations_orientation
---

# Slerp: The Smooth Spin Between Orientations

> [!abstract] **The Concept in a Nutshell**
> **Spherical Linear Interpolation (Slerp)** smoothly blends between two quaternion orientations along the shortest arc on a 4D unit sphere, maintaining **constant angular velocity**. It's the gold standard for rotation interpolation in games — used for camera transitions, character turning, animation blending, and anything that needs to rotate smoothly from A to B. The cheaper alternative, **Nlerp** (normalized linear interpolation), is faster but has non-constant speed.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Cinematic Camera Transition**
> In a cutscene, the camera must smoothly rotate from looking at a castle gate (orientation $\mathbf{q}_A$) to looking up at a dragon perched on a tower (orientation $\mathbf{q}_B$) over exactly 3 seconds.
>
> Using **Lerp** on Euler angles? Disaster — the camera might spin the wrong way, pass through gimbal lock, or accelerate/decelerate unpredictably.
>
> Using **Slerp** on quaternions? Perfect — the camera traces the shortest rotation path at constant angular speed. Frame 0: looking at the gate. Frame 90 (at 30fps, halfway through): looking exactly between the gate and the dragon. Frame 180: looking at the dragon. The rotation feels buttery smooth.
>
> For gameplay (character turning to face a target), **Nlerp** is often preferred because it's faster and the slight speed variation is imperceptible, especially when combined with `RotateTowards` which clamps the step size.

---

## The Blueprint (Formula & Structure)

### Slerp Formula

For two unit quaternions $\mathbf{q}_0$ and $\mathbf{q}_1$ with interpolation parameter $t \in [0, 1]$:

$$\text{Slerp}(\mathbf{q}_0, \mathbf{q}_1, t) = \frac{\sin((1-t)\Omega)}{\sin\Omega}\mathbf{q}_0 + \frac{\sin(t\Omega)}{\sin\Omega}\mathbf{q}_1$$

Where $\Omega$ is the angle between the quaternions on the 4D unit sphere:

$$\cos\Omega = \mathbf{q}_0 \cdot \mathbf{q}_1 = w_0w_1 + x_0x_1 + y_0y_1 + z_0z_1$$

### Properties of Slerp

| Property | Description |
|---|---|
| **Constant angular velocity** | The rotation speed is the same throughout the interpolation |
| **Shortest path** | Always takes the shorter arc (with hemisphere correction) |
| **Torque-minimal** | Produces the most natural-looking rotation |
| **Unit quaternion output** | The result is always a valid unit quaternion |
| **Endpoint interpolation** | $\text{Slerp}(q_0, q_1, 0) = q_0$, $\text{Slerp}(q_0, q_1, 1) = q_1$ |

### Shortest Path (Hemisphere Correction)

Since $\mathbf{q}$ and $-\mathbf{q}$ represent the same rotation, Slerp could take the long way around the 4D sphere. To ensure the shortest path:

$$\text{If } \mathbf{q}_0 \cdot \mathbf{q}_1 < 0, \text{ negate one quaternion: } \mathbf{q}_1 \leftarrow -\mathbf{q}_1$$

This flips to the same hemisphere, guaranteeing $\Omega \leq 90°$ (which corresponds to a 3D rotation $\leq 180°$).

### Nlerp (Normalized Linear Interpolation)

A cheaper alternative — linearly interpolate, then normalize:

$$\text{Nlerp}(\mathbf{q}_0, \mathbf{q}_1, t) = \text{normalize}\left((1 - t)\mathbf{q}_0 + t\mathbf{q}_1\right)$$

| Aspect | Slerp | Nlerp |
|---|---|---|
| Speed (computation) | Slower (trig functions) | Faster (add + normalize) |
| Angular velocity | **Constant** | Variable (faster in middle) |
| Commutative | No | No |
| Quality | Perfect arc | Slight acceleration |
| Use when | Precise timing needed | Gameplay, frame-by-frame |

### When Quaternions Are Very Close ($\Omega \approx 0$)

As $\Omega \to 0$, $\sin\Omega \to 0$ and Slerp's division becomes numerically unstable. In this case, **fall back to Lerp** (or Nlerp):

$$\text{If } \Omega < \epsilon: \quad \text{result} \approx (1-t)\mathbf{q}_0 + t\mathbf{q}_1$$

### Lerp on Euler Angles vs Slerp on Quaternions

**Never** linearly interpolate Euler angles for rotation blending:

- Euler Lerp $(\alpha_0, \beta_0, \gamma_0) \to (\alpha_1, \beta_1, \gamma_1)$: can take wrong path (e.g., $350° \to 10°$ goes through $180°$ instead of the short $20°$ arc), passes through gimbal lock configurations, non-constant speed.
- Quaternion Slerp: always shortest path, no singularities, constant speed.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Walking Along a Globe's Surface**
> Imagine two cities on a globe. **Lerp** would dig a straight tunnel through the Earth between them. **Slerp** walks along the surface (the great circle arc) — it stays on the sphere.
>
> For quaternions, the "globe" is a 4D unit hypersphere. Each point on this sphere is a possible 3D orientation. Slerp traces the shortest surface path between two orientations, just like an airplane follows a great circle route between cities.
>
> **Nlerp** is like stretching a straight rubber band between the two points and then pushing each point back onto the sphere surface. It approximately follows the arc but the "pushback" makes it speed up in the middle (the rubber band is farthest from the surface at the midpoint).

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Smooth camera transitions and rotation blending
using UnityEngine;

public class CameraTransition : MonoBehaviour
{
    public Transform cameraTransform;
    public Transform lookAtGate;     // Target A
    public Transform lookAtDragon;   // Target B

    [Header("Transition Settings")]
    public float transitionDuration = 3f;
    public bool useSlerp = true; // Toggle Slerp vs Nlerp

    private Quaternion startRot;
    private Quaternion endRot;
    private float timer = 0f;
    private bool transitioning = false;

    void Update()
    {
        // Start transition with spacebar
        if (Input.GetKeyDown(KeyCode.Space) && !transitioning)
        {
            startRot = Quaternion.LookRotation(
                lookAtGate.position - cameraTransform.position);
            endRot = Quaternion.LookRotation(
                lookAtDragon.position - cameraTransform.position);

            // Hemisphere check — ensure shortest path
            if (Quaternion.Dot(startRot, endRot) < 0f)
            {
                endRot = new Quaternion(-endRot.x, -endRot.y,
                                         -endRot.z, -endRot.w);
            }

            timer = 0f;
            transitioning = true;
        }

        if (transitioning)
        {
            timer += Time.deltaTime;
            float t = Mathf.Clamp01(timer / transitionDuration);

            if (useSlerp)
            {
                // SLERP: Constant angular velocity
                cameraTransform.rotation = Quaternion.Slerp(
                    startRot, endRot, t);
            }
            else
            {
                // NLERP: Cheaper, slightly variable speed
                cameraTransform.rotation = NlerpManual(
                    startRot, endRot, t);
            }

            if (t >= 1f)
                transitioning = false;

            // Visualize angular speed (should be constant for Slerp)
            float angularSpeed = Quaternion.Angle(
                cameraTransform.rotation,
                Quaternion.Slerp(startRot, endRot,
                    Mathf.Clamp01(t + 0.01f))
            ) / 0.01f;
            Debug.Log($"t={t:F2} | Angular speed: {angularSpeed:F1} °/unit_t");
        }
    }

    /// <summary>
    /// Manual Nlerp: linearly interpolate then normalize.
    /// </summary>
    Quaternion NlerpManual(Quaternion a, Quaternion b, float t)
    {
        // Linear interpolation of components
        Quaternion raw = new Quaternion(
            a.x + (b.x - a.x) * t,
            a.y + (b.y - a.y) * t,
            a.z + (b.z - a.z) * t,
            a.w + (b.w - a.w) * t
        );

        // Normalize to ensure unit quaternion
        float mag = Mathf.Sqrt(raw.x * raw.x + raw.y * raw.y
                              + raw.z * raw.z + raw.w * raw.w);
        float inv = 1f / mag;
        return new Quaternion(raw.x * inv, raw.y * inv,
                              raw.z * inv, raw.w * inv);
    }

    /// <summary>
    /// Manual Slerp implementation (educational).
    /// </summary>
    Quaternion SlerpManual(Quaternion a, Quaternion b, float t)
    {
        float dot = a.x * b.x + a.y * b.y + a.z * b.z + a.w * b.w;

        // Hemisphere correction
        if (dot < 0f)
        {
            b = new Quaternion(-b.x, -b.y, -b.z, -b.w);
            dot = -dot;
        }

        // If quaternions are very close, fall back to Nlerp
        if (dot > 0.9995f)
            return NlerpManual(a, b, t);

        float omega = Mathf.Acos(dot);
        float sinOmega = Mathf.Sin(omega);

        float weightA = Mathf.Sin((1f - t) * omega) / sinOmega;
        float weightB = Mathf.Sin(t * omega) / sinOmega;

        return new Quaternion(
            weightA * a.x + weightB * b.x,
            weightA * a.y + weightB * b.y,
            weightA * a.z + weightB * b.z,
            weightA * a.w + weightB * b.w
        );
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Write the Slerp formula for quaternions. :: $\text{Slerp}(\mathbf{q}_0, \mathbf{q}_1, t) = \frac{\sin((1-t)\Omega)}{\sin\Omega}\mathbf{q}_0 + \frac{\sin(t\Omega)}{\sin\Omega}\mathbf{q}_1$, where $\cos\Omega = \mathbf{q}_0 \cdot \mathbf{q}_1$.
- What is the key advantage of Slerp over Nlerp? :: Slerp maintains **constant angular velocity** — the rotation speed is uniform throughout the interpolation. Nlerp is faster to compute but accelerates in the middle of the transition.
- Why must you check the quaternion dot product sign before interpolating? :: Because $\mathbf{q}$ and $-\mathbf{q}$ represent the same rotation. If $\mathbf{q}_0 \cdot \mathbf{q}_1 < 0$, the interpolation would take the **long way** around the 4D sphere (> 180° rotation). Negating one quaternion ensures the shortest path.
- When should you fall back from Slerp to Lerp/Nlerp? :: When the two quaternions are **very close** ($\Omega \approx 0$, i.e., $\cos\Omega \approx 1$). At small angles, $\sin\Omega \to 0$ and Slerp's division becomes numerically unstable. Lerp is a fine approximation here.
- In gameplay, when would you prefer Nlerp over Slerp? :: When performance matters more than precision — e.g., rotating 100 enemies per frame to face the player. Nlerp avoids expensive trig functions and the slight speed variation is imperceptible in frame-by-frame gameplay rotations.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting the hemisphere check before Slerp, causing objects to rotate the "long way" (300° instead of 60°) seemingly at random.
> - **The Fix:** Always check `Quaternion.Dot(q0, q1)`. If negative, negate one of the quaternions before interpolating. Unity's built-in `Quaternion.Slerp` handles this automatically, but manual implementations often forget.
> - **Why:** Due to double cover, Slerp doesn't know which path you want unless you explicitly choose the correct hemisphere.

> [!danger] **Watch Out!**
> - **The Mistake:** Using `Quaternion.Slerp(current, target, Time.deltaTime * speed)` and expecting linear progress. The object slows down asymptotically and never quite reaches the target.
> - **The Fix:** Either track a separate `t` parameter that goes from 0 to 1, or use `Quaternion.RotateTowards(current, target, degreesPerSecond * Time.deltaTime)` for constant-speed rotation.
> - **Why:** `Slerp(current, target, 0.1)` always moves 10% of the **remaining** distance — it's an exponential ease-out, not linear progress. It looks smooth but mathematically never converges.

---

## Related Topics
- [[Math/06_Rotations_Orientation/quaternion_rotations|Quaternion Rotations]]
- [[Math/08_Curves_Interpolation/lerp_fundamentals|Lerp Fundamentals]]
- [[Math/08_Curves_Interpolation/easing_functions|Easing Functions]]
