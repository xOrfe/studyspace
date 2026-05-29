---
title: "Fluid Dynamics Basics: Navier-Stokes, SPH, and Grid-Based Fluids"
tags:
  - math
  - level/Lv4
  - category/advanced_topics
---

# Fluid Dynamics Basics: Navier-Stokes, SPH, and Grid-Based Fluids

> [!abstract] **The Concept in a Nutshell**
> Fluid simulation models the motion of liquids (water, lava) and gases (smoke, fire). It is governed by the **Navier-Stokes equations**, which describe how velocity, pressure, viscosity, and external forces interact within a fluid. In games, we approximate these complex equations using two primary approaches: **Eulerian (Grid-Based)** methods, which track fluid changes across a static grid, and **Lagrangian (Particle-Based)** methods, like **Smoothed Particle Hydrodynamics (SPH)**, which simulate fluids using moving particles.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Simulating Potion Liquids Swirling in a Bottle**
> You are building an alchemy game where players mix colored liquids in bottles. When the player shakes the bottle, the liquids should swirl, mix, and form realistic ripples.
>
> Using simple textures or vertex waving animations fails to react to shaking. 
> To implement realistic mixing, the game uses **Smoothed Particle Hydrodynamics (SPH)**:
> 1. The liquid is represented by $1000$ physics particles.
> 2. The engine calculates the **density** at each particle's position by checking nearby particles.
> 3. Based on density differences, the engine calculates **pressure forces** that push particles apart (preventing collapse) and **viscosity forces** that make the fluid flow smoothly.
>
> This creates a dynamic, interactive fluid reaction that looks and behaves like real liquid.

---

## The Blueprint (Formula & Structure)

### 1. The Navier-Stokes Equations
The motion of an incompressible fluid is described by the momentum equation:
$$\frac{\partial \vec{v}}{\partial t} + (\vec{v} \cdot \nabla)\vec{v} = -\frac{1}{\rho}\nabla p + \nu \nabla^2 \vec{v} + \vec{g}$$

Where:
- $\frac{\partial \vec{v}}{\partial t}$: Acceleration of the fluid.
- $(\vec{v} \cdot \nabla)\vec{v}$: **Advection** (fluid carrying velocity along its own flow).
- $-\frac{1}{\rho}\nabla p$: **Pressure** forces (particles pushing away from high-density areas).
- $\nu \nabla^2 \vec{v}$: **Viscosity** (internal friction/thickness, high for honey, low for water).
- $\vec{g}$: External forces (gravity, wind).

### 2. Grid-Based (Eulerian) vs. Particle-Based (Lagrangian)

| Method | Approach | Game Dev Use Case | Pros | Cons |
|--------|----------|-------------------|------|------|
| **Eulerian (Grid)** | Space is split into cells; we track fluid passing through cells. | Smoke, fire, fog, large-scale oceans | Extremely stable, clean boundaries | Hard to capture fine splashes, high memory usage |
| **Lagrangian (SPH)** | Fluid is represented by moving particles. | Spills, splashes, pouring water, mixing potion | Great for splashes, easy physics integration | Computationally expensive ($O(N^2)$ neighbor search) |

### 3. Smoothed Particle Hydrodynamics (SPH)
SPH approximates fluid properties by averaging values over nearby particles using a **Kernel Function $W$**:
- **Density ($\rho_i$) at particle $i$:**
  $$\rho_i = \sum_{j} m_j W(\vec{r}_i - \vec{r}_j, h)$$
- **Pressure ($p_i$) calculation (using ideal gas state equation):**
  $$p_i = k(\rho_i - \rho_0)$$
- **Pressure Force on particle $i$:**
  $$\vec{F}_{i,\text{pressure}} = -\sum_{j} m_j \left(\frac{p_i}{\rho_i^2} + \frac{p_j}{\rho_j^2}\right) \nabla W(\vec{r}_i - \vec{r}_j, h)$$
Where $h$ is the smoothing radius (cutoff range).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Grid Coordinates vs. Floating Dust**
> - **Eulerian (Grid)** is like standing on a bridge and counting cars passing through intersection points. The grid cells stay still, but we record the speed of the traffic flowing through them.
> - **Lagrangian (SPH)** is like putting GPS trackers on individual cars and following their paths. The particles carry their own mass and velocity vectors through space.

---

## Code Example (Applied in Engine)

```csharp
// Outline code for SPH density calculation in 2D
using UnityEngine;

public class SPHFluidSimulator : MonoBehaviour
{
    struct FluidParticle
    {
        public Vector2 position;
        public Vector2 velocity;
        public Vector2 force;
        public float density;
        public float pressure;
    }

    public int numParticles = 200;
    public float smoothingRadius = 1f; // Cutoff distance h
    public float targetDensity = 1.0f; // Rest density rho_0
    public float gasConstant = 2000f; // Stiffness coefficient k

    private FluidParticle[] particles;

    // SPH smoothing kernel (Spiky Kernel for density/pressure)
    float Kernel(float dist, float h)
    {
        if (dist >= h) return 0f;
        float factor = 15f / (Mathf.PI * Mathf.Pow(h, 6));
        return factor * Mathf.Pow(h - dist, 3);
    }

    void Update()
    {
        float h = smoothingRadius;

        // 1. Calculate Densities
        for (int i = 0; i < numParticles; i++)
        {
            float densitySum = 0f;
            for (int j = 0; j < numParticles; j++)
            {
                float dist = Vector2.Distance(particles[i].position, particles[j].position);
                densitySum += Kernel(dist, h);
            }
            particles[i].density = densitySum;
            
            // Calculate pressure: p = k * (rho - rho_0)
            particles[i].pressure = gasConstant * (particles[i].density - targetDensity);
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What physical equations govern fluid dynamics? :: The **Navier-Stokes equations**.
- What is the difference between Eulerian and Lagrangian fluid simulation? :: Eulerian tracks fluid passing through a static grid of space, while Lagrangian simulates moving particles carrying fluid properties.
- What does the SPH acronym stand for? :: **S**moothed **P**article **H**ydrodynamics.
- What is advection in fluid dynamics? :: The transport of fluid properties (like velocity, temperature, or smoke density) along with the fluid's own flow.
- What does viscosity measure? :: A fluid's internal friction or resistance to flow (thickness, e.g. honey has high viscosity, water has low).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using a naive $O(N^2)$ double loop to find neighbor particles in SPH simulations.
> - **The Fix:** Implement spatial hashing or grid partitioning to restrict neighbor searches to adjacent cells.
> - **Why:** A double loop over $2{,}000$ particles requires $4{,}000{,}000$ distance checks per frame, which will bottleneck the CPU. Spatial partitioning reduces this to local checks, keeping the simulation real-time.

---

## Related Topics
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
- [[Math/10_Physics_Math/forces_gravity_friction|Forces: Gravity, Friction & Springs]]
- [[Math/12_Advanced_Topics/particle_systems|Particle Systems]]
