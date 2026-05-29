---
title: "Vector Operations: Combining Forces & Finding Directions"
tags:
  - math
  - level/Lv2
  - category/vectors
---

# Vector Operations: Combining Forces & Finding Directions

> [!abstract] **The Concept in a Nutshell**
> Vector operations — addition, subtraction, scalar multiplication, and negation — are the core arithmetic of game math. They let you combine forces, compute directions between objects, scale velocities, and reverse headings. Every frame of every game performs hundreds of these operations to drive movement, physics, and AI.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Sailing Ship in a Storm**
> In *Corsair's Wake*, the player's galleon has an engine thrust vector $\vec{F}_{\text{engine}} = (5, 0, 3)$, a wind force $\vec{F}_{\text{wind}} = (-2, 0, 4)$, and an ocean current $\vec{F}_{\text{current}} = (1, 0, -1)$. The net force on the ship is their **vector sum**: $\vec{F}_{\text{net}} = (5, 0, 3) + (-2, 0, 4) + (1, 0, -1) = (4, 0, 6)$. The captain wants to steer toward a lighthouse at position $\vec{L} = (50, 10, 80)$ from the ship at $\vec{S} = (10, 2, 20)$ — the direction is the **vector subtraction** $\vec{L} - \vec{S} = (40, 8, 60)$. To apply a speed boost of $2\times$, we **scalar-multiply**: $2 \cdot \vec{F}_{\text{engine}} = (10, 0, 6)$.

---

## The Blueprint (Formula & Structure)

### Vector Addition
$$\vec{a} + \vec{b} = (a_x + b_x, \; a_y + b_y, \; a_z + b_z)$$

**Geometric interpretation:** Place the tail of $\vec{b}$ at the head of $\vec{a}$. The result is the arrow from the tail of $\vec{a}$ to the head of $\vec{b}$ (tip-to-tail method).

**Properties:**
- Commutative: $\vec{a} + \vec{b} = \vec{b} + \vec{a}$
- Associative: $(\vec{a} + \vec{b}) + \vec{c} = \vec{a} + (\vec{b} + \vec{c})$
- Identity: $\vec{a} + \vec{0} = \vec{a}$

### Vector Subtraction
$$\vec{a} - \vec{b} = (a_x - b_x, \; a_y - b_y, \; a_z - b_z)$$

**Geometric interpretation:** $\vec{a} - \vec{b}$ is the vector pointing **from** $\vec{b}$ **to** $\vec{a}$. Equivalently, $\vec{a} + (-\vec{b})$.

### The Parallelogram Rule
When adding two vectors $\vec{a}$ and $\vec{b}$ that share the same starting point, the sum $\vec{a} + \vec{b}$ is the **diagonal** of the parallelogram formed by $\vec{a}$ and $\vec{b}$. The other diagonal gives $\vec{a} - \vec{b}$.

### Scalar Multiplication
$$k \cdot \vec{a} = (k \cdot a_x, \; k \cdot a_y, \; k \cdot a_z)$$

- $k > 1$: Stretches (lengthens) the vector
- $0 < k < 1$: Shrinks (shortens) the vector
- $k = 0$: Produces the zero vector $\vec{0}$
- $k < 0$: Reverses direction AND scales

**Properties:**
- Distributive: $k(\vec{a} + \vec{b}) = k\vec{a} + k\vec{b}$
- Associative: $(jk)\vec{a} = j(k\vec{a})$

### Negation
$$-\vec{a} = (-a_x, -a_y, -a_z)$$

**Geometric interpretation:** Same length, opposite direction. A 180° flip.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Tug-of-War Rope**
> Imagine vector addition as a tug-of-war with multiple ropes tied to a ring. Each rope pulls in a different direction with a different strength. The ring's actual movement — the net result — is the **vector sum** of all the pulls. If the wind pulls northeast and the current pulls south, the boat goes somewhere in between, determined by the combined arrow. Scalar multiplication is simply one team getting stronger (multiplied force) without changing which direction they pull — unless the scalar is negative, in which case they switch sides entirely.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Vector operations in a sailing simulation
using UnityEngine;

public class VectorOperations : MonoBehaviour
{
    public Transform lighthouse;
    public float speedBoost = 1f;

    void Update()
    {
        // --- VECTOR ADDITION: Combining multiple forces ---
        Vector3 engineForce  = new Vector3(5f, 0f, 3f);
        Vector3 windForce    = new Vector3(-2f, 0f, 4f);
        Vector3 currentForce = new Vector3(1f, 0f, -1f);

        Vector3 netForce = engineForce + windForce + currentForce;
        // netForce = (4, 0, 6)

        // --- VECTOR SUBTRACTION: Direction from ship to lighthouse ---
        Vector3 shipPos = transform.position;
        Vector3 lighthousePos = lighthouse.position;
        Vector3 towardLighthouse = lighthousePos - shipPos;

        // --- SCALAR MULTIPLICATION: Applying speed boost ---
        Vector3 boostedEngine = speedBoost * engineForce;

        // --- NEGATION: Reversing a direction ---
        Vector3 retreatDir = -towardLighthouse; // Run away from the lighthouse!

        // Apply net force as velocity
        Vector3 totalForce = boostedEngine + windForce + currentForce;
        transform.position += totalForce * Time.deltaTime;

        // Visualize forces
        Debug.DrawRay(shipPos, engineForce, Color.green);
        Debug.DrawRay(shipPos, windForce, Color.cyan);
        Debug.DrawRay(shipPos, currentForce, Color.blue);
        Debug.DrawRay(shipPos, netForce, Color.red);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does $\vec{B} - \vec{A}$ represent geometrically? :: The vector pointing **from** point A **to** point B. It gives the direction and distance from A to B.
- Is vector addition commutative? Is subtraction? :: Addition is commutative ($\vec{a} + \vec{b} = \vec{b} + \vec{a}$). Subtraction is NOT ($\vec{a} - \vec{b} \neq \vec{b} - \vec{a}$, they point in opposite directions).
- What happens when you scalar-multiply a vector by $-2$? :: The vector reverses direction (180° flip) and doubles in length.
- How do you combine multiple forces acting on a game object? :: Add all force vectors together: $\vec{F}_{\text{net}} = \vec{F}_1 + \vec{F}_2 + \cdots + \vec{F}_n$. The result is the net force that determines actual movement.
- What is the parallelogram rule? :: When two vectors share a starting point, their sum is the diagonal of the parallelogram they form. The other diagonal represents their difference.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Subtracting vectors in the wrong order: writing `playerPos - targetPos` when you want the direction *toward* the target.
> - **The Fix:** Always think: "direction = **destination** minus **origin**." To go from A to B, compute $\vec{B} - \vec{A}$.
> - **Why:** Reversing the order gives a vector pointing in the **opposite** direction. Your AI enemy would run away from the player instead of chasing them!

---

## Related Topics
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
- [[Math/03_Vectors/magnitude_normalization|Magnitude & Normalization]]
