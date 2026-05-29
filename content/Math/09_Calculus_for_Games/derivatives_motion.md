---
title: "Derivatives & Motion — The Math Behind Velocity and Acceleration"
tags:
  - math
  - level/Lv3
  - category/calculus
---

# Derivatives & Motion: The Math Behind Velocity and Acceleration

> [!abstract] **The Concept in a Nutshell**
> The derivative measures the *instantaneous rate of change* of a quantity. In games, velocity is the derivative of position with respect to time, and acceleration is the derivative of velocity. Understanding derivatives lets you predict where objects will be, how fast they're moving, and how their motion is changing — the mathematical foundation for all physics simulation and AI prediction.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Predictive Aiming for a Turret AI**
> Your tower-defense turret needs to lead its shots against a moving alien. The alien's position is $\mathbf{p}(t)$, but by the time the bullet arrives, the alien will have moved. The turret samples the alien's position over two frames and computes the velocity $\mathbf{v} = \frac{\Delta \mathbf{p}}{\Delta t}$ — that's a discrete derivative. It then predicts the alien's future position: $\mathbf{p}_{\text{future}} = \mathbf{p} + \mathbf{v} \cdot t_{\text{bullet}}$. If the alien is accelerating (turning), the turret uses the second derivative (acceleration) for an even better quadratic prediction: $\mathbf{p}_{\text{future}} = \mathbf{p} + \mathbf{v} t + \frac{1}{2}\mathbf{a} t^2$.

---

## The Blueprint (Formula & Structure)

### The Derivative — Formal Definition

$$f'(x) = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x}$$

In game terms, replace $x$ with time $t$ and $f$ with position $p$:

$$v(t) = \lim_{\Delta t \to 0} \frac{p(t + \Delta t) - p(t)}{\Delta t} = \frac{dp}{dt}$$

### The Chain: Position → Velocity → Acceleration

$$\text{position: } \mathbf{p}(t)$$

$$\text{velocity: } \mathbf{v}(t) = \frac{d\mathbf{p}}{dt} = \dot{\mathbf{p}}$$

$$\text{acceleration: } \mathbf{a}(t) = \frac{d\mathbf{v}}{dt} = \frac{d^2\mathbf{p}}{dt^2} = \ddot{\mathbf{p}}$$

$$\text{jerk (rate of acceleration change): } \mathbf{j}(t) = \frac{d\mathbf{a}}{dt} = \frac{d^3\mathbf{p}}{dt^3}$$

### Common Derivative Rules

| Rule | Formula | Game Example |
|---|---|---|
| **Constant** | $\frac{d}{dt}[c] = 0$ | A stationary wall has zero velocity |
| **Power** | $\frac{d}{dt}[t^n] = n t^{n-1}$ | Quadratic motion: $p = \frac{1}{2}at^2 \Rightarrow v = at$ |
| **Sum** | $(f+g)' = f' + g'$ | Velocity components add independently |
| **Product** | $(fg)' = f'g + fg'$ | Moving + rotating hitbox |
| **Chain** | $\frac{d}{dt}[f(g(t))] = f'(g(t)) \cdot g'(t)$ | Easing function applied to time |
| **Trig** | $\frac{d}{dt}[\sin t] = \cos t$ | Oscillating platform velocity |
| **Exponential** | $\frac{d}{dt}[e^t] = e^t$ | Exponential decay (e.g., drag) |

### Discrete Derivative (What Games Actually Use)

In a game loop, time is not continuous — you have frames. The discrete approximation:

$$v_n \approx \frac{p_n - p_{n-1}}{\Delta t}$$

This is a **backward finite difference**. Other options:

| Method | Formula | Accuracy |
|---|---|---|
| **Forward difference** | $\frac{p_{n+1} - p_n}{\Delta t}$ | $O(\Delta t)$ |
| **Backward difference** | $\frac{p_n - p_{n-1}}{\Delta t}$ | $O(\Delta t)$ |
| **Central difference** | $\frac{p_{n+1} - p_{n-1}}{2\Delta t}$ | $O(\Delta t^2)$ — more accurate! |

### Partial Derivatives (Preview)

When a quantity depends on multiple variables — say, terrain height $h(x, z)$ — the **partial derivative** measures the rate of change along one axis while holding others constant:

$$\frac{\partial h}{\partial x} = \text{slope in the } x \text{ direction}$$

$$\frac{\partial h}{\partial z} = \text{slope in the } z \text{ direction}$$

The **gradient** $\nabla h = \left(\frac{\partial h}{\partial x}, \frac{\partial h}{\partial z}\right)$ points in the direction of steepest ascent — critical for terrain normal calculation and water flow simulation.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Speedometer**
> Your car has a position (odometer reading) and a speedometer. The speedometer is the derivative of the odometer — it tells you how quickly the odometer number is *changing* right now. The rate at which the *speedometer itself* changes is acceleration — you feel it as being pushed back into your seat. In games, `transform.position` is the odometer. The difference between this frame's position and last frame's position, divided by `deltaTime`, is the speedometer. And the difference between this frame's velocity and last frame's velocity, divided by `deltaTime`, is the acceleration you'd feel.

