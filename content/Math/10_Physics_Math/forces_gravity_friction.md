---
title: "Forces: Gravity, Friction, Drag, and Springs"
tags:
  - math
  - level/Lv3
  - category/physics_math
---

# Forces: Gravity, Friction, Drag, and Springs

> [!abstract] **The Concept in a Nutshell**
> Forces dictate how objects interact with their environment. While Newton's Second Law ($\vec{F} = m\vec{a}$) defines the relationship between force and motion, we need specific mathematical models to calculate the individual forces: **gravity** (vertical acceleration), **friction** (resisting sliding motion), **drag** (air resistance slowing fast objects), and **spring forces** (elastic recoil). These mathematical models give objects weight, traction, and elasticity.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Implementing a Grappling Hook with Spring Physics**
> You are building a game featuring a grappling hook. When the player hooks onto a ceiling anchor $A$, you want the player character $P$ to swing dynamically and bounce like they are hanging from an elastic rope.
>
> Using simple interpolation (like LERP) to pull the player to the anchor looks rigid and artificial. Instead, you can model the rope as a **spring-damper system** using **Hooke's Law**:
> 1. Calculate the spring stretch: displacement $\vec{x}$ from anchor to player relative to the rope's rest length.
> 2. Calculate the restoring force: $\vec{F}_{\text{spring}} = -k\vec{x}$, where $k$ is the spring stiffness.
> 3. Calculate a damping force: $\vec{F}_{\text{damping}} = -c\vec{v}$, where $c$ is a friction factor and $\vec{v}$ is player velocity. Without damping, the player would bounce forever.
>
> Adding this spring force vector to the player's physics calculations results in a smooth, elastic swing that feels great to play.

---

## The Blueprint (Formula & Structure)

### 1. Gravity Models
- **Constant Gravity (Local Scale):**
  $$\vec{F}_g = m\vec{g}$$
  Where $\vec{g} \approx (0, -9.81, 0)\text{ m/s}^2$. Used for standard ground-based games.
- **Newton's Law of Universal Gravitation (Space Scale):**
  $$\vec{F}_g = G \frac{M m}{r^2} \vec{u}_r$$
  Used for orbital mechanics, space gravity, or attracting planets. Force decreases with the square of the distance ($r^2$).

### 2. Friction
Friction acts parallel to the contact surface, opposing the direction of sliding.
- **Static Friction (Resting):** Resists initial movement. Maximum static friction is:
  $$F_s \le \mu_s F_N$$
- **Kinetic Friction (Sliding):** Slows down moving objects:
  $$F_k = \mu_k F_N$$
Where $F_N$ is the normal force pushing the surfaces together, and $\mu$ is the **friction coefficient** (high for concrete, low for ice).

### 3. Drag (Fluid Resistance)
Air resistance slows down moving objects, scaling quadratically with speed:
$$\vec{F}_d = -\frac{1}{2} \rho v^2 C_d A \vec{u}_v$$
- In games, we simplify this to: $\vec{F}_d = -c_d |\vec{v}|\vec{v}$ or linear drag: $\vec{F}_d = -k \vec{v}$.
- **Terminal Velocity:** The maximum speed an object can fall at, reached when drag force equals gravity force.

### 4. Hooke's Law (Springs)
The force exerted by an ideal spring is directly proportional to its displacement:
$$\vec{F}_s = -k \vec{x}$$
Where $k$ is stiffness and $\vec{x}$ is the extension/compression vector from rest length.
- **Spring-Damper (Realistic Spring):** Adds a damping force to prevent infinite oscillation:
  $$\vec{F}_{\text{spring}} = -k \vec{x} - c \vec{v}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Friction & Spring Slides**
> - **Friction** is like dragging a heavy rug across concrete: it resists you, but if you don't push, it doesn't move.
> - **Drag** is like wading through water: if you stand still, you feel nothing; if you run, the water pushes back hard.
> - **Springs** are like rubber bands: the further you stretch them, the harder they snap back.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simulating Spring forces and Friction
using UnityEngine;

public class ForceSimulator : MonoBehaviour
{
    [Header("Spring Settings")]
    public Transform anchor;
    public float restLength = 3f;
    public float springStiffness = 50f;
    public float damping = 2f;

    [Header("Friction & Drag")]
    public float kineticFrictionCoeff = 0.3f;
    public float dragCoeff = 0.1f;

    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void FixedUpdate()
    {
        // 1. Spring Force (Hooke's Law + Damping)
        Vector3 offset = transform.position - anchor.position;
        float currentLength = offset.magnitude;
        Vector3 direction = offset.normalized;

        float extension = currentLength - restLength;
        // Spring restoring force
        Vector3 springForce = -springStiffness * extension * direction;
        // Damping force along spring direction
        float velocityAlongSpring = Vector3.Dot(rb.velocity, direction);
        Vector3 dampingForce = -damping * velocityAlongSpring * direction;

        rb.AddForce(springForce + dampingForce);

        // 2. Drag (Air Resistance: proportional to speed squared)
        Vector3 dragForce = -dragCoeff * rb.velocity.magnitude * rb.velocity;
        rb.AddForce(dragForce);

        // 3. Ground Friction (If touching a flat floor)
        if (transform.position.y <= 0.1f) // Simple ground check
        {
            float normalForce = rb.mass * 9.81f; // F_N = m * g
            Vector3 slidingDirection = new Vector3(rb.velocity.x, 0f, rb.velocity.z).normalized;
            Vector3 frictionForce = -kineticFrictionCoeff * normalForce * slidingDirection;
            
            rb.AddForce(frictionForce);
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Write Hooke's Law equation. :: $F = -kx$, where $k$ is the spring stiffness and $x$ is displacement.
- What is the difference between static and kinetic friction? :: Static friction resists the start of movement on a stationary object, whereas kinetic friction resists the motion of a sliding object.
- Why is damping necessary in a spring physics simulation? :: Damping removes energy from the spring system, mimicking natural friction and preventing the spring from bouncing forever.
- What factors determine air resistance (drag) force in physics? :: Air density ($\rho$), object velocity squared ($v^2$), drag coefficient ($C_d$), and cross-sectional area ($A$).
- What is terminal velocity? :: The constant speed reached by a falling object when the upward drag force equals the downward force of gravity, resulting in zero acceleration.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Implementing a spring without damping (e.g. only using $F = -kx$).
> - **The Fix:** Always calculate the relative velocity along the spring vector and apply a opposing damping force.
> - **Why:** Without damping, the spring will accumulate floating-point rounding errors and bounce with increasing amplitude, eventually exploding the object's position into infinity (`NaN`).

---

## Related Topics
- [[Math/10_Physics_Math/newtonian_dynamics|Newtonian Dynamics]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
- [[Math/10_Physics_Math/collision_response|Collision Response]]
