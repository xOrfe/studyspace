---
title: "Newtonian Dynamics: Forces, Acceleration, and Net Vectors"
tags:
  - math
  - level/Lv3
  - category/physics_math
---

# Newtonian Dynamics: Forces, Acceleration, and Net Vectors

> [!abstract] **The Concept in a Nutshell**
> While kinematics describes *how* objects move, dynamics describes *why* they move: **forces**. Newtonian dynamics is the foundation of almost all physical simulations in video games. It explains how forces (like wind, gravity, thrust, and friction) act on an object with mass to change its velocity over time. By applying Newton's three laws of motion, we can build custom physics systems, rigid-body engines, and responsive character controllers.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Building a Custom Spaceship Controller**
> You are programming a space combat game. The spaceship has a mass of $M = 1000$ kg. The player presses the throttle, firing thrusters that apply a forward force of $F = 5000$ Newtons. At the same time, the ship is caught in a tractor beam pulling it backward with $2000$ Newtons, and a solar wind pushing it sideways with $1000$ Newtons.
>
> To determine how the ship accelerates, you can't just modify the speed variable directly. You must:
> 1. Calculate the **net force** vector by summing all forces:
>    $$\vec{F}_{\text{net}} = \vec{F}_{\text{thruster}} + \vec{F}_{\text{beam}} + \vec{F}_{\text{wind}}$$
> 2. Apply Newton's Second Law ($\vec{F} = m\vec{a}$) to find the acceleration vector:
>    $$\vec{a} = \frac{\vec{F}_{\text{net}}}{M}$$
> 3. Add this acceleration to the velocity vector each frame. This creates realistic inertia, sliding, and drift that players expect in space games.

---

## The Blueprint (Formula & Structure)

### Newton's Three Laws of Motion

#### 1. The Law of Inertia (First Law)
An object at rest stays at rest, and an object in motion stays in motion with the same velocity and direction, unless acted upon by a net external force.
- **Game Dev Context:** Objects in a physics engine shouldn't stop sliding unless a force like friction or drag acts on them.

#### 2. The Law of Acceleration (Second Law)
The acceleration ($\vec{a}$) of an object is directly proportional to the net force ($\vec{F}$) acting on it, and inversely proportional to its mass ($m$):
$$\vec{F} = m\vec{a} \implies \vec{a} = \frac{\vec{F}}{m}$$

#### 3. Action-Reaction (Third Law)
For every action, there is an equal and opposite reaction. If Object A exerts a force on Object B, Object B exerts an equal force in the opposite direction on Object A:
$$\vec{F}_{\text{A on B}} = -\vec{F}_{\text{B on A}}$$
- **Game Dev Context:** When a player jumps off a physics-controlled crate, the crate should get pushed downward with the exact same impulse force used to launch the player upward.

### Superposition of Forces (Net Force)
Multiple forces acting on a single point can be summed together into a single equivalent force vector:
$$\vec{F}_{\text{net}} = \sum \vec{F}_i = \vec{F}_1 + \vec{F}_2 + \vec{F}_3 + \dots$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Speed Tug-of-War**
> Think of forces as players in a tug-of-war game, pulling a heavy block in different directions.
> - The mass of the block represents its resistance to moving. A heavier block moves slower for the same pull.
> - The direction the block accelerates is determined by adding up all the pulls. If X pulls harder than Y, the block accelerates in X's direction.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simulating Newtonian Dynamics on a custom object
using UnityEngine;

public class CustomPhysicsBody : MonoBehaviour
{
    public float mass = 2.0f; // Mass in kg
    public Vector3 velocity;
    
    private Vector3 netForce;

    // Call this to apply a force to the object
    public void AddForce(Vector3 force)
    {
        // Forces are additive (Superposition)
        netForce += force;
    }

    void FixedUpdate()
    {
        // 1. Calculate acceleration: a = F / m
        Vector3 acceleration = netForce / mass;

        // 2. Update velocity: v = v + a * dt
        velocity += acceleration * Time.fixedDeltaTime;

        // 3. Update position: p = p + v * dt
        transform.position += velocity * Time.fixedDeltaTime;

        // 4. Reset net force for the next frame
        netForce = Vector3.zero;

        // Apply constant gravity force (F = m * g)
        AddForce(new Vector3(0f, -9.81f * mass, 0f));
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- State Newton's Second Law mathematically. :: $\vec{F} = m\vec{a}$ (Force equals mass times acceleration).
- What happens to the acceleration of an object if its mass doubles but the force applied remains the same? :: The acceleration is cut in half ($\vec{a} = \frac{\vec{F}}{2m}$).
- Give a game development example of Newton's Third Law. :: A rocket thruster pushing exhaust gases downward (action), which pushes the rocket upward (reaction).
- What is net force? :: The vector sum of all individual forces acting on an object: $\vec{F}_{\text{net}} = \sum \vec{F}_i$.
- Why does a custom physics engine calculate acceleration from force instead of updating velocity directly? :: Because updating velocity directly ignores mass and inertia, making objects feel weightless and movement look jerky.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to clear the accumulated net force vector at the end of each physics frame.
> - **The Fix:** Set the net force vector to zero (`netForce = Vector3.zero`) immediately after updating velocity.
> - **Why:** If you don't reset the forces, the object will continuously re-apply forces from previous frames, resulting in infinite acceleration and the object flying off the screen.

---

## Related Topics
- [[Math/10_Physics_Math/kinematics|Kinematics]]
- [[Math/10_Physics_Math/forces_gravity_friction|Forces: Gravity, Friction & Springs]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
