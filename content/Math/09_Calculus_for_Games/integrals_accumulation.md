---
title: "Integrals & Accumulation — Adding Up the Pieces of Motion"
tags:
  - math
  - level/Lv3
  - category/calculus
---

# Integrals & Accumulation: Adding Up the Pieces of Motion

> [!abstract] **The Concept in a Nutshell**
> Integration is the reverse of differentiation: where derivatives break a quantity into its rate of change, integrals *accumulate* those rates back into totals. Position is the integral of velocity over time. Total damage is the integral of damage-per-second over a duration. Every game loop that adds `velocity * deltaTime` to position is performing numerical integration — understanding the math helps you do it correctly.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Accumulating Poison Damage Over Time**
> Your RPG has a "Venomous Bite" debuff that deals damage at a rate that *decays* exponentially: $\text{dps}(t) = 20 e^{-0.5t}$ (20 DPS at onset, fading over time). The total damage dealt after $T$ seconds is the integral:
> $$D = \int_0^T 20 e^{-0.5t}\, dt = \left[-40 e^{-0.5t}\right]_0^T = 40(1 - e^{-0.5T})$$
> After 4 seconds: $D = 40(1 - e^{-2}) \approx 34.6$ total damage. Without integration, you can't predict the total — and your game balance spreadsheet becomes guesswork.

---

## The Blueprint (Formula & Structure)

### The Integral — Formal Definition

$$\int_a^b f(t)\, dt = \lim_{n \to \infty} \sum_{i=0}^{n-1} f(t_i) \cdot \Delta t$$

This is the "area under the curve" of $f(t)$ from $a$ to $b$. Each $f(t_i) \cdot \Delta t$ is a thin rectangular strip; their sum converges to the exact area.

### Definite vs Indefinite Integral

| Type | Notation | Returns | Example |
|---|---|---|---|
| **Indefinite** | $\int f(t)\, dt$ | A *family of functions* + constant $C$ | $\int 2t\, dt = t^2 + C$ |
| **Definite** | $\int_a^b f(t)\, dt$ | A *number* (the accumulated total) | $\int_0^3 2t\, dt = 9$ |

### The Fundamental Theorem of Calculus

$$\int_a^b f(t)\, dt = F(b) - F(a) \quad \text{where } F'(t) = f(t)$$

Translation: if you know the *antiderivative* $F$ of $f$, the definite integral is just $F$ evaluated at the endpoints.

### The Position-Velocity-Acceleration Chain (Integration Direction)

$$\mathbf{a}(t) \xrightarrow{\int} \mathbf{v}(t) = \mathbf{v}_0 + \int_0^t \mathbf{a}(\tau)\, d\tau \xrightarrow{\int} \mathbf{p}(t) = \mathbf{p}_0 + \int_0^t \mathbf{v}(\tau)\, d\tau$$

Starting from acceleration (forces / mass), you integrate once to get velocity, then again to get position.

### Common Antiderivatives Used in Games

| Function $f(t)$ | Antiderivative $F(t)$ | Game Use |
|---|---|---|
| $c$ (constant) | $ct + C$ | Constant velocity → linear position |
| $t$ | $\frac{t^2}{2} + C$ | Constant acceleration → quadratic position |
| $t^n$ | $\frac{t^{n+1}}{n+1} + C$ | Polynomial motion curves |
| $\sin(t)$ | $-\cos(t) + C$ | Oscillating forces → position |
| $\cos(t)$ | $\sin(t) + C$ | Oscillating position from force |
| $e^{-kt}$ | $-\frac{1}{k}e^{-kt} + C$ | Exponential decay (drag, poison) |

### Discrete Integration (What Game Loops Actually Do)

Every frame, your game loop performs:

$$\mathbf{v}_{n+1} = \mathbf{v}_n + \mathbf{a}_n \cdot \Delta t$$
$$\mathbf{p}_{n+1} = \mathbf{p}_n + \mathbf{v}_n \cdot \Delta t$$

This is **Euler integration** — the simplest discrete integral. It approximates the continuous integral with rectangular strips of width $\Delta t$.

### Discrete vs Continuous: The Approximation Gap

| Aspect | Continuous | Discrete (Game Loop) |
|---|---|---|
| Time | Real number $t$ | Frame ticks $n \cdot \Delta t$ |
| Accuracy | Exact | Approximate (error ~ $\Delta t$) |
| Computation | Closed-form if possible | Sum each frame |
| Stability | Always stable | Can diverge with large $\Delta t$ |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Filling a Bucket with a Variable Tap**
> Imagine a water tap whose flow rate changes over time — sometimes a trickle, sometimes a gush. The *derivative* tells you how fast water is flowing *right now*. The *integral* tells you the total amount of water in the bucket after some time. Each frame of your game is like sampling the tap's flow rate and adding that amount to the bucket: `bucket += flowRate * deltaTime`. This is numerical integration. A wider bucket (larger deltaTime) means each sample covers more time but is less precise — you might miss a sudden surge or drop.

