---
title: "Collision Response: Impulse Resolution, Restitution, and Friction"
tags:
  - math
  - level/Lv3
  - category/physics_math
---

# Collision Response: Impulse Resolution, Restitution, and Friction

> [!abstract] **The Concept in a Nutshell**
> Once a collision is detected, the game engine must determine how the colliding objects react. **Collision response** resolves this by correcting object positions (preventing them from sinking into each other) and applying an **impulse force** that instantly changes their velocities. How bouncy the collision is depends on the **coefficient of restitution**, while **friction impulses** dictate how much objects slide or roll against each other.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Billiard Balls Colliding**
> In a pool/billiards game, when the cue ball hits the 8-ball, the cue ball should stop or deflect, and the 8-ball should roll forward.
>
> If you just invert their velocity vectors (e.g. `velocity = -velocity`), the interaction will look like a glitchy cartoon. 
> Instead, the physics engine calculates a contact normal ($\vec{n}$), the relative velocity between the balls ($\vec{v}_{\text{rel}}$), and applies an **impulse scalar $J$** along the contact normal:
> 1. Compute the separating velocity.
> 2. Determine the impulse magnitude using their masses and bounce settings.
> 3. Apply the impulse vector: $+\vec{J}$ to the 8-ball, and $-\vec{J}$ to the cue ball.
>
> This results in an exact, realistic momentum transfer that satisfies physical conservation laws.

---

## The Blueprint (Formula & Structure)

### 1. Relative Velocity
For two objects with velocities $\vec{v}_A$ and $\vec{v}_B$:
$$\vec{v}_{\text{rel}} = \vec{v}_B - \vec{v}_A$$
The component along the contact normal $\vec{n}$ is:
$$v_{\text{normal}} = \vec{v}_{\text{rel}} \cdot \vec{n}$$
- If $v_{\text{normal}} > 0$, the objects are moving apart (no response needed).
- If $v_{\text{normal}} < 0$, they are moving toward each other (collision response triggered).

### 2. Linear Impulse Formula (No Rotation)
The impulse scalar $j$ applied along the normal is calculated as:
$$j = \frac{-(1 + e) (\vec{v}_{\text{rel}} \cdot \vec{n})}{\frac{1}{m_A} + \frac{1}{m_B}}$$

Where:
- $e$: **Coefficient of Restitution** (elasticity/bounciness, range $[0, 1]$).
  - $e = 0$: Perfectly inelastic (objects stick together, e.g. clay).
  - $e = 1$: Perfectly elastic (no kinetic energy lost, e.g. superball).
- $m_A, m_B$: Masses of the two objects.

The impulse vectors applied are:
$$\vec{v}_A' = \vec{v}_A - \frac{j}{m_A}\vec{n}$$
$$\vec{v}_B' = \vec{v}_B + \frac{j}{m_B}\vec{n}$$

### 3. Penetration Resolution (Positional Correction)
Due to discrete time steps, objects often sink slightly into each other before a collision is detected. To prevent objects from sinking or getting stuck, we push them apart along the collision normal:
$$\text{Correction} = \frac{\text{Penetration Depth}}{\frac{1}{m_A} + \frac{1}{m_B}} \times \text{Percent} \times \vec{n}$$
We apply a small percentage scaler (e.g. $20\%$ to $80\%$) to prevent jittering artifacts (sometimes called **sinking correction** or **projection**).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Instant Push**
> Think of an impulse as an incredibly strong force applied over an infinitely short time (like a hammer strike). 
> - A standard force changes velocity gradually.
> - An impulse changes velocity *instantly*.
> 
> The coefficient of restitution determines how much of the incoming velocity is preserved. A bouncy rubber ball preserves $90\%$ ($e = 0.9$), while a lead weight preserves $0\%$ ($e = 0$).

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simple Impulse Collision Resolver (Linear, No Rotation)
using UnityEngine;

public class SimpleCollisionResolver : MonoBehaviour
{
    public struct CollisionManifold
    {
        public Rigidbody bodyA;
        public Rigidbody bodyB;
        public Vector3 normal; // Pointing from A to B
        public float penetration;
    }

    public float restitution = 0.5f; // Bounciness e
    public float positionalCorrectionPercent = 0.8f; // Slack correction
    public float positionalCorrectionEpsilon = 0.01f; // Penetration allowance

    public void ResolveCollision(CollisionManifold manifold)
    {
        Rigidbody a = manifold.bodyA;
        Rigidbody b = manifold.bodyB;
        Vector3 n = manifold.normal;

        // 1. Calculate relative velocity
        Vector3 velA = a != null ? a.velocity : Vector3.zero;
        Vector3 velB = b != null ? b.velocity : Vector3.zero;
        Vector3 relativeVelocity = velB - velA;

        // 2. Calculate separating velocity along normal
        float velAlongNormal = Vector3.Dot(relativeVelocity, n);

        // Do not resolve if velocities are already separating
        if (velAlongNormal > 0) return;

        // Calculate inverse masses (treat static/kinematic objects as infinite mass)
        float invMassA = a != null ? 1f / a.mass : 0f;
        float invMassB = b != null ? 1f / b.mass : 0f;
        float totalInvMass = invMassA + invMassB;

        if (totalInvMass == 0) return; // Both objects are static

        // 3. Compute Impulse scalar j
        float e = restitution;
        float j = -(1f + e) * velAlongNormal;
        j /= totalInvMass;

        // 4. Apply impulse to each object
        Vector3 impulseVector = j * n;
        if (a != null) a.velocity -= invMassA * impulseVector;
        if (b != null) b.velocity += invMassB * impulseVector;

        // 5. Penetration Resolution (Positional correction to prevent sinking)
        float correctionMagnitude = Mathf.Max(manifold.penetration - positionalCorrectionEpsilon, 0.0f) / totalInvMass * positionalCorrectionPercent;
        Vector3 correctionVector = correctionMagnitude * n;

        if (a != null) a.transform.position -= invMassA * correctionVector;
        if (b != null) b.transform.position += invMassB * correctionVector;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the Coefficient of Restitution ($e$) measure? :: The ratio of final relative separating speed to initial relative closing speed, indicating bounciness (range $[0, 1]$).
- What happens in a collision when the coefficient of restitution is $e = 0$? :: The collision is perfectly inelastic; the objects stop separating and stick together along the contact normal.
- What is an impulse in physics simulation? :: A sudden, instantaneous change in velocity, calculated as force multiplied by time ($\vec{J} = \vec{F}\Delta t$).
- Why do physics engines require positional correction (sinking correction)? :: Rounding errors and discrete timesteps cause objects to sink into each other; positional correction pushes them apart to prevent clipping.
- Why is an object's inverse mass ($1/m$) used in collision response calculations instead of mass ($m$)? :: Because static obstacles (walls, terrain) can be treated mathematically as having infinite mass, which results in a clean inverse mass of $0$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Resolving velocity changes without checking if objects are already separating (e.g. ignoring `if (velAlongNormal > 0)`).
> - **The Fix:** Always guard your resolver so it only runs if the relative velocity is pointing *into* the contact normal.
> - **Why:** If you skip this check, the physics engine will apply a separation impulse *again* on the next frame as the objects are already flying apart, resulting in sticky collisions or objects getting sucked back together.

---

## Related Topics
- [[Math/03_Vectors/vector_projection|Vector Projection]]
- [[Math/10_Physics_Math/collision_detection|Collision Detection]]
- [[Math/10_Physics_Math/rigid_body_dynamics|Rigid Body Dynamics]]
