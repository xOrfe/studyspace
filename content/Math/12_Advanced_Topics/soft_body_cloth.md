---
title: "Soft Body & Cloth Simulation: Mass-Spring Networks and Constraints"
tags:
  - math
  - level/Lv4
  - category/advanced_topics
---

# Soft Body & Cloth Simulation: Mass-Spring Networks and Constraints

> [!abstract] **The Concept in a Nutshell**
> Unlike rigid bodies (which cannot deform), **soft bodies** and **cloth** bend, stretch, and wiggle. To simulate these elastic objects, we represent them as a network of mass points (particles) connected by virtual springs. By structuring these springs in specific patterns (structural, shear, and bending springs) and solving the network equations, we can simulate realistic waving flags, character capes, jelly physics, and deformable cushions.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Simulating a Waving Flag or Character Cape**
> You want to add a realistic cape to your main character that drags behind them, drapes over obstacles, and flutters in the wind.
>
> If you animate the cape using hand-crafted bones, it will look repetitive and fail to react to dynamic game forces (like explosions or wind).
> Instead, you model the cape mesh as a grid of points:
> 1. Each vertex is a particle with mass.
> 2. Vertices are connected to their neighbors by virtual springs.
> 3. Gravity, wind force, and character movements act on the particles.
> 4. The solver resolves the spring forces to keep the cloth from stretching into infinity, resulting in realistic, dynamic cloth physics.

---

## The Blueprint (Formula & Structure)

### 1. The Mass-Spring Network
A cloth mesh is represented by a grid of particles connected by three distinct types of springs to handle different forms of deformation:

```
        V0---S---V1---S---V2
        | \     / | \     / |
        S  Sh  Sh S  Sh  Sh S
        |   \ /   |   \ /   |
        V3---S---V4---S---V5
```

1. **Structural Springs (S):** Connect vertices directly adjacent horizontally and vertically. Resists stretching and compression.
2. **Shear Springs (Sh):** Connect vertices diagonally. Prevents the cloth from shearing (collapsing into diamond shapes).
3. **Bending Springs (B):** Connect vertices separated by one step (e.g. $V_0$ to $V_2$). Resists folding and bending, giving the cloth stiffness.

### 2. Spring Force Calculations
For each spring connecting particle $i$ at position $\vec{p}_i$ and particle $j$ at position $\vec{p}_j$:
1. Calculate distance vector: $\vec{x} = \vec{p}_j - \vec{p}_i$
2. Calculate current length: $L = |\vec{x}|$
3. Calculate Hooke's elastic restoring force:
   $$\vec{F}_{\text{spring}} = k \left( L - L_{\text{rest}} \right) \frac{\vec{x}}{L}$$
Where $k$ is the spring stiffness and $L_{\text{rest}}$ is the starting length of the spring.

### 3. Position-Based Dynamics (PBD)
In modern games, instead of calculating spring *forces* and using integration (which is highly unstable and explodes easily), we use **Position-Based Dynamics (PBD)**. 
PBD projects positions directly to satisfy distance constraints:
$$\Delta \vec{p}_i = \frac{w_i}{w_i + w_j} \left( L - L_{\text{rest}} \right) \frac{\vec{x}}{L}$$
Where $w_i = 1/m_i$ is the inverse mass. PBD is extremely stable and prevents cloth from exploding even under intense stretching.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Rubber Grid**
> Think of cloth as a grid of small lead weights connected by rubber bands.
> - If you only have horizontal/vertical bands (structural), you can easily twist the grid sideways.
> - Adding diagonal bands (shear) keeps the grid square.
> - Adding longer bands that skip a weight (bending) keeps the grid flat, preventing it from folding like paper.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simulating a 1D String of Spring-Mass Particles (Rope / Hair)
using UnityEngine;

public class MassSpringRope : MonoBehaviour
{
    struct Particle
    {
        public Vector3 position;
        public Vector3 oldPosition;
        public Vector3 acceleration;
        public bool isStatic;
    }

    struct Spring
    {
        public int indexA;
        public int indexB;
        public float restLength;
    }

    public int numParticles = 10;
    public float segmentLength = 0.5f;
    public float stiffness = 100f;
    public Vector3 gravity = new Vector3(0f, -9.81f, 0f);

    private Particle[] particles;
    private Spring[] springs;

    void Start()
    {
        particles = new Particle[numParticles];
        springs = new Spring[numParticles - 1];

        // Initialize particles in a vertical line
        for (int i = 0; i < numParticles; i++)
        {
            particles[i].position = transform.position + Vector3.down * (i * segmentLength);
            particles[i].oldPosition = particles[i].position;
            particles[i].isStatic = (i == 0); // Anchor the top point
        }

        // Initialize springs
        for (int i = 0; i < numParticles - 1; i++)
        {
            springs[i].indexA = i;
            springs[i].indexB = i + 1;
            springs[i].restLength = segmentLength;
        }
    }

    void FixedUpdate()
    {
        // 1. Apply External Forces (Verlet Integration)
        for (int i = 0; i < numParticles; i++)
        {
            if (particles[i].isStatic) continue;

            particles[i].acceleration = gravity;
            Vector3 temp = particles[i].position;
            
            // Verlet Integration: p_new = p + (p - p_old) + a * dt^2
            particles[i].position += (particles[i].position - particles[i].oldPosition) + particles[i].acceleration * Time.fixedDeltaTime * Time.fixedDeltaTime;
            particles[i].oldPosition = temp;
        }

        // 2. Resolve Spring Constraints (Relaxation iterations)
        for (int iteration = 0; iteration < 5; iteration++)
        {
            for (int i = 0; i < springs.Length; i++)
            {
                Spring spring = springs[i];
                Vector3 offset = particles[spring.indexB].position - particles[spring.indexA].position;
                float currentLength = offset.magnitude;
                Vector3 direction = offset.normalized;
                
                float difference = spring.restLength - currentLength;
                Vector3 translation = direction * difference * 0.5f;

                if (!particles[spring.indexA].isStatic) particles[spring.indexA].position -= translation;
                if (!particles[spring.indexB].isStatic) particles[spring.indexB].position += translation;
            }
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What are the three types of springs used in cloth simulations? :: **Structural** (stretching), **Shear** (skewing), and **Bending** (folding) springs.
- What is the difference between force-based mass-spring systems and Position-Based Dynamics (PBD)? :: Force-based systems integrate spring forces (often unstable), while PBD adjusts particle coordinates directly to satisfy distance constraints (highly stable).
- In cloth simulation, what does a Shear Spring prevent? :: It prevents the cloth from shearing or collapsing diagonally into diamond shapes.
- How does Verlet integration calculate velocity? :: Velocity is calculated implicitly by subtracting the particle's old position from its current position: $\vec{v} \approx \vec{p} - \vec{p}_{\text{old}}$.
- Why does standard Euler integration fail for stiff springs in games? :: Stiff springs create large forces that oscillate rapidly, causing the integration step to overshoot and blow up the simulation.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using standard Euler integration (adding forces directly to velocity) for cloth simulations with high spring stiffness.
> - **The Fix:** Use Verlet integration paired with Position-Based Dynamics (PBD) constraint iterations.
> - **Why:** Euler integration requires tiny timesteps to remain stable under stiff forces. If the framerate dips, the simulation accumulates massive forces that blow the mesh apart. Verlet + PBD is stable regardless of timestep duration.

---

## Related Topics
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
- [[Math/10_Physics_Math/forces_gravity_friction|Forces: Gravity, Friction & Springs]]
- [[Math/12_Advanced_Topics/particle_systems|Particle Systems]]
