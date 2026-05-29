---
title: "Numerical Integration — Keeping Physics from Exploding"
tags:
  - math
  - level/Lv3
  - category/calculus
---

# Numerical Integration: Keeping Physics from Exploding

> [!abstract] **The Concept in a Nutshell**
> Games can't evaluate continuous integrals — they step forward in discrete time increments. The choice of *how* you step matters enormously: naive Euler integration causes energy gain and instability; semi-implicit Euler fixes much of this cheaply; Verlet integration excels for constraints; and RK4 provides high accuracy for complex systems. Understanding these methods is the difference between physics that feels solid and physics that "explodes."

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Why Your Cloth Sim Explodes at Low FPS**
> You've built a cloth simulation using springs between vertices. At 60 FPS everything billows beautifully. But when the player's GPU stutters to 15 FPS, $\Delta t$ quadruples — and suddenly the cloth stretches wildly, then snaps inward, oscillates with growing amplitude, and within a few frames the vertices are at coordinates in the millions. The simulation has "exploded." The culprit: **explicit Euler integration** with a timestep too large for the spring stiffness. Switching to **semi-implicit Euler** and using a **fixed timestep** accumulator solves the problem completely.

---

## The Blueprint (Formula & Structure)

### The Integration Problem

Given an ODE (ordinary differential equation):

$$\frac{d\mathbf{y}}{dt} = f(t, \mathbf{y})$$

We want to advance the state $\mathbf{y}$ forward by a timestep $h = \Delta t$.

For game physics, the typical state is $\mathbf{y} = (\mathbf{p}, \mathbf{v})$ and $f$ computes $(\mathbf{v}, \mathbf{a})$.

---

### Method 1: Explicit (Forward) Euler

$$\mathbf{v}_{n+1} = \mathbf{v}_n + \mathbf{a}_n \cdot h$$
$$\mathbf{p}_{n+1} = \mathbf{p}_n + \mathbf{v}_n \cdot h$$

| Property | Value |
|---|---|
| **Order** | 1st-order accurate ($O(h)$ local error) |
| **Stability** | **Poor** — adds energy over time |
| **Cost** | 1 force evaluation per step |
| **When to use** | Quick prototypes, non-critical systems |

The fundamental problem: velocity is evaluated at time $n$ but used to advance position to time $n+1$. This "lag" causes systematic energy gain in oscillatory systems (springs, orbits).

---

### Method 2: Semi-Implicit (Symplectic) Euler

$$\mathbf{v}_{n+1} = \mathbf{v}_n + \mathbf{a}_n \cdot h$$
$$\mathbf{p}_{n+1} = \mathbf{p}_n + \mathbf{v}_{n+1} \cdot h$$

The only change: position uses the **updated** velocity $\mathbf{v}_{n+1}$.

| Property | Value |
|---|---|
| **Order** | 1st-order accurate |
| **Stability** | **Much better** — symplectic (energy-conserving on average) |
| **Cost** | 1 force evaluation per step |
| **When to use** | **Most game physics** — best cost/stability ratio |

This is what most game engines (including Unity and Box2D) use for rigidbody simulation.

---

### Method 3: Velocity Verlet (Störmer-Verlet)

$$\mathbf{p}_{n+1} = \mathbf{p}_n + \mathbf{v}_n \cdot h + \frac{1}{2}\mathbf{a}_n \cdot h^2$$
$$\mathbf{a}_{n+1} = f(t_{n+1}, \mathbf{p}_{n+1})$$
$$\mathbf{v}_{n+1} = \mathbf{v}_n + \frac{\mathbf{a}_n + \mathbf{a}_{n+1}}{2} \cdot h$$

| Property | Value |
|---|---|
| **Order** | 2nd-order accurate ($O(h^2)$ local error) |
| **Stability** | Excellent — symplectic, time-reversible |
| **Cost** | 1–2 force evaluations per step |
| **When to use** | Particle systems, cloth, soft body, molecular dynamics |

The Verlet family is especially good for **constraint-based** physics (e.g., keeping particles at fixed distances for cloth or ropes), because position is the primary variable.

---

### Method 4: Runge-Kutta 4th Order (RK4)

$$\mathbf{k}_1 = f(t_n, \mathbf{y}_n)$$
$$\mathbf{k}_2 = f\!\left(t_n + \frac{h}{2}, \mathbf{y}_n + \frac{h}{2}\mathbf{k}_1\right)$$
$$\mathbf{k}_3 = f\!\left(t_n + \frac{h}{2}, \mathbf{y}_n + \frac{h}{2}\mathbf{k}_2\right)$$
$$\mathbf{k}_4 = f(t_n + h, \mathbf{y}_n + h\,\mathbf{k}_3)$$
$$\mathbf{y}_{n+1} = \mathbf{y}_n + \frac{h}{6}(\mathbf{k}_1 + 2\mathbf{k}_2 + 2\mathbf{k}_3 + \mathbf{k}_4)$$

| Property | Value |
|---|---|
| **Order** | 4th-order accurate ($O(h^4)$ local error) |
| **Stability** | Good but **not** symplectic — energy drifts over long times |
| **Cost** | 4 force evaluations per step |
| **When to use** | Orbital mechanics, precision trajectories, scientific simulation |

### Comparison Summary

