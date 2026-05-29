---
title: "The Unit Circle & Radians: Scaling and Rotation Fundamentals"
tags:
  - math
  - level/Lv1
  - category/geometry_trigonometry
---

# The Unit Circle & Radians: Scaling and Rotation Fundamentals

> [!abstract] **The Concept in a Nutshell**
> The unit circle is a circle with a radius of $1.0$ centered at the origin $(0, 0)$. In mathematics and game physics, angles are typically measured in **radians** rather than degrees. A radian represents the angle created when the arc length along a circle is equal to the radius. While humans think in degrees ($360^\circ$ to a full circle), computer engines calculate using radians ($2\pi$ to a full circle). Bridging these two measurements is crucial to avoiding bugs in rotation and movement logic.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The "Fast Spinning Character" Bug**
> You write a simple script to rotate a camera using mouse input:
> ```csharp
> transform.Rotate(Vector3.up * mouseX); // expectation: Rotate by degrees
> ```
> This works fine because `Transform.Rotate` expects degrees. But then you try to calculate character offset manually:
> ```csharp
> float targetX = Mathf.Cos(currentRotationDegrees); // ❌ BUG!
> ```
> The player starts spinning around the origin at hyperspeed. This happens because `Mathf.Cos` and `Mathf.Sin` expect angles in **radians**, not degrees! Passing $90$ degrees into `Mathf.Cos` actually calculates the cosine of $90$ radians (equivalent to about $5{,}156$ degrees), leading to erratic calculations.
>
> The correct code requires converting degrees to radians first:
> ```csharp
> float targetX = Mathf.Cos(currentRotationDegrees * Mathf.Deg2Rad);
> ```

---

## The Blueprint (Formula & Structure)

### Radians vs. Degrees
A full circle is $360^\circ$, which is equivalent to $2\pi$ radians.

- **Degrees to Radians:**
  $$\text{radians} = \text{degrees} \times \frac{\pi}{180^\circ}$$
  In Unity: `float rad = deg * Mathf.Deg2Rad;`

- **Radians to Degrees:**
  $$\text{degrees} = \text{radians} \times \frac{180^\circ}{\pi}$$
  In Unity: `float deg = rad * Mathf.Rad2Deg;`

### Key Angles Reference Table

| Degrees | Radians | Cosine ($\cos$) | Sine ($\sin$) | Direction Vector |
|---------|---------|-----------------|---------------|------------------|
| $0^\circ$ | $0$ | $1$ | $0$ | $(1, 0)$ |
| $45^\circ$ | $\frac{\pi}{4} \approx 0.785$ | $\frac{\sqrt{2}}{2} \approx 0.707$ | $\frac{\sqrt{2}}{2} \approx 0.707$ | $(0.707, 0.707)$ |
| $90^\circ$ | $\frac{\pi}{2} \approx 1.570$ | $0$ | $1$ | $(0, 1)$ |
| $180^\circ$ | $\pi \approx 3.141$ | $-1$ | $0$ | $(-1, 0)$ |
| $270^\circ$ | $\frac{3\pi}{2} \approx 4.712$ | $0$ | $-1$ | $(0, -1)$ |
| $360^\circ$ | $2\pi \approx 6.283$ | $1$ | $0$ | $(1, 0)$ |

### Angle Wrapping (Normalization)
Angles wrap around every $360^\circ$ ($2\pi$ radians). Keeping angles inside the range $[0, 360^\circ]$ or $[-\pi, \pi]$ prevents overflow and simplifies interpolation (LERP):
- **Wrapping to $[0, 2\pi]$ (radians):** $\theta \pmod{2\pi}$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Radians are Walking the Circle**
> Think of radians as the actual distance you would walk along the edge of a circle of radius $1$.
> - If you walk $1$ unit of distance along the circle's circumference, the angle you've traversed is exactly $1$ radian.
> - If you walk halfway around the circle, you've walked $\pi \approx 3.141$ units.
> - If you walk all the way around, you've walked $2\pi \approx 6.283$ units.
> 
> Radians connect linear distance (arc length) directly to angular rotation, which makes them much more natural for physics and rendering calculations than degrees.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Demonstrating Radian/Degree Conversion and Angle Wrapping
using UnityEngine;

public class RotationUtil : MonoBehaviour
{
    // Wrap angle in degrees to range [0, 360)
    public static float WrapAngleDegrees(float angle)
    {
        angle = angle % 360f;
        if (angle < 0)
            angle += 360f;
        return angle;
    }

    // Wrap angle in radians to range [-pi, pi]
    public static float WrapAngleRadians(float angle)
    {
        while (angle > Mathf.PI) angle -= 2f * Mathf.PI;
        while (angle < -Mathf.PI) angle += 2f * Mathf.PI;
        return angle;
    }

    void Start()
    {
        float deg = 90f;
        // Convert to radians for trigonometric functions
        float rad = deg * Mathf.Deg2Rad;

        float sinValue = Mathf.Sin(rad); // Correct: returns 1.0f
        float cosValue = Mathf.Cos(rad); // Correct: returns 0.0f (approx due to float)

        Debug.Log($"90 Degrees: Sin={sinValue:F2}, Cos={cosValue:F2}");
        
        // Convert back
        float backToDeg = rad * Mathf.Rad2Deg; // 90f
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the equivalent of $180^\circ$ in radians? :: $\pi$ radians ($\approx 3.14159$).
- How do you convert an angle in degrees to radians? :: Multiply by $\frac{\pi}{180^\circ}$ (in Unity: `angle * Mathf.Deg2Rad`).
- Why do computer processors and engines use radians instead of degrees for trig functions? :: Radians represent real geometric arc length and simplify Taylor series expansion calculations in the hardware CPU/GPU.
- What is the direction vector for an angle of $90^\circ$ (pointing straight up in 2D)? :: $(0, 1)$ because $\cos(90^\circ) = 0$ and $\sin(90^\circ) = 1$.
- How many degrees is $1$ radian approximately? :: $\approx 57.29^\circ$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Passing raw degrees to `Mathf.Sin` or `Mathf.Cos` functions.
> - **The Fix:** Always multiply your degree variable by `Mathf.Deg2Rad` inside trig functions: `Mathf.Sin(degrees * Mathf.Deg2Rad)`.
> - **Why:** Standard math libraries are strictly built around radians. Passing degrees changes the logic and generates bizarre movement vectors.

---

## Related Topics
- [[Math/02_Geometry_Trigonometry/trigonometric_functions|Trigonometric Functions]]
- [[Math/02_Geometry_Trigonometry/trig_applications|Trigonometry in Game Dev]]
- [[Math/06_Rotations_Orientation/rotation_matrices_2d|2D Rotation Matrices]]
