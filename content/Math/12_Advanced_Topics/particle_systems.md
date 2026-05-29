---
title: "Particle Systems: Lifecycle, Emitters, Billboarding, and Simulation"
tags:
  - math
  - level/Lv3
  - category/advanced_topics
---

# Particle Systems: Lifecycle, Emitters, Billboarding, and Simulation

> [!abstract] **The Concept in a Nutshell**
> Many visual phenomena in video games — fire, smoke, sparks, explosions, rain, and magic spells — do not have solid geometry. Instead, they are simulated as **particle systems**: collections of thousands of tiny, independent coordinate points. Each particle undergoes a simple kinematic simulation (updating position, velocity, and color over its lifetime) and is typically rendered as a 2D quad that rotates to face the camera (called **billboarding**).

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Simulating Sparks Flying from a Sword Clash**
> When two swords clash in an action game, you want bright sparks to spray outward.
>
> To implement this, you instantiate a **Particle Emitter** at the point of impact:
> 1. The emitter spawns $100$ particles in a cone shape.
> 2. Each particle is initialized with a starting position (clash point), random velocity vector, and a random lifetime (e.g., $0.5$ to $1.2$ seconds).
> 3. Each frame, the system updates their positions: $\vec{P} = \vec{P} + \vec{v}\Delta t$, and applies gravity.
> 4. Over time, the system fades the particles from bright yellow to dark orange, and finally deletes them when their lifetime reaches zero.
>
> Because these calculations are simple, games can simulate tens of thousands of particles simultaneously on the GPU.

---

## The Blueprint (Formula & Structure)

```
        [ Emitter Cone ]
            /   |   \
           *    *    *   <-- Newly spawned particles (Full alpha)
          *     *     *
         *      *      *  <-- Moving under kinematics, fading out
        x       x       x <-- Lifetime = 0 (Destroyed)
```

### 1. Particle State Representation
A single particle is represented by a compact data structure:
$$\text{State} = \{ \vec{P}, \vec{v}, t_{\text{age}}, t_{\text{max\_age}}, \text{color}, \text{size} \}$$
Where $t_{\text{age}} / t_{\text{max\_age}}$ gives a normalized lifetime parameter $u \in [0, 1]$, which is used to interpolate size, speed, and color.

### 2. Kinematic Integration
Every frame, the active particles update their parameters:
$$t_{\text{age}} = t_{\text{age}} + \Delta t$$
$$\vec{v} = \vec{v} + \vec{a}_{\text{gravity}}\Delta t + \vec{a}_{\text{wind}}\Delta t$$
$$\vec{P} = \vec{P} + \vec{v}\Delta t$$
If $t_{\text{age}} \ge t_{\text{max\_age}}$, the particle is recycled or deactivated.

### 3. Billboard Rendering
To save rendering costs, particles are usually drawn as flat 2D textures (quads). To prevent players from seeing that they are flat sheets, the quads are rotated to face the camera directly.
- **Screen-Space Billboarding:** We copy the camera's rotation matrix directly to the quad's model matrix, ignoring the camera's Z translation.
- **World-Space Billboarding:** We align the quad's normal vector to point directly at the camera position:
  $$\vec{z}_{\text{quad}} = \text{normalize}\left(\vec{P}_{\text{camera}} - \vec{P}_{\text{particle}}\right)$$
  $$\vec{x}_{\text{quad}} = \text{normalize}\left(\vec{u}_{\text{world\_up}} \times \vec{z}_{\text{quad}}\right)$$
  $$\vec{y}_{\text{quad}} = \vec{z}_{\text{quad}} \times \vec{x}_{\text{quad}}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Fountain**
> Think of a particle system as a water fountain.
> - The nozzle is the **emitter**, defining the starting direction and spray cone.
> - The water droplets are the **particles**, each executing simple parabolic arcs under gravity.
> - As they fall, they evaporate (fade out) and vanish.
> - The collective stream of hundreds of droplets creates the illusion of a solid flowing arc of water.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simulating a basic custom CPU Particle System
using System.Collections.Generic;
using UnityEngine;

public class CustomParticleSystem : MonoBehaviour
{
    struct Particle
    {
        public Vector3 position;
        public Vector3 velocity;
        public float age;
        public float maxLifetime;
        public Color startColor;
        public Color endColor;
    }

    public int maxParticles = 500;
    public float emissionRate = 20f; // Particles per second
    public Vector3 gravity = new Vector3(0f, -4f, 0f);

    private List<Particle> activeParticles = new List<Particle>();
    private float emissionAccumulator = 0f;

    void Update()
    {
        // 1. Handle Spawning (Emission)
        emissionAccumulator += Time.deltaTime * emissionRate;
        while (emissionAccumulator >= 1.0f && activeParticles.Count < maxParticles)
        {
            SpawnParticle();
            emissionAccumulator -= 1.0f;
        }

        // 2. Simulate particles (Kinematics)
        float dt = Time.deltaTime;
        for (int i = activeParticles.Count - 1; i >= 0; i--)
        {
            Particle p = activeParticles[i];
            p.age += dt;

            if (p.age >= p.maxLifetime)
            {
                activeParticles.RemoveAt(i);
                continue;
            }

            // Update physics: v = v + a*dt, p = p + v*dt
            p.velocity += gravity * dt;
            p.position += p.velocity * dt;

            // Apply update back to list
            activeParticles[i] = p;
        }
    }

    private void SpawnParticle()
    {
        Particle p = new Particle
        {
            position = transform.position,
            // Random direction in a upward cone
            velocity = new Vector3(Random.Range(-1f, 1f), Random.Range(2f, 5f), Random.Range(-1f, 1f)),
            age = 0f,
            maxLifetime = Random.Range(1f, 2.5f),
            startColor = Color.yellow,
            endColor = Color.red
        };
        activeParticles.Add(p);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is billboarding in graphics rendering? :: Rotating flat 2D quads (billboards) so they always face the camera, giving the illusion of 3D volume.
- What core parameters define the state of a single particle? :: Position, velocity, age, maximum lifetime, size, and color.
- Why are particle systems highly optimized for GPU simulation? :: Because particles do not interact with one another, their calculations are completely independent (embarrassingly parallel), making them perfect for GPU threads.
- How is the normalized lifetime parameter calculated? :: $u = \frac{t_{\text{age}}}{t_{\text{max\_age}}}$, returning a value in the range $[0, 1]$.
- What forces commonly act on particles in games? :: Gravity, wind fields, fluid drag, and attractor forces.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Spawning new GameObjects in Unity for every individual particle.
> - **The Fix:** Use a dedicated particle buffer and draw them in a single draw call (like Unity's `ParticleSystem` or GPU Instancing).
> - **Why:** Creating and destroying GameObjects is extremely expensive. Instantiating hundreds of objects per second will trigger garbage collection, causing severe CPU lag and framing dips.

---

## Related Topics
- [[Math/10_Physics_Math/kinematics|Kinematics]]
- [[Math/05_Coordinate_Spaces/view_projection_space|View & Projection Space]]
- [[Math/12_Advanced_Topics/soft_body_cloth|Soft Body & Cloth Simulation]]
