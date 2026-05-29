---
title: "3D Coordinate Systems: Navigating the Handedness Wars"
tags:
  - math
  - level/Lv1
  - category/foundations
---

# 3D Coordinate Systems: Navigating the Handedness Wars

> [!abstract] **The Concept in a Nutshell**
> A 3D coordinate system extends the 2D plane by adding a third axis — depth. Three numbers $(x, y, z)$ locate any point in space. The catch: there's no universal agreement on which direction each axis points, leading to the **handedness wars** between engines. Unity uses left-handed Y-up, Unreal uses left-handed Z-up, and Blender uses right-handed Z-up. Importing models between them without understanding these conventions produces mirrored, rotated, or sideways assets.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The Upside-Down Dragon**
> Your artist models a dragon in Blender (right-handed, Z-up). They export it as FBX and import it into Unity (left-handed, Y-up). The dragon appears on its side — what was "up" (Z in Blender) is now pointing along Unity's Z-axis (forward). Its mouth faces the ground. Its wings stick out horizontally instead of vertically.
>
> You rotate it $-90°$ around X to fix it, but now the animations play incorrectly because the bone hierarchy's local axes are scrambled. The fire breath particle emitter shoots sideways.
>
> The fix? Understanding the conversion: Blender's $(x, y, z)$ maps to Unity's $(x, z, y)$ with a sign flip to handle the handedness change. FBX importers usually do this automatically, but only if configured correctly.

---

## The Blueprint (Formula & Structure)

### The Three Axes
A 3D coordinate system has three mutually perpendicular axes:
- **X-axis:** typically left/right
- **Y-axis:** up/down or forward/back (depends on convention!)
- **Z-axis:** the remaining direction

### Handedness: Left vs. Right

**Right-Hand Rule:**
Point your right hand's fingers along +X, curl them toward +Y. Your thumb points toward +Z.

**Left-Hand Rule:**
Same gesture, but with your left hand. Your thumb points in the *opposite* Z direction.

$$\text{Right-handed: } \hat{x} \times \hat{y} = +\hat{z}$$
$$\text{Left-handed: } \hat{x} \times \hat{y} = -\hat{z}$$

The key difference: **the cross product yields opposite Z directions.** This affects winding order, normal calculations, and rotation direction.

### Engine Convention Comparison

| Engine | Handedness | Up Axis | Forward Axis | Right Axis |
|--------|------------|---------|--------------|------------|
| **Unity** | Left-handed | +Y | +Z | +X |
| **Unreal Engine** | Left-handed | +Z | +X | +Y |
| **Blender** | Right-handed | +Z | -Y | +X |
| **OpenGL** | Right-handed | +Y | -Z | +X |
| **DirectX** | Left-handed | +Y | +Z | +X |
| **Godot** | Right-handed | +Y | -Z | +X |

### Conversion Between Conventions

**Blender (RH, Z-up) → Unity (LH, Y-up):**
$$x_{\text{Unity}} = x_{\text{Blender}}$$
$$y_{\text{Unity}} = z_{\text{Blender}}$$
$$z_{\text{Unity}} = y_{\text{Blender}}$$

Then negate the Z axis to switch handedness:
$$z_{\text{Unity}} = -y_{\text{Blender}} \quad \text{(after swapping)}$$

Or equivalently, negate one axis of your choice — swapping two axes and negating one converts between handedness.

**General rule:** To convert between left-handed and right-handed systems:
1. Swap or remap axes to match the "up" and "forward" conventions
2. Negate exactly one axis to flip handedness

### Why Does Handedness Matter?
- **Triangle winding order:** A triangle with vertices in counter-clockwise order faces the camera in right-handed systems, but faces *away* in left-handed systems. Wrong handedness = invisible (backface-culled) meshes.
- **Cross products:** The direction of computed normals flips.
- **Rotation direction:** "Positive rotation around Y" means counter-clockwise from above in right-handed, clockwise in left-handed.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Room Analogy**
> Stand in the corner of a room facing a wall.
>
> **Right-handed (OpenGL/Blender):** X goes right along the wall. Y goes up to the ceiling. Z comes *toward you* (out of the wall). Think: "I'm looking at the wall, and Z pushes me backward." Like reading a book — the page is the XY plane, and Z comes out at your face.
>
> **Left-handed (Unity/DirectX):** X goes right. Y goes up. Z goes *into the wall* (away from you). Think: "Z is depth — things farther away have higher Z." Like a first-person game — forward is +Z.
>
> **Z-up (Unreal/Blender):** Instead of Y pointing up, Z points to the ceiling. Y points forward (or backward). Think: "Looking at a blueprint/floorplan on a table — the flat plan is the XY plane, and Z is the height off the table."
>
> When you export a model between systems, it's like picking up a figure from one room and placing it in a room with different "up" and "forward" labels.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Coordinate system conversions and conventions
using UnityEngine;

