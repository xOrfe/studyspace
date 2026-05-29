---
title: "Boolean Logic & Sets: The Decision Engine Behind Every Game"
tags:
  - math
  - level/Lv1
  - category/foundations
---

# Boolean Logic & Sets: The Decision Engine Behind Every Game

> [!abstract] **The Concept in a Nutshell**
> Boolean algebra is the mathematics of true/false decisions — and games are *made* of decisions. Can the player jump? Is the enemy visible? Does this bullet hit that armor layer? Beyond simple booleans, **bitwise operations** pack dozens of flags into a single integer, **set theory** governs collision layer interactions, and **graph theory** underpins pathfinding. These are the tools that make game logic efficient and expressive.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The Platformer's Input Logic**
> In "Pixel Knight," the player presses the jump button. But should the character actually jump? The game checks:
>
> ```
> canJump = isGrounded AND !isStunned AND (stamina > 0) AND !isCrouching
> ```
>
> That's four boolean conditions combined with AND and NOT. But there's more — the game uses a **layer mask** to determine what counts as "ground." The ground check raycast should hit layers `Ground` (bit 3) and `Platform` (bit 7), but NOT `Water` (bit 4) or `Enemy` (bit 6). This is done with a **bitmask:**
>
> ```
> groundMask = (1 << 3) | (1 << 7) = 0b10001000 = 136
> ```
>
> The raycast checks: `if (hitLayer & groundMask) != 0` — a single bitwise AND replaces four separate layer comparisons. Meanwhile, the level's navigation graph — nodes connected by edges — tells the AI which rooms connect to which, enabling A* pathfinding.

---

## The Blueprint (Formula & Structure)

### Boolean Operators

| Operator | Symbol | Math | Description |
|----------|--------|------|-------------|
| AND | `&&` | $A \land B$ | True only when BOTH are true |
| OR | `\|\|` | $A \lor B$ | True when AT LEAST ONE is true |
| NOT | `!` | $\lnot A$ | Flips true↔false |
| XOR | `^` | $A \oplus B$ | True when EXACTLY ONE is true |

### Truth Tables

| $A$ | $B$ | $A \land B$ | $A \lor B$ | $A \oplus B$ | $\lnot A$ |
|-----|-----|-------------|------------|--------------|-----------|
| 0 | 0 | 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 |

### De Morgan's Laws
$$\lnot(A \land B) = \lnot A \lor \lnot B$$
$$\lnot(A \lor B) = \lnot A \land \lnot B$$

In code: `!(a && b)` is the same as `(!a || !b)`.

### Bitwise Operations (Bitmask Flags)
Instead of storing 32 separate booleans, pack them into one `int` — each bit is a flag:

| Operation | Code | Effect |
|-----------|------|--------|
| Set flag | `flags \|= (1 << n)` | Turns on bit $n$ |
| Clear flag | `flags &= ~(1 << n)` | Turns off bit $n$ |
| Toggle flag | `flags ^= (1 << n)` | Flips bit $n$ |
| Check flag | `(flags & (1 << n)) != 0` | Tests if bit $n$ is on |

### Set Theory Basics

| Concept | Notation | Game Example |
|---------|----------|--------------|
| Union | $A \cup B$ | All layers that either collider can interact with |
| Intersection | $A \cap B$ | Layers that BOTH colliders share — determines if collision happens |
| Difference | $A \setminus B$ | Layers in A but not in B |
| Complement | $\overline{A}$ | Every layer NOT in A |

In collision systems: two objects collide if $\text{layerA} \cap \text{layerB} \neq \emptyset$, which is just `(maskA & maskB) != 0`.

### Graph Theory Basics

A **graph** $G = (V, E)$ consists of:
- **Vertices (nodes)** $V$: positions, rooms, waypoints
- **Edges** $E$: connections between vertices (weighted = distance/cost)

Key concepts:
- **Path:** sequence of edges connecting two nodes
- **Adjacency:** two nodes directly connected by an edge
- **Degree:** number of edges on a node
- **Directed vs. undirected:** one-way vs. two-way connections (one-way doors!)

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Nightclub Bouncer**
> Think of boolean logic like a series of bouncers at a nightclub:
>
> - **AND gate:** Two bouncers stand at the door. Both must approve you. If either says no, you don't get in. ("Must be on the guest list AND have valid ID.")
> - **OR gate:** Two side doors, each with a bouncer. Getting past either one lets you in. ("VIP pass OR regular ticket.")
> - **NOT gate:** A bouncer who lets through everyone who *isn't* on the ban list.
> - **XOR gate:** A revolving door that only works when exactly one person pushes. If both push or neither pushes, it stays stuck.
>
> **Bitmasks** are like a row of 32 light switches. Each switch represents a flag (isFlying, isInvisible, hasShield...). Instead of checking each switch individually, you can compare the *entire panel* of switches at once with a single bitwise operation — this is why bitmasks are so fast.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Boolean logic, bitmasks, and graph basics
using UnityEngine;
using System.Collections.Generic;

public class BooleanLogicDemo : MonoBehaviour
{
    // --- Boolean Game State Conditions ---
    bool isGrounded = true;
    bool isStunned = false;
    float stamina = 50f;
    bool isCrouching = false;

