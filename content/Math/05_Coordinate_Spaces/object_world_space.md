---
title: "Object & World Space: Where Things Actually Are"
tags:
  - math
  - level/Lv2
  - category/coordinate_spaces
---

# Object & World Space: Where Things Actually Are

> [!abstract] **The Concept in a Nutshell**
> Every object in a game lives in its own **local (object) space** — a private coordinate system centered on itself. The **world space** is the shared, global coordinate system where everything comes together. The **model matrix** transforms vertices from local space to world space by applying scale, rotation, and translation in sequence. Parent-child hierarchies chain these transforms so moving a parent automatically moves all its children.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Knight's Sword in an RPG**
> Sir Aldric draws his longsword. The sword mesh was modeled with its grip at the origin and blade along the local Y-axis. In the game world, the sword is parented to the knight's `RightHand` bone.
>
> - **Sword local space:** grip at $(0, 0, 0)$, tip at $(0, 1.2, 0)$.
> - **Hand local space:** hand is offset $(0.3, 0, 0.1)$ from the forearm joint.
> - **Character world space:** Sir Aldric stands at $(15, 0, -8)$, rotated $45°$ to face an enemy.
>
> When you call `sword.transform.position` (world), Unity chains: **Sword Local → Hand Local → Forearm Local → ... → Character Local → World**. That tip at local $(0, 1.2, 0)$ ends up somewhere around $(16.1, 1.8, -7.3)$ in the world — and it updates every frame as the animation plays. Without this hierarchy, you'd need to manually recompute every attachment point every frame.

---

## The Blueprint (Formula & Structure)

### The Model Matrix (Local → World)

A point $\mathbf{p}_\text{local}$ in object space is transformed to world space by the **model matrix** $\mathbf{M}$:

$$\mathbf{p}_\text{world} = \mathbf{M} \cdot \mathbf{p}_\text{local}$$

The model matrix is composed of Scale ($\mathbf{S}$), Rotation ($\mathbf{R}$), and Translation ($\mathbf{T}$):

$$\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$$

In expanded 4×4 homogeneous form:

$$\mathbf{M} = \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} r_{00} & r_{01} & r_{02} & 0 \\ r_{10} & r_{11} & r_{12} & 0 \\ r_{20} & r_{21} & r_{22} & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} s_x & 0 & 0 & 0 \\ 0 & s_y & 0 & 0 \\ 0 & 0 & s_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

### Nested Transforms (Parent-Child Chain)

For a hierarchy **Grandparent → Parent → Child**, the world transform of the child is:

$$\mathbf{M}_\text{child→world} = \mathbf{M}_\text{grandparent} \cdot \mathbf{M}_\text{parent} \cdot \mathbf{M}_\text{child}$$

Each matrix converts from one local space to the next parent's local space. The full chain converts from the deepest child all the way to world space.

### Key Distinction: `localPosition` vs `position`

| Property | Space | Meaning |
|---|---|---|
| `Transform.localPosition` | Parent's local space | Offset from the parent's origin |
| `Transform.position` | World space | Absolute position in the scene |
| `Transform.localRotation` | Parent's local space | Rotation relative to parent |
| `Transform.rotation` | World space | Absolute rotation in the scene |

### Inverse Transform (World → Local)

To convert a world-space point *into* an object's local space:

$$\mathbf{p}_\text{local} = \mathbf{M}^{-1} \cdot \mathbf{p}_\text{world}$$

Unity provides `Transform.InverseTransformPoint()` for exactly this.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Russian Nesting Dolls (Matryoshka)**
> Think of coordinate spaces as nested Russian dolls. The smallest doll (the sword) lives in its own tiny world. It sits inside the hand doll, which sits inside the arm doll, which sits inside the character doll, which sits in the room (world).
>
> Moving the biggest doll (the character) moves everything inside it. Moving the hand doll only moves the hand and whatever's inside (the sword). The sword doesn't care about the world — it only knows its offset from the hand. Each doll only stores its relationship to the *next doll out*.
>
> **Local space = "how far from my parent?"**
> **World space = "where in the universe?"**
>
> The model matrix is the instruction manual for unpacking all the dolls to find the absolute world position.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Converting between local and world space
// Demonstrates parent-child hierarchy and space conversions

using UnityEngine;

public class SpaceConversion : MonoBehaviour
{
    public Transform sword;      // Child of hand
    public Transform hand;       // Child of character
    public Transform character;  // Root object