public class CoordinateConversions : MonoBehaviour
{
    // --- Blender (Right-handed, Z-up) to Unity (Left-handed, Y-up) ---
    // Blender: X=right, Y=forward, Z=up (right-handed)
    // Unity:   X=right, Y=up,      Z=forward (left-handed)
    public static Vector3 BlenderToUnity(Vector3 blender)
    {
        // Swap Y and Z, negate new Z (to flip handedness)
        return new Vector3(blender.x, blender.z, -blender.y);
    }

    // --- Unreal (Left-handed, Z-up) to Unity (Left-handed, Y-up) ---
    // Unreal: X=forward, Y=right, Z=up
    // Unity:  X=right,   Y=up,    Z=forward
    public static Vector3 UnrealToUnity(Vector3 unreal)
    {
        return new Vector3(unreal.y, unreal.z, unreal.x);
        // Same handedness, just axis remapping
    }

    // --- Demonstrate Unity's coordinate system ---
    void Start()
    {
        // Unity's basis vectors
        Debug.Log($"Right:   {Vector3.right}");    // (1, 0, 0)  = +X
        Debug.Log($"Up:      {Vector3.up}");       // (0, 1, 0)  = +Y
        Debug.Log($"Forward: {Vector3.forward}");  // (0, 0, 1)  = +Z

        // Left-handed cross product: right × up = forward (in LH)
        Vector3 cross = Vector3.Cross(Vector3.right, Vector3.up);
        Debug.Log($"right × up = {cross}"); // (0, 0, 1) — confirms left-handed

        // In right-handed systems, right × up = -forward
        // This is why normals flip when importing from Blender!

        // --- Example conversion ---
        Vector3 blenderPos = new Vector3(2f, 5f, 3f); // X=2, Y=5(fwd), Z=3(up)
        Vector3 unityPos = BlenderToUnity(blenderPos);
        Debug.Log($"Blender (2,5,3) → Unity {unityPos}"); // (2, 3, -5)
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the right-hand rule for coordinate systems? :: Point your right hand's fingers along +X, curl them toward +Y, and your thumb indicates +Z. If a system follows this, it's right-handed.
- Which coordinate system does Unity use? :: Left-handed, Y-up. +X is right, +Y is up, +Z is forward.
- Why do models from Blender sometimes appear sideways in Unity? :: Blender uses Z-up while Unity uses Y-up. Without axis remapping, the model's "up" (Z) aligns with Unity's "forward" (Z), laying the model on its side.
- How do you convert between left-handed and right-handed systems? :: Remap the axes to match the up/forward conventions, then negate exactly one axis to flip the handedness.
- What happens to triangle winding order when handedness changes? :: Counter-clockwise winding (front-facing in right-handed) becomes clockwise (back-facing in left-handed), causing triangles to face away from the camera and become invisible due to backface culling.
- Does Unreal Engine use Y-up or Z-up? :: Z-up. Unreal uses a left-handed coordinate system with +Z as up, +X as forward, and +Y as right.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Manually rotating an imported model by 90° to "fix" the orientation instead of correcting the axis mapping.
> - **The Fix:** Fix the coordinate conversion at the import step (FBX importer settings in Unity, or use the correct export preset in Blender). A rotation hack breaks animations, physics colliders, and child transforms.
> - **Why:** A rotation changes the object's *transform* but doesn't remap its internal axis data. Bone local axes, animation curves, and particle directions will all be wrong.

> [!danger] **Watch Out!**
> - **The Mistake:** Assuming all left-handed systems are identical. Unity (LH, Y-up) and Unreal (LH, Z-up) are both left-handed but have different axis assignments.
> - **The Fix:** Always check both handedness AND axis-to-direction mapping for each engine.
> - **Why:** Handedness alone only tells you one thing (cross product direction). The mapping of axes to spatial directions is independent.

---

## Related Topics
- [[Math/01_Foundations/coordinate_systems_2d|2D Coordinate Systems]]
- [[Math/05_Coordinate_Spaces/object_world_space|Object & World Space]]
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/06_Rotations_Orientation/rotation_matrices_2d|2D Rotation Matrices]]