> [!tip] **Mental Model: The Tangent Line**
> The derivative at a point is the *slope of the tangent line* — the line that just barely touches the curve at that point. If you zoomed in infinitely close to a smooth curve at any point, it would look like a straight line. The slope of that line is the derivative. For a position-time graph, a steep tangent = high velocity; a flat tangent = the object is momentarily stopped.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Computing velocity and acceleration from position, predictive aiming
using UnityEngine;

public class MotionDerivatives : MonoBehaviour
{
    [Header("Target to track")]
    public Transform target;

    // Stored previous frame values
    private Vector3 prevPosition;
    private Vector3 prevVelocity;

    // Computed derivatives
    public Vector3 Velocity    { get; private set; }
    public Vector3 Acceleration { get; private set; }

    void Start()
    {
        prevPosition = target.position;
        prevVelocity = Vector3.zero;
    }

    void FixedUpdate()
    {
        float dt = Time.fixedDeltaTime;

        // First derivative: velocity = dp/dt
        Velocity = (target.position - prevPosition) / dt;

        // Second derivative: acceleration = dv/dt
        Acceleration = (Velocity - prevVelocity) / dt;

        // Store for next frame
        prevPosition = target.position;
        prevVelocity = Velocity;
    }

    /// <summary>
    /// Predict where the target will be after 'time' seconds.
    /// Uses quadratic prediction: p + v*t + 0.5*a*t^2
    /// </summary>
    public Vector3 PredictPosition(float time)
    {
        return target.position
             + Velocity * time
             + 0.5f * Acceleration * time * time;
    }
}

// --- Usage: Turret predictive aiming ---
public class PredictiveTurret : MonoBehaviour
{
    public MotionDerivatives targetMotion;
    public float bulletSpeed = 50f;
    public Transform firePoint;

    void Update()
    {
        // Estimate bullet flight time
        float distance = Vector3.Distance(firePoint.position, targetMotion.target.position);
        float flightTime = distance / bulletSpeed;

        // Predict where target will be when bullet arrives
        Vector3 aimPoint = targetMotion.PredictPosition(flightTime);

        // Aim at predicted position
        transform.forward = (aimPoint - firePoint.position).normalized;

        Debug.DrawLine(firePoint.position, aimPoint, Color.red);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is velocity in terms of a derivative? :: Velocity is the first derivative of position with respect to time: $\mathbf{v} = \frac{d\mathbf{p}}{dt}$.
- What is the chain rule and when does it appear in games? :: $\frac{d}{dt}[f(g(t))] = f'(g(t)) \cdot g'(t)$. It appears when easing functions warp time: the actual speed is the easing derivative times the base speed.
- What is the central difference formula and why is it better? :: $\frac{p_{n+1} - p_{n-1}}{2\Delta t}$ — it has $O(\Delta t^2)$ accuracy vs $O(\Delta t)$ for forward/backward differences.
- What does the gradient $\nabla h$ of a height field represent? :: A vector pointing in the direction of steepest ascent, whose magnitude is the slope in that direction.
- How do you predict a target's future position using derivatives? :: Quadratic prediction: $\mathbf{p}_{\text{future}} = \mathbf{p} + \mathbf{v} t + \frac{1}{2}\mathbf{a} t^2$.
- What is jerk, and why does it matter in games? :: Jerk is the derivative of acceleration ($d\mathbf{a}/dt$). It matters for camera smoothness — jerky acceleration produces uncomfortable, jarring motion.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Computing velocity in `Update()` using `Time.deltaTime`, which varies between frames, causing noisy derivative estimates.
> - **The Fix:** Use `FixedUpdate()` with `Time.fixedDeltaTime` for physics-related derivatives, or apply smoothing (exponential moving average) to the computed velocity.
> - **Why:** Variable frame rates cause $\Delta t$ to fluctuate, and since velocity is divided by $\Delta t$, tiny time steps amplify noise. Fixed timestep gives stable, consistent derivatives.

> [!danger] **Watch Out!**
> - **The Mistake:** Confusing the derivative of *speed* (scalar) with the derivative of *velocity* (vector).
> - **The Fix:** Speed is $|\mathbf{v}|$ — its derivative tells you if the object is speeding up or slowing down. Acceleration $\mathbf{a} = d\mathbf{v}/dt$ tells you the full change in velocity, *including direction changes*.
> - **Why:** An object moving in a circle at constant speed still has nonzero acceleration (centripetal) because its velocity *direction* is changing.

---

## Related Topics
- [[Math/09_Calculus_for_Games/integrals_accumulation|Integrals & Accumulation]]
- [[Math/10_Physics_Math/kinematics|Kinematics]]
- [[Math/10_Physics_Math/forces_gravity_friction|Forces, Gravity & Friction]]
