---
title: "Vector Fundamentals: The Language of Movement"
tags:
  - math
  - level/Lv2
  - category/vectors
---

# Vector Fundamentals: The Language of Movement

> [!abstract] **The Concept in a Nutshell**
> A vector is a mathematical object with both **magnitude** (length) and **direction**. In game development, vectors are the fundamental building blocks for representing positions, velocities, forces, and directions — essentially anything that needs "how much" and "which way." Understanding vectors unlocks the ability to move characters, aim projectiles, and simulate physics.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Open-World RPG Character Navigation**
> In *Chronicles of Avalon*, the player character Kael stands at position $\vec{P} = (3, 0, 7)$ in the world. A treasure chest sits at $\vec{T} = (10, 0, 12)$. The game needs a **position vector** for Kael (where he is), and a **direction vector** $\vec{D} = \vec{T} - \vec{P} = (7, 0, 5)$ pointing from Kael toward the chest. Meanwhile, Kael's velocity vector $\vec{v} = (2, 0, 1)$ describes that he moves 2 units/sec on X and 1 unit/sec on Z. Without vectors, none of this — pathfinding, movement, quest markers pointing at objectives — would be possible.

---

## The Blueprint (Formula & Structure)

### Definition
A **vector** in $n$-dimensional space is an ordered tuple of $n$ real numbers:

$$\vec{v} = (v_x, v_y, v_z) \in \mathbb{R}^3$$

Each component represents displacement along one axis.

### Components
| Component | 2D | 3D | Meaning |
|---|---|---|---|
| $v_x$ | Horizontal | Right/Left | Displacement along X-axis |
| $v_y$ | Vertical | Up/Down | Displacement along Y-axis |
| $v_z$ | — | Forward/Back | Displacement along Z-axis |

### Position Vectors vs Direction Vectors
- **Position vector:** Points from the origin to a specific location. Represents *where* something is.
  $$\vec{P} = (x, y, z) \quad \text{(anchored at origin)}$$
- **Direction vector:** Represents *which way* and *how far*, with no fixed starting point.
  $$\vec{D} = \vec{B} - \vec{A} \quad \text{(from point A toward point B)}$$

### Free Vectors vs Bound Vectors
- **Free vector:** Has magnitude and direction only — can be placed anywhere in space. Most vectors in game math are free vectors (velocity, force, normals).
- **Bound vector (applied vector):** Tied to a specific point of application. A position vector is technically a bound vector anchored at the origin.

### Geometric vs Algebraic Interpretation
- **Geometric:** A vector is an arrow in space with a tail and a head. Length = magnitude, arrow direction = direction.
- **Algebraic:** A vector is a column (or row) of numbers that you can add, scale, and transform with formulas.

Both views are essential: the geometric view builds intuition, while the algebraic view enables computation.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Treasure Map Arrow**
> Think of a vector as an arrow drawn on a treasure map. The arrow tells you two things: *how far to walk* (magnitude) and *which direction to walk* (direction). A **position vector** is like GPS coordinates — "the treasure is HERE." A **direction vector** is like the instruction "walk 5 steps northeast" — it doesn't care where you start. If you pick up the arrow and place it somewhere else on the map without rotating or stretching it, you have the same free vector. That's the key insight: free vectors are portable instructions.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Working with position and direction vectors
using UnityEngine;

public class VectorFundamentals : MonoBehaviour
{
    public Transform treasureChest;

    void Update()
    {
        // POSITION VECTOR: where the player is in world space
        Vector3 playerPos = transform.position; // e.g., (3, 0, 7)

        // POSITION VECTOR: where the treasure is
        Vector3 chestPos = treasureChest.position; // e.g., (10, 0, 12)

        // DIRECTION VECTOR: from player toward treasure
        Vector3 toChest = chestPos - playerPos; // = (7, 0, 5)

        // Access individual components
        float horizontalDist = toChest.x; // 7
        float verticalDist   = toChest.y; // 0
        float depthDist      = toChest.z; // 5

        // Visualize the direction vector as a debug ray
        Debug.DrawRay(playerPos, toChest, Color.yellow);

        // VELOCITY VECTOR: direction + speed combined
        Vector3 velocity = new Vector3(2f, 0f, 1f); // 2 units/s on X, 1 on Z
        transform.position += velocity * Time.deltaTime;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What two properties define a vector? :: **Magnitude** (length/size) and **direction**. A scalar only has magnitude.
- How do you compute the direction vector from point A to point B? :: $\vec{D} = \vec{B} - \vec{A}$. Subtract the start from the destination.
- What is a free vector? :: A vector that has magnitude and direction but no fixed position — it can be translated anywhere without changing its identity.
- What is the difference between a position vector and a direction vector in Unity? :: A position vector (like `transform.position`) represents a point relative to the origin. A direction vector (like `target.position - transform.position`) represents displacement with no anchor point.
- In a left-handed coordinate system (Unity), which direction does +Z point? :: Forward, into the screen. Unity uses a left-handed Y-up system where +X is right, +Y is up, and +Z is forward.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Treating a position as a direction or vice versa. For example, using `transform.position` directly as a movement direction.
> - **The Fix:** Always compute a direction by *subtracting* two positions: `Vector3 dir = target.position - transform.position;` Then normalize if you only need the direction.
> - **Why:** A position like $(100, 50, 200)$ is NOT a meaningful direction — it's a point in space. Using it as a direction would send objects flying toward some arbitrary direction relative to the origin instead of toward the target.

---

## Related Topics
- [[Math/03_Vectors/vector_operations|Vector Operations]]
- [[Math/01_Foundations/coordinate_systems_2d|2D Coordinate Systems]]
- [[Math/01_Foundations/coordinate_systems_3d|3D Coordinate Systems]]
