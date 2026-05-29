---
title: "Trigonometric Functions: Triangles, Ratios, and Angle Calculations"
tags:
  - math
  - level/Lv1
  - category/geometry_trigonometry
---

# Trigonometric Functions: Triangles, Ratios, and Angle Calculations

> [!abstract] **The Concept in a Nutshell**
> Trigonometry is the study of relationships between the sides and angles of triangles. In game development, trigonometric functions — Sine ($\sin$), Cosine ($\cos$), and Tangent ($\tan$) — act as bridges converting angles into direction vectors, and vice versa. They are the mathematical machinery behind character aiming, projectile tracking, and player rotation.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Aiming a Turret at the Player**
> You have a 2D defense turret located at $T(x_t, y_t)$ and the player is moving at $P(x_p, y_p)$. To make the turret rotate and point directly at the player, you must calculate the angle $\theta$ between the turret's default forward direction and the player's direction.
>
> The delta coordinates are $\Delta x = x_p - x_t$ and $\Delta y = y_p - y_t$. By treating this offset as a right triangle, we can use the inverse tangent function:
> $$\theta = \arctan\left(\frac{\Delta y}{\Delta x}\right)$$
>
> In game development, standard $\arctan$ fails if $\Delta x = 0$ (division by zero) and cannot tell if the player is in front or behind the turret. To solve this, game engines provide a special function: **$\text{atan2}(y, x)$**, which handles the division, signs, and division-by-zero cases automatically.

---

## The Blueprint (Formula & Structure)

### SOH-CAH-TOA (Right Triangle Ratios)
For a right triangle with an angle $\theta$, opposite side ($O$), adjacent side ($A$), and hypotenuse ($H$):

$$\sin\theta = \frac{\text{Opposite}}{\text{Hypotenuse}} \quad \text{(SOH)}$$
$$\cos\theta = \frac{\text{Adjacent}}{\text{Hypotenuse}} \quad \text{(CAH)}$$
$$\tan\theta = \frac{\text{Opposite}}{\text{Adjacent}} \quad \text{(TOA)}$$

```
          /|
         / |
     H  /  |  O (Opposite)
       /   |
      /θ___|
        A (Adjacent)
```

### Inverse Trigonometric Functions
Used to find an angle when the side ratio is known:
- $\arcsin(x)$ (or $\sin^{-1}$): Returns angle $\theta$ given $\frac{O}{H}$. Range: $[-\pi/2, \pi/2]$.
- $\arccos(x)$ (or $\cos^{-1}$): Returns angle $\theta$ given $\frac{A}{H}$. Range: $[0, \pi]$.
- $\arctan(x)$ (or $\tan^{-1}$): Returns angle $\theta$ given $\frac{O}{A}$. Range: $(-\pi/2, \pi/2)$.
- **$\text{atan2}(y, x)$**: Crucial game dev utility. Returns the angle in radians between the positive X-axis and the point $(x, y)$. Range: $[-\pi, \pi]$.

### Essential Game Dev Trig Identities
1. **Pythagorean Identity:**
   $$\sin^2\theta + \cos^2\theta = 1$$
2. **Tangent Definition:**
   $$\tan\theta = \frac{\sin\theta}{\cos\theta}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Projection Cast**
> Imagine $\sin\theta$ and $\cos\theta$ as the coordinates of a laser point projected on a unit circle.
> - $\cos\theta$ measures how far right/left the point is (horizontal projection).
> - $\sin\theta$ measures how far up/down the point is (vertical projection).
> - $\tan\theta$ measures the slope of the line from the origin to the point.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Aiming a turret and converting angles to direction
using UnityEngine;

public class TurretAiming : MonoBehaviour
{
    public Transform player;
    public float rotationSpeed = 5f;

    void Update()
    {
        // Vector pointing from turret to player
        Vector3 targetDirection = player.position - transform.position;

        // Calculate 2D angle (ignoring Z for simplicity in 2D gameplay)
        // Note: atan2 takes (y, x)
        float angleRad = Mathf.Atan2(targetDirection.y, targetDirection.x);
        float angleDeg = angleRad * Mathf.Rad2Deg;

        // Apply rotation to turret
        Quaternion targetRotation = Quaternion.Euler(0, 0, angleDeg);
        transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * rotationSpeed);
    }

    // Function to calculate a direction vector from a given angle (in degrees)
    public Vector2 DirectionFromAngle(float angleDegrees)
    {
        float angleRad = angleDegrees * Mathf.Deg2Rad;
        // Direction vector = (cos(theta), sin(theta))
        return new Vector2(Mathf.Cos(angleRad), Mathf.Sin(angleRad));
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does SOH-CAH-TOA stand for? :: **S**ine = **O**pposite / **H**ypotenuse, **C**osine = **A**djacent / **H**ypotenuse, **T**angent = **O**pposite / **A**djacent.
- Why is `Mathf.Atan2(y, x)` preferred over `Mathf.Atan(y / x)` in game dev? :: `Atan2` handles the division-by-zero case when $x = 0$, and it identifies the correct quadrant (since it takes the signs of $x$ and $y$ separately), returning the full $[-\pi, \pi]$ range.
- How do you construct a 2D direction unit vector from an angle $\theta$ (in radians)? :: $\vec{u} = (\cos\theta, \sin\theta)$.
- What is the range of output values for the Sine and Cosine functions? :: $[-1, 1]$.
- What is the value of $\sin^2\theta + \cos^2\theta$? :: $1$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Passing parameters to `Atan2` in the wrong order: `Atan2(x, y)`.
> - **The Fix:** Always pass the vertical component first: `Atan2(y, x)`.
> - **Why:** Math libraries globally define `atan2` as taking the vertical parameter first and horizontal second. Reversing them rotates your resulting angle by $90^\circ$ and flips the axes.

---

## Related Topics
- [[Math/02_Geometry_Trigonometry/unit_circle_radians|The Unit Circle & Radians]]
- [[Math/02_Geometry_Trigonometry/trig_applications|Trigonometry in Game Dev]]
- [[Math/06_Rotations_Orientation/rotation_matrices_2d|2D Rotation Matrices]]
