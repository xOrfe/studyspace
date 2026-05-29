---
title: "Vector Projection: Decomposing Motion Along Surfaces"
tags:
  - math
  - level/Lv2
  - category/vectors
---

# Vector Projection: Decomposing Motion Along Surfaces

> [!abstract] **The Concept in a Nutshell**
> **Vector projection** decomposes a vector into two components: one that lies along a given direction (the **projection**) and one that is perpendicular to it (the **rejection**). In game development, this is the math behind wall sliding, slope physics, reflecting off surfaces, and separating forces into useful components. If the dot product asks "how aligned are these?", projection answers "show me exactly the aligned part."

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Wall Sliding in an FPS**
> In *Iron Corridor*, the player runs diagonally into a wall at velocity $\vec{v} = (3, 0, 4)$. The wall's surface normal is $\hat{n} = (1, 0, 0)$ (pointing away from the wall). We need to strip out the component of velocity going INTO the wall (the part along $\hat{n}$) and keep the component sliding ALONG it. The projection of $\vec{v}$ onto $\hat{n}$ is $(3, 0, 0)$ — that's the "into-the-wall" component. The **rejection** (remainder) is $\vec{v} - (3, 0, 0) = (0, 0, 4)$ — the player slides smoothly along the wall at speed 4 on the Z-axis. Without projection, the player would either stop dead or clip through the wall.

---

## The Blueprint (Formula & Structure)

### Scalar Projection
The scalar projection of $\vec{a}$ onto $\vec{b}$ gives a single number — how far along $\vec{b}$ the vector $\vec{a}$ reaches:

$$\text{comp}_{\vec{b}}\vec{a} = \frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|}$$

If $\vec{b}$ is already a unit vector $\hat{b}$:

$$\text{comp}_{\hat{b}}\vec{a} = \vec{a} \cdot \hat{b}$$

- **Positive value:** $\vec{a}$ has a component in the same direction as $\vec{b}$
- **Negative value:** $\vec{a}$ has a component opposite to $\vec{b}$
- **Zero:** $\vec{a}$ is perpendicular to $\vec{b}$

### Vector Projection
The vector projection of $\vec{a}$ onto $\vec{b}$ gives the actual vector component of $\vec{a}$ along $\vec{b}$:

$$\text{proj}_{\vec{b}}\vec{a} = \frac{\vec{a} \cdot \vec{b}}{\vec{b} \cdot \vec{b}} \vec{b} = \frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|^2} \vec{b}$$

If $\vec{b}$ is a unit vector $\hat{b}$, this simplifies to:

$$\text{proj}_{\hat{b}}\vec{a} = (\vec{a} \cdot \hat{b})\hat{b}$$

### Vector Rejection (Perpendicular Component)
The rejection is the "leftover" — the component of $\vec{a}$ perpendicular to $\vec{b}$:

$$\text{rej}_{\vec{b}}\vec{a} = \vec{a} - \text{proj}_{\vec{b}}\vec{a}$$

### Component Decomposition
Any vector $\vec{a}$ can be split into two perpendicular parts relative to a direction $\vec{b}$:

$$\vec{a} = \underbrace{\text{proj}_{\vec{b}}\vec{a}}_{\text{parallel to } \vec{b}} + \underbrace{\text{rej}_{\vec{b}}\vec{a}}_{\text{perpendicular to } \vec{b}}$$

This is an **orthogonal decomposition** — the two parts are at 90° to each other.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Shadow on a Ruler**
> Hold a pencil (vector $\vec{a}$) at an angle above a ruler (vector $\vec{b}$) and shine a light straight down. The shadow of the pencil on the ruler is the **projection** — the part of $\vec{a}$ that lies along $\vec{b}$. The vertical gap between the pencil tip and its shadow is the **rejection** — the part perpendicular to $\vec{b}$. Tilting the pencil to be parallel to the ruler makes the shadow match the full length (max projection). Holding it straight up makes zero shadow (zero projection, full rejection). The pencil itself is always the sum of its shadow plus the gap.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Wall sliding and slope physics using vector projection
using UnityEngine;

