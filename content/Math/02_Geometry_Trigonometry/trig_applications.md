---
title: "Trigonometry in Game Dev: Wave Patterns, Oscillation, and Orbiting"
tags:
  - math
  - level/Lv2
  - category/geometry_trigonometry
---

# Trigonometry in Game Dev: Wave Patterns, Oscillation, and Orbiting

> [!abstract] **The Concept in a Nutshell**
> Beyond calculating simple angles, trigonometric functions are dynamic forces in game loop execution. Because sine and cosine waves repeat infinitely between $-1$ and $1$, they are perfect for creating periodic patterns: floating/bobbing items, orbiting objects, oscillating lights, pendulum swing simulations, and wave-like enemy movement patterns.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The Bobbing Gold Coin**
> You are designing a platformer game. To make collectibles like coins or keys visually stand out, they shouldn't sit statically on the floor; they should float up and down gently and rotate.
>
> To implement this, you can feed game time ($t$) into the sine function. Because $\sin(t)$ oscillates smoothly between $-1$ and $1$, multiplying the output by a small scalar creates a perfect, smooth vertical bobbing motion:
> $$\text{offset}_y = \sin(\text{time} \times \text{speed}) \times \text{amplitude}$$
>
> We can combine this vertical oscillation with a constant angular rotation to make the coin rotate while bobbing, creating a classic, polished arcade effect.

---

## The Blueprint (Formula & Structure)

### 1. Simple Oscillation
To bounce a value between two bounds over time:
$$y = y_0 + A \sin(\omega t + \phi)$$
- $y_0$: Base position (midpoint).
- $A$: **Amplitude** (how far it moves from midpoint).
- $\omega$: **Angular frequency** (speed of oscillation, where period $T = \frac{2\pi}{\omega}$).
- $t$: Current game time.
- $\phi$: **Phase shift** (starting point offset).

### 2. Parametric Circular Motion (Orbiting)
To make an object orbit a pivot point $P_0(x_0, y_0)$ at a radius $R$:
$$x = x_0 + R \cos(\theta)$$
$$y = y_0 + R \sin(\theta)$$
Where angle $\theta = \text{speed} \times t$.

### 3. Lissajous Curves
By choosing different frequencies for the X and Y axes, you can generate complex paths (great for boss movement paths or particle trajectories):
$$x = A \cos(a t)$$
$$y = B \sin(b t)$$

### 4. Wave Interference
Adding multiple sine waves with different frequencies and amplitudes together generates natural-looking terrain height maps, water ripples, or camera shake effects.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Carousel Hanger**
> Think of sine and cosine waves as a carousel wheel rotating at a constant speed. 
> - If you look at the carousel from the **side** (vertical only), you see the horses moving up and down in a smooth sine wave.
> - If you look at it from the **front** (horizontal only), you see them moving left and right in a cosine wave.
> - Combined, they form a perfect, continuous circle.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Periodic movement, circular orbits, and bobbing
using UnityEngine;

public class PeriodicMovement : MonoBehaviour
{
    public enum MovementType { Bobbing, Orbit, Lissajous }
    public MovementType type;

    [Header("General Settings")]
    public float speed = 2f;
    public float amplitude = 1f;

    [Header("Orbit Settings")]
    public Transform orbitPivot;
    public float orbitRadius = 3f;

    private Vector3 startPosition;

    void Start()
    {
        startPosition = transform.position;
    }

    void Update()
    {
        float t = Time.time * speed;

        switch (type)
        {
            case MovementType.Bobbing:
                // Gentle up and down float
                float newY = startPosition.y + Mathf.Sin(t) * amplitude;
                transform.position = new Vector3(startPosition.x, newY, startPosition.z);
                break;

            case MovementType.Orbit:
                if (orbitPivot != null)
                {
                    // Orbit around pivot in the XZ plane
                    float x = orbitPivot.position.x + orbitRadius * Mathf.Cos(t);
                    float z = orbitPivot.position.z + orbitRadius * Mathf.Sin(t);
                    transform.position = new Vector3(x, transform.position.y, z);
                }
                break;

            case MovementType.Lissajous:
                // Figure-8 pattern
                float lx = startPosition.x + amplitude * Mathf.Sin(2f * t);
                float ly = startPosition.y + amplitude * Mathf.Sin(3f * t);
                transform.position = new Vector3(lx, ly, transform.position.z);
                break;
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How do you calculate the horizontal and vertical offsets for a circular orbit of radius $R$? :: $x = R \cos\theta$ and $y = R \sin\theta$.
- In the formula $y = A \sin(\omega t)$, what does $A$ control? :: The **amplitude**, which dictates the height (maximum displacement) of the wave.
- What is a Lissajous curve? :: A curve generated by parametric equations where X and Y coordinates oscillate at different frequencies, forming complex winding paths.
- Why is a sine wave useful for camera shake? :: It provides smooth, continuous acceleration and deceleration, preventing jerky motions while staying bounded between $[-1, 1]$.
- How do you increase the speed of an oscillation represented by $\sin(t)$? :: Multiply the time parameter $t$ by a speed coefficient (frequency): $\sin(t \times \text{speed})$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `Time.deltaTime` inside sine functions instead of cumulative time `Time.time`.
> - **The Fix:** Always multiply speed by absolute cumulative time (like `Time.time` in Unity or game elapsed time).
> - **Why:** `Time.deltaTime` is the duration of the *single last frame* (e.g., $0.016$s). If you pass it directly, your oscillation will freeze in place because the input value doesn't accumulate across frames.

---

## Related Topics
- [[Math/02_Geometry_Trigonometry/trigonometric_functions|Trigonometric Functions]]
- [[Math/02_Geometry_Trigonometry/unit_circle_radians|The Unit Circle & Radians]]
- [[Math/08_Curves_Interpolation/easing_functions|Easing Functions]]