    bool CanJump()
    {
        return isGrounded && !isStunned && stamina > 0f && !isCrouching;
    }

    // --- Bitmask Flags for Status Effects ---
    [System.Flags]
    public enum StatusEffect
    {
        None      = 0,
        Poisoned  = 1 << 0,  // bit 0 = 1
        Burning   = 1 << 1,  // bit 1 = 2
        Frozen    = 1 << 2,  // bit 2 = 4
        Stunned   = 1 << 3,  // bit 3 = 8
        Shielded  = 1 << 4,  // bit 4 = 16
        Invisible = 1 << 5   // bit 5 = 32
    }

    StatusEffect currentEffects = StatusEffect.None;

    void ApplyEffect(StatusEffect effect) => currentEffects |= effect;
    void RemoveEffect(StatusEffect effect) => currentEffects &= ~effect;
    void ToggleEffect(StatusEffect effect) => currentEffects ^= effect;
    bool HasEffect(StatusEffect effect) => (currentEffects & effect) != 0;

    // --- Layer Mask for Raycasting ---
    void GroundCheck()
    {
        // Only check "Ground" (layer 3) and "Platform" (layer 7)
        int groundMask = (1 << 3) | (1 << 7);
        // Equivalent to LayerMask.GetMask("Ground", "Platform")

        RaycastHit2D hit = Physics2D.Raycast(
            transform.position, Vector2.down, 1.1f, groundMask
        );
        isGrounded = hit.collider != null;
    }

    // --- Simple Graph for Pathfinding ---
    Dictionary<string, List<string>> roomGraph = new Dictionary<string, List<string>>
    {
        { "Entrance",  new List<string> { "Hallway", "Armory" } },
        { "Hallway",   new List<string> { "Entrance", "Throne Room", "Dungeon" } },
        { "Armory",    new List<string> { "Entrance" } },
        { "Throne Room", new List<string> { "Hallway" } },
        { "Dungeon",   new List<string> { "Hallway", "Secret Exit" } },
        { "Secret Exit", new List<string> { "Dungeon" } }
    };

    List<string> GetNeighbors(string room) => roomGraph.ContainsKey(room)
        ? roomGraph[room] : new List<string>();

    void Start()
    {
        // Bitmask demo
        ApplyEffect(StatusEffect.Poisoned | StatusEffect.Burning);
        Debug.Log($"Poisoned? {HasEffect(StatusEffect.Poisoned)}"); // True
        Debug.Log($"Frozen? {HasEffect(StatusEffect.Frozen)}");     // False
        RemoveEffect(StatusEffect.Poisoned);
        Debug.Log($"Still poisoned? {HasEffect(StatusEffect.Poisoned)}"); // False

        // Graph demo
        Debug.Log($"Rooms connected to Hallway: {string.Join(", ", GetNeighbors("Hallway"))}");
        // Output: Entrance, Throne Room, Dungeon
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the XOR operator return? :: True when exactly one of the two inputs is true, false when both are the same (both true or both false). $A \oplus B$.
- How do you set bit $n$ in a bitmask without affecting other bits? :: `flags |= (1 << n)` — the OR operation turns on bit $n$ while preserving all other bits.
- What is De Morgan's Law in plain English? :: "NOT (A AND B)" is the same as "(NOT A) OR (NOT B)," and "NOT (A OR B)" is the same as "(NOT A) AND (NOT B)." Negation distributes by swapping AND↔OR.
- How do layer masks determine collision in Unity? :: Two objects can collide if the bitwise AND of their layer masks is non-zero: `(maskA & maskB) != 0`. Each bit represents a layer, and the AND checks for overlapping enabled layers.
- What is a graph in mathematical terms? :: A set of vertices (nodes) $V$ and edges (connections) $E$: $G = (V, E)$. Vertices represent locations or states, edges represent connections or transitions.
- Why are bitmask flags faster than arrays of booleans? :: A single 32-bit integer holds 32 flags, and checking/setting bits uses single CPU instructions (AND, OR, XOR). An array requires memory allocation, indexing, and multiple comparisons.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Confusing `&` (bitwise AND) with `&&` (logical AND). Writing `if (flags & mask)` in C# causes a compiler error because the result is an `int`, not a `bool`.
> - **The Fix:** Always compare the result to zero: `if ((flags & mask) != 0)`.
> - **Why:** In C/C++, non-zero integers are truthy, but C# requires explicit boolean comparisons. This is a cross-language trap.

> [!danger] **Watch Out!**
> - **The Mistake:** Using `flags = flag` instead of `flags |= flag` to add a status effect, which wipes out all other active effects.
> - **The Fix:** Use `|=` to set, `&= ~flag` to clear, and `^=` to toggle. Never use plain assignment unless you intend to replace all flags.
> - **Why:** `flags = flag` overwrites the entire bitmask. `flags |= flag` merges the new bit in while keeping existing bits intact.

---

## Related Topics
- [[Math/12_Advanced_Topics/pathfinding_graph_theory|Pathfinding & Graph Theory]]
- [[Math/10_Physics_Math/collision_detection|Collision Detection]]
- [[Math/01_Foundations/number_systems|Number Systems & Representation]]
- [[Math/01_Foundations/algebra_fundamentals|Algebra Fundamentals]]
