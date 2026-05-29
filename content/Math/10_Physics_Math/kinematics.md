---
title: "Kinematics: Velocity, Acceleration, and Projectile Motion"
tags:
  - math
  - level/Lv3
  - category/physics_math
---

# Kinematics: Velocity, Acceleration, and Projectile Motion

> [!abstract] **The Concept in a Nutshell**
> Kinematics is the branch of physics that describes the motion of points, bodies, and systems of objects without considering the forces that cause the motion. In games, kinematics governs how objects move under constant acceleration (like gravity) or constant velocity. By mastering kinematic equations (often called **SUVAT equations**), we can predict where a projectile will land, calculate the exact velocity needed to make a character jump to a specific height, and compute trajectories.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Calculating Jump Velocity for a Platformer Character**
> You're building a platformer. You want the player to jump exactly $H = 3$ meters high to reach a ledge. The gravity in your game is $g = 9.81\text{ m/s}^2$ (pointing down).
>
> If you just guess the jump velocity (e.g., `rb.velocity = new Vector3(0, 15f, 0)`), players will overshoot or undershoot the platform. 
> To calculate the exact initial velocity $v_y$ required, we use the kinematic equation:
> $$v^2 = u^2 + 2as$$
>
> At the peak of the jump, the vertical velocity is $0$ ($v = 0$). The acceleration is gravity ($a = -g$). The displacement is jump height ($s = H$). Solving for initial velocity ($u$):
> $$0 = u^2 - 2gH \implies u = \sqrt{2gH}$$
>
> Plugging in the numbers: $u = \sqrt{2 \times 9.81 \times 3} \approx 7.67\text{ m/s}$. Setting the character's upward velocity to $7.67$ makes them peak exactly at $3$ meters.

---

## The Blueprint (Formula & Structure)

### The SUVAT Equations of Motion (Constant Acceleration)
For motion along a straight line with constant acceleration:

1. **Velocity-Time:**
   $$v = u + at$$
2. **Displacement-Time (Traditional):**
   $$s = ut + \frac{1}{2}at^2$$
3. **Velocity-Displacement:**
   $$v^2 = u^2 + 2as$$
4. **Displacement-Average Velocity:**
   $$s = \frac{u+v}{2}t$$

Where:
- $s$: Displacement (change in position).
- $u$: Initial velocity.
- $v$: Final velocity.
- $a$: Constant acceleration.
- $t$: Time elapsed.

### Projectile Motion (2D / 3D)
A projectile launched with initial speed $v_0$ at an angle $\theta$ experiences constant gravity acceleration $g$ along the vertical axis and zero acceleration horizontally:

- **Horizontal position at time $t$:**
  $$x(t) = v_0 \cos(\theta) t$$
- **Vertical position at time $t$:**
  $$y(t) = v_0 \sin(\theta) t - \frac{1}{2}gt^2$$
- **Total Time of Flight (to return to launch height $y=0$):**
  $$t_{\text{flight}} = \frac{2v_0 \sin(\theta)}{g}$$
- **Horizontal Range:**
  $$R = \frac{v_0^2 \sin(2\theta)}{g}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Accumulator Chain**
> Think of motion as an assembly line:
> - **Acceleration** is the speed at which you are adding to velocity.
> - **Velocity** is the speed at which you are adding to position.
> - **Position** is where you are.
>
> If you accelerate at $2\text{ m/s}^2$, every second your speedometer (velocity) increases by $2$. As velocity grows, your position advances faster and faster.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Jump height calculation and projectile trajectory prediction
using UnityEngine;

public class KinematicController : MonoBehaviour
{
    public float gravity = 9.81f;
    public Transform target;
    public LineRenderer trajectoryLine;
    public int resolution = 30;

    // Calculate jump velocity required to reach height
    public Vector3 CalculateJumpVelocity(float targetHeight)
    {
        // v^2 = u^2 + 2as => u = sqrt(2 * g * h)
        float upwardSpeed = Mathf.Sqrt(2f * gravity * targetHeight);
        return new Vector3(0f, upwardSpeed, 0f);
    }

    // Predict and draw the trajectory of a projectile launched with a velocity
    public void DrawTrajectory(Vector3 launchPosition, Vector3 launchVelocity)
    {
        trajectoryLine.positionCount = resolution;
        
        for (int i = 0; i < resolution; i++)
        {
            float t = (i / (float)resolution) * 3f; // Simulate 3 seconds
            
            // s = ut + 0.5 * a * t^2
            Vector3 displacement = launchVelocity * t + 0.5f * new Vector3(0f, -gravity, 0f) * (t * t);
            Vector3 position = launchPosition + displacement;
            
            trajectoryLine.SetPosition(i, position);
        }
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Rigidbody rb = GetComponent<Rigidbody>();
            if (rb != null)
            {
                // Set velocity to reach exactly 3 meters height
                rb.velocity = CalculateJumpVelocity(3f);
            }
        }
        
        // Predict trajectory from current position pointing forward
        DrawTrajectory(transform.position, transform.forward * 10f + Vector3.up * 5f);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the SUVAT parameter $s$ represent? :: **Displacement** (the change in position, not just distance).
- What formula calculates the initial velocity $u$ needed to reach a maximum vertical height $H$ under gravity $g$? :: $u = \sqrt{2gH}$.
- Why does horizontal velocity remain constant in basic projectile motion equations? :: Because gravity acts only vertically (pointing down), and air resistance (drag) is assumed to be zero.
- Write the kinematic equation that relates final velocity, initial velocity, acceleration, and displacement without using time. :: $v^2 = u^2 + 2as$.
- What is the total time of flight for a projectile launched vertically with initial velocity $u$ under gravity $g$ returning to its starting height? :: $t_{\text{flight}} = \frac{2u}{g}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using kinematic equations when gravity changes dynamically over time.
> - **The Fix:** Use numerical integration (e.g. Verlet or Runge-Kutta) for complex physics systems where forces scale dynamically.
> - **Why:** The SUVAT equations *only* work if acceleration is constant. If gravity scales with distance (like in planetary gravity systems) or includes drag force (which changes with speed), these formulas will generate incorrect trajectories.

---

## Related Topics
- [[Math/09_Calculus_for_Games/derivatives_motion|Derivatives & Motion]]
- [[Math/10_Physics_Math/newtonian_dynamics|Newtonian Dynamics]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
---