public class VectorProjectionDemo : MonoBehaviour
{
    public float moveSpeed = 5f;
    public float slopeLimit = 45f;

    void Update()
    {
        // === WALL SLIDING ===
        Vector3 velocity = transform.forward * moveSpeed;

        // Simulate hitting a wall with this normal
        Vector3 wallNormal = Vector3.right; // Wall faces +X

        // Project velocity onto wall normal (the "into the wall" part)
        float intoWall = Vector3.Dot(velocity, wallNormal);

        if (intoWall < 0) // Only slide if moving INTO the wall
        {
            // Remove the "into-wall" component, keep the sliding component
            Vector3 slideVelocity = velocity - intoWall * wallNormal;
            // slideVelocity is the REJECTION — the part parallel to the wall
            transform.position += slideVelocity * Time.deltaTime;
        }
        else
        {
            transform.position += velocity * Time.deltaTime;
        }
    }

    // === SLOPE PHYSICS ===
    Vector3 GetSlopeVelocity(Vector3 velocity, Vector3 groundNormal)
    {
        // Project velocity onto the slope surface
        // First, find the "into-ground" component
        float intoGround = Vector3.Dot(velocity, groundNormal);

        // Rejection = movement along the slope surface
        Vector3 slopeVelocity = velocity - intoGround * groundNormal;

        // Check slope angle
        float slopeAngle = Vector3.Angle(groundNormal, Vector3.up);
        if (slopeAngle > slopeLimit)
        {
            // Too steep! Add gravity slide component
            Vector3 gravityProjection = Vector3.ProjectOnPlane(
                Physics.gravity, groundNormal);
            slopeVelocity += gravityProjection * Time.deltaTime;
        }

        return slopeVelocity;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the formula for projecting $\vec{a}$ onto unit vector $\hat{b}$? :: $\text{proj}_{\hat{b}}\vec{a} = (\vec{a} \cdot \hat{b})\hat{b}$. Dot the vector with the unit direction, then scale that direction by the result.
- What is a vector rejection? :: The component of $\vec{a}$ perpendicular to $\vec{b}$: $\text{rej}_{\vec{b}}\vec{a} = \vec{a} - \text{proj}_{\vec{b}}\vec{a}$. It's the "leftover" after removing the projected part.
- How does wall sliding work mathematically? :: Remove the velocity component going into the wall: `slideVelocity = velocity - Vector3.Dot(velocity, wallNormal) * wallNormal`. The result is the rejection — movement parallel to the wall surface.
- What Unity method performs projection onto a plane? :: `Vector3.ProjectOnPlane(vector, planeNormal)` returns the vector component that lies on the plane (i.e., the rejection from the normal direction).
- How do projection and rejection relate to the original vector? :: They form an orthogonal decomposition: $\vec{a} = \text{proj}_{\vec{b}}\vec{a} + \text{rej}_{\vec{b}}\vec{a}$. The two parts are perpendicular and sum to the original.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to check the sign of the dot product before applying wall sliding. If the player moves AWAY from the wall ($\vec{v} \cdot \hat{n} > 0$), subtracting the projection would *pull* them back toward the wall.
> - **The Fix:** Only remove the normal component when `Vector3.Dot(velocity, wallNormal) < 0` (moving into the wall). If positive, the player is already moving away — apply velocity normally.
> - **Why:** The projection formula works mathematically in both cases, but game logic requires you to only "slide" when there's actual contact. Applying it unconditionally creates a sticky wall that traps players.

---

## Related Topics
- [[Math/03_Vectors/dot_product|Dot Product]]
- [[Math/10_Physics_Math/collision_response|Collision Response]]
- [[Math/07_Geometric_Primitives/planes_implicit|Planes & Half-Spaces]]