    void Update()
    {
        // The sword tip in LOCAL space (relative to sword's own origin)
        Vector3 tipLocal = new Vector3(0f, 1.2f, 0f);

        // Convert sword-local point to WORLD space
        // Unity automatically walks the entire parent chain
        Vector3 tipWorld = sword.TransformPoint(tipLocal);
        Debug.Log($"Sword tip world pos: {tipWorld}");

        // Convert a world point INTO the character's local space
        // Useful for checking "is the enemy in front of me?"
        Vector3 enemyWorld = new Vector3(20f, 0f, -5f);
        Vector3 enemyLocal = character.InverseTransformPoint(enemyWorld);
        Debug.Log($"Enemy in character local space: {enemyLocal}");

        // If enemyLocal.z > 0 → enemy is in front of character
        // If enemyLocal.x > 0 → enemy is to the right
        bool enemyInFront = enemyLocal.z > 0f;

        // Setting local vs world position
        // This places the sword 0.1m above the hand's origin
        sword.localPosition = new Vector3(0f, 0.1f, 0f);

        // This would teleport the sword to an absolute world position
        // (breaking the visual attachment to the hand)
        // sword.position = new Vector3(5f, 3f, 0f); // Usually NOT what you want

        // Manual matrix chain (educational — Unity does this internally)
        Matrix4x4 charMatrix = character.localToWorldMatrix;
        Matrix4x4 handLocal  = Matrix4x4.TRS(hand.localPosition,
                                              hand.localRotation,
                                              hand.localScale);
        Matrix4x4 swordLocal = Matrix4x4.TRS(sword.localPosition,
                                              sword.localRotation,
                                              sword.localScale);

        // Full chain: sword local → world
        Matrix4x4 fullChain = charMatrix * handLocal * swordLocal;
        Vector3 tipManual = fullChain.MultiplyPoint3x4(tipLocal);
        Debug.Log($"Manual chain result: {tipManual}");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does the model matrix do? :: Transforms vertices from **object/local space** to **world space** by combining Scale, Rotation, and Translation ($\mathbf{M} = \mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$).
- What's the difference between `Transform.position` and `Transform.localPosition`? :: `position` gives the object's coordinates in **world space** (absolute). `localPosition` gives the offset from the **parent's** origin in the parent's coordinate system.
- In a chain Grandparent → Parent → Child, how do you get the child's world matrix? :: $\mathbf{M}_\text{world} = \mathbf{M}_\text{grandparent} \cdot \mathbf{M}_\text{parent} \cdot \mathbf{M}_\text{child}$. Multiply from the outermost ancestor to the deepest child.
- How do you convert a world-space point to an object's local space in Unity? :: Use `transform.InverseTransformPoint(worldPoint)`, which applies $\mathbf{M}^{-1}$ to the world-space point.
- Why is the model matrix order $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ and not $\mathbf{S} \cdot \mathbf{R} \cdot \mathbf{T}$? :: Because matrices apply right-to-left to the point: first scale the object, then rotate it (around the origin), then translate it to its world position. Reversing the order would rotate/scale around the wrong origin.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Setting `transform.position` on a child object to "place" it relative to its parent, then being confused when it drifts away during parent movement.
> - **The Fix:** Use `transform.localPosition` when you want to offset a child from its parent. Use `transform.position` only when you need an absolute world-space placement (which effectively un-parents the object visually).
> - **Why:** `position` is in world space. If the parent moves, the child's `localPosition` stays the same (so it follows the parent), but the same world `position` is now a *different* local offset — the child appears to slide.

> [!danger] **Watch Out!**
> - **The Mistake:** Applying non-uniform scale to a parent and then wondering why child rotations look skewed or sheared.
> - **The Fix:** Keep parent scales uniform $(s, s, s)$ whenever possible, or apply non-uniform scale only on leaf objects with no rotated children.
> - **Why:** Non-uniform scale followed by rotation introduces **shear**, which deforms the child's local space. This is a fundamental limitation of composing $\mathbf{T} \cdot \mathbf{R} \cdot \mathbf{S}$ chains.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/view_projection_space|View & Projection Space]]
- [[Math/04_Matrices_Transforms/affine_transformations|Affine Transformations]]
- [[Math/02_Geometry_Trigonometry/coordinate_systems_3d|3D Coordinate Systems]]