> [!tip] **Mental Model: Area Under the Curve**
> Draw a velocity-vs-time graph. The area between the curve and the time axis equals the total distance traveled. A constant velocity of 5 m/s for 3 seconds gives a rectangle of area 15 m. An accelerating object produces a triangle or curved shape — the area still equals displacement. Your game loop computes this area one thin strip at a time, each strip being `velocity * deltaTime`.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Demonstrating integration: velocity → position, DOT accumulation
using UnityEngine;

public class IntegrationDemo : MonoBehaviour
{
    [Header("Physics Integration")]
    public Vector3 velocity = new Vector3(2f, 5f, 0f);
    public Vector3 gravity = new Vector3(0f, -9.81f, 0f);

    [Header("Damage Over Time")]
    public float initialDPS = 20f;
    public float decayRate = 0.5f;
    private float totalDamage = 0f;
    private float dotElapsed = 0f;
    private bool dotActive = false;

    void FixedUpdate()
    {
        float dt = Time.fixedDeltaTime;

        // --- Integration: acceleration → velocity → position ---
        // ∫ acceleration dt → velocity  (Euler step)
        velocity += gravity * dt;

        // ∫ velocity dt → position  (Euler step)
        transform.position += velocity * dt;

        // --- Accumulating damage over time ---
        if (dotActive)
        {
            dotElapsed += dt;

            // DPS at current time: initialDPS * e^(-decayRate * t)
            float currentDPS = initialDPS * Mathf.Exp(-decayRate * dotElapsed);

            // Numerical integration: add this frame's damage contribution
            totalDamage += currentDPS * dt;

            // Compare with analytical integral:
            // Total = (initialDPS / decayRate) * (1 - e^(-decayRate * t))
            float analyticalTotal = (initialDPS / decayRate)
                                  * (1f - Mathf.Exp(-decayRate * dotElapsed));

            Debug.Log($"DOT — Numerical: {totalDamage:F2}, " +
                      $"Analytical: {analyticalTotal:F2}, " +
                      $"Error: {Mathf.Abs(totalDamage - analyticalTotal):F4}");
        }
    }

    public void ApplyDOT()
    {
        dotActive = true;
        dotElapsed = 0f;
        totalDamage = 0f;
    }

    /// <summary>
    /// Analytical total damage after 'duration' seconds of the DOT effect.
    /// Uses the closed-form integral of initialDPS * e^(-decayRate * t).
    /// </summary>
    public float PredictTotalDamage(float duration)
    {
        return (initialDPS / decayRate) * (1f - Mathf.Exp(-decayRate * duration));
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the relationship between velocity and position via integration? :: Position is the integral of velocity: $\mathbf{p}(t) = \mathbf{p}_0 + \int_0^t \mathbf{v}(\tau)\, d\tau$.
- What does the Fundamental Theorem of Calculus say? :: If $F'(t) = f(t)$, then $\int_a^b f(t)\, dt = F(b) - F(a)$.
- What does `position += velocity * deltaTime` represent mathematically? :: A discrete Euler approximation of the integral $\mathbf{p}(t + \Delta t) = \mathbf{p}(t) + \int_t^{t+\Delta t} \mathbf{v}(\tau)\, d\tau$.
- What is the antiderivative of $e^{-kt}$? :: $-\frac{1}{k}e^{-kt} + C$.
- Why might a numerical integral diverge from the analytical one? :: Numerical integration accumulates small errors each step. With large $\Delta t$ or rapidly changing functions, these errors compound.
- How do you compute total accumulated value from a rate function? :: Integrate the rate function over the time interval: $\text{Total} = \int_0^T \text{rate}(t)\, dt$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `Update()` with variable `Time.deltaTime` for physics integration, causing different behavior at different frame rates.
> - **The Fix:** Use `FixedUpdate()` with `Time.fixedDeltaTime` for all physics-related integration. Or use a fixed timestep accumulator pattern.
> - **Why:** Integration error is proportional to $\Delta t$. Variable frame rates mean variable errors, causing objects to land at different positions at 30 FPS vs 144 FPS.

> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting the initial condition ($+C$ / $\mathbf{v}_0$ / $\mathbf{p}_0$) when integrating.
> - **The Fix:** Always include the starting value. $\mathbf{p}(t) = \mathbf{p}_0 + \int \mathbf{v}\, dt$, not just $\int \mathbf{v}\, dt$.
> - **Why:** The integral gives you the *change* in quantity. Without the initial value, you don't know the absolute result — like knowing you walked 5 km but not knowing where you started.

---

## Related Topics
- [[Math/09_Calculus_for_Games/derivatives_motion|Derivatives & Motion]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration]]
- [[Math/10_Physics_Math/kinematics|Kinematics]]