| Method | Accuracy | Stability | Cost | Best For |
|---|---|---|---|---|
| Explicit Euler | $O(h)$ | Bad | 1 eval | Prototypes only |
| Semi-Implicit Euler | $O(h)$ | Good | 1 eval | General game physics |
| Velocity Verlet | $O(h^2)$ | Excellent | 1–2 evals | Particles, cloth, constraints |
| RK4 | $O(h^4)$ | Good | 4 evals | Precision, orbits |

### Fixed Timestep Pattern

To ensure consistent physics regardless of frame rate:

```
accumulator += deltaTime
while (accumulator >= fixedDT):
    integrate(fixedDT)
    accumulator -= fixedDT
alpha = accumulator / fixedDT
renderPosition = lerp(previousState, currentState, alpha)
```

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Walking Down a Foggy Hill**
> Imagine walking down a curved hill in dense fog — you can only see your feet. **Explicit Euler** is like taking one step in the direction the ground slopes right where you're standing — but by the time you land, the slope might have changed. If the hill curves upward ahead, you overshoot and end up *higher* than where you started (energy gain!). **Semi-implicit Euler** peeks at the slope at your landing spot and adjusts. **RK4** takes four tiny "look-ahead" steps in the fog, averages their slopes, and then takes one accurate step — much more reliable but four times the work.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Comparing integration methods on a spring simulation
using UnityEngine;

public class IntegrationComparison : MonoBehaviour
{
    public enum Method { ExplicitEuler, SemiImplicitEuler, VelocityVerlet, RK4 }
    public Method method = Method.SemiImplicitEuler;

    public float stiffness = 50f;
    public float damping = 0.5f;
    public float mass = 1f;

    private float position;
    private float velocity;
    private float prevAccel;

    void Start()
    {
        position = 2f; // Start displaced from equilibrium
        velocity = 0f;
        prevAccel = Accel(position, velocity);
    }

    void FixedUpdate()
    {
        float h = Time.fixedDeltaTime;

        switch (method)
        {
            case Method.ExplicitEuler:
            {
                float a = Accel(position, velocity);
                position += velocity * h;
                velocity += a * h;
                break;
            }
            case Method.SemiImplicitEuler:
            {
                float a = Accel(position, velocity);
                velocity += a * h;
                position += velocity * h;  // Uses updated velocity
                break;
            }
            case Method.VelocityVerlet:
            {
                position += velocity * h + 0.5f * prevAccel * h * h;
                float newAccel = Accel(position, velocity);
                velocity += 0.5f * (prevAccel + newAccel) * h;
                prevAccel = newAccel;
                break;
            }
            case Method.RK4:
            {
                // State: (position, velocity)
                float p = position, v = velocity;

                float k1v = Accel(p, v);
                float k1p = v;

                float k2v = Accel(p + 0.5f * h * k1p, v + 0.5f * h * k1v);
                float k2p = v + 0.5f * h * k1v;

                float k3v = Accel(p + 0.5f * h * k2p, v + 0.5f * h * k2v);
                float k3p = v + 0.5f * h * k2v;

                float k4v = Accel(p + h * k3p, v + h * k3v);
                float k4p = v + h * k3v;

                position += (h / 6f) * (k1p + 2f * k2p + 2f * k3p + k4p);
                velocity += (h / 6f) * (k1v + 2f * k2v + 2f * k3v + k4v);
                break;
            }
        }

        // Visualize
        transform.position = new Vector3(position, 0f, 0f);
    }

    /// <summary>Spring + damping acceleration: F/m = (-kx - cv) / m</summary>
    float Accel(float x, float v)
    {
        return (-stiffness * x - damping * v) / mass;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Why does explicit Euler add energy to oscillatory systems? :: It evaluates velocity at time $n$ but advances position to time $n+1$, consistently overshooting on both sides of oscillation, causing the amplitude to grow.
- What one-line change turns explicit Euler into semi-implicit Euler? :: Use the *updated* velocity (after acceleration step) to advance position instead of the old velocity.
- What makes Verlet integration good for constraints? :: It treats position as the primary variable, making it easy to enforce distance constraints by directly adjusting positions.
- How many force evaluations does RK4 need per step? :: 4 evaluations (at the start, two at the midpoint, one at the end).
- What is a symplectic integrator? :: An integrator that preserves the phase-space volume of a Hamiltonian system, meaning it conserves energy *on average* over long simulations (no systematic drift).
- Why is a fixed timestep important for physics stability? :: Variable $\Delta t$ makes integration error unpredictable; large spikes can exceed the stability limit of the integrator, causing the simulation to diverge.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using RK4 for everything because "higher order = better."
> - **The Fix:** Semi-implicit Euler is sufficient (and cheaper) for the vast majority of game physics. Reserve RK4 for when accuracy over long time periods is critical (orbital mechanics, precise trajectory prediction).
> - **Why:** RK4 is 4× more expensive per step and is not symplectic — it still drifts energy over very long simulations. Semi-implicit Euler is symplectic and perfectly adequate for real-time games.

> [!danger] **Watch Out!**
> - **The Mistake:** Not using a fixed timestep, causing physics to behave differently at different frame rates.
> - **The Fix:** Implement a fixed-timestep accumulator loop. Decouple physics updates from rendering.
> - **Why:** All integration methods have a stability limit proportional to $\Delta t$. A frame rate drop that triples $\Delta t$ can push the system past that limit, causing explosions.

---

## Related Topics
- [[Math/09_Calculus_for_Games/integrals_accumulation|Integrals & Accumulation]]
- [[Math/10_Physics_Math/kinematics|Kinematics]]
- [[Math/10_Physics_Math/newtonian_dynamics|Newtonian Dynamics]]
