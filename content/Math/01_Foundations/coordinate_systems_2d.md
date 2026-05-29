---
title: "2D Coordinate Systems: Mapping Every Pixel to a Purpose"
tags:
  - math
  - level/Lv1
  - category/foundations
---

# 2D Coordinate Systems: Mapping Every Pixel to a Purpose

> [!abstract] **The Concept in a Nutshell**
> A 2D coordinate system is a framework for describing the position of every point on a flat surface using two numbers: $(x, y)$. In game development, you constantly juggle *multiple* 2D coordinate systems — world space, screen space, UV space — and understanding how they relate is essential for placing objects, handling input, and rendering correctly.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: The Platformer Coordinate Confusion**
> You're building "Pixel Knight," a 2D platformer. Your character stands at world position $(3.5, 2.0)$. The screen is 1920×1080 pixels, with the camera centered on the player.
>
> A mouse click arrives at screen coordinates $(960, 540)$ — the center of the screen. But screen Y increases *downward* (pixel convention), while your world Y increases *upward* (math convention). If you naively convert, the player fires a grappling hook toward the ground instead of straight ahead!
>
> Meanwhile, your tilemap uses integer coordinates where tile $(5, 3)$ means column 5, row 3. Each tile is 32×32 pixels. Translating between tile coordinates, world coordinates, and screen coordinates is a three-way conversion you must get right, or everything misaligns.

---

## The Blueprint (Formula & Structure)

### The Cartesian Plane
Named after René Descartes, the Cartesian plane defines position using two perpendicular axes:

- **X-axis:** horizontal (positive = right)
- **Y-axis:** vertical (positive = up in math, **down in screen space**)
- **Origin:** $(0, 0)$ — where the axes intersect

### Quadrants

| Quadrant | X Sign | Y Sign | Example |
|----------|--------|--------|---------|
| I | $+$ | $+$ | $(3, 4)$ — upper-right |
| II | $-$ | $+$ | $(-2, 5)$ — upper-left |
| III | $-$ | $-$ | $(-1, -3)$ — lower-left |
| IV | $+$ | $-$ | $(4, -2)$ — lower-right |

### Screen Coordinates vs. World Coordinates

**Screen (Pixel) Space:**
- Origin at **top-left** corner of the window
- X increases rightward: $0$ to $\text{screenWidth} - 1$
- Y increases **downward**: $0$ to $\text{screenHeight} - 1$
- Units: pixels

**World Space (2D Game):**
- Origin defined by the designer (often center of starting area)
- X increases rightward
- Y increases **upward** (standard math convention)
- Units: game units (meters, tiles, etc.)

### Conversion Formulas

**World → Screen:**
$$x_{\text{screen}} = (x_{\text{world}} - x_{\text{cam}}) \times \text{pixelsPerUnit} + \frac{\text{screenWidth}}{2}$$
$$y_{\text{screen}} = \frac{\text{screenHeight}}{2} - (y_{\text{world}} - y_{\text{cam}}) \times \text{pixelsPerUnit}$$

Note the **negation** on the Y-axis — this is the critical flip between math-Y-up and screen-Y-down.

**Screen → World:**
$$x_{\text{world}} = \frac{x_{\text{screen}} - \frac{\text{screenWidth}}{2}}{\text{pixelsPerUnit}} + x_{\text{cam}}$$
$$y_{\text{world}} = \frac{\frac{\text{screenHeight}}{2} - y_{\text{screen}}}{\text{pixelsPerUnit}} + y_{\text{cam}}$$

### Tile Coordinates
For a grid-based game with tile size $T$:

$$\text{tileX} = \lfloor x_{\text{world}} / T \rfloor, \quad \text{tileY} = \lfloor y_{\text{world}} / T \rfloor$$

$$x_{\text{world}} = \text{tileX} \times T + \frac{T}{2} \quad \text{(tile center)}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Map and the Window**
> Imagine a huge paper map spread across a table (world space). You place a small rectangular frame (the camera/screen) on top of the map. What you see through the frame is screen space.
>
> - Moving the frame left/right/up/down across the map = camera panning
> - Zooming in = making `pixelsPerUnit` larger (each game unit covers more pixels)
> - The map uses "north = up" (Y-up), but the frame's label sticker numbering starts from the top-left corner and counts downward (Y-down) — that's the screen convention
>
> **The key insight:** world space and screen space describe the *same* points, just measured from different origins with a flipped Y-axis. Unity's `Camera.WorldToScreenPoint()` performs this conversion for you.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Working with 2D coordinate systems
using UnityEngine;

public class CoordinateSystems2D : MonoBehaviour
{
    [Header("Grid Settings")]
    public int tileSize = 1; // 1 world unit per tile

    void Update()
    {
        // --- Screen to World conversion (Unity handles the Y-flip internally) ---
        Vector3 mouseScreen = Input.mousePosition;
        Vector3 mouseWorld = Camera.main.ScreenToWorldPoint(
            new Vector3(mouseScreen.x, mouseScreen.y, -Camera.main.transform.position.z)
        );
        // mouseWorld.y is in world space (Y-up) — correct for gameplay

        // --- World to Tile conversion ---
        int tileX = Mathf.FloorToInt(mouseWorld.x / tileSize);
        int tileY = Mathf.FloorToInt(mouseWorld.y / tileSize);
        // Now (tileX, tileY) identifies the grid cell the mouse hovers over

        // --- Tile center back to World ---
        Vector2 tileCenter = new Vector2(
            tileX * tileSize + tileSize * 0.5f,
            tileY * tileSize + tileSize * 0.5f
        );

        // --- Manual Screen to World (without Camera helper) ---
        // Useful for understanding what Unity does internally
        float pixelsPerUnit = Screen.height / (Camera.main.orthographicSize * 2f);
        Vector2 camPos = Camera.main.transform.position;
        float worldX = (mouseScreen.x - Screen.width / 2f) / pixelsPerUnit + camPos.x;
        float worldY = (mouseScreen.y - Screen.height / 2f) / pixelsPerUnit + camPos.y;
        // Note: Unity's screen space already has Y-up (0 at bottom), unlike raw OS events

        if (Input.GetMouseButtonDown(0))
        {
            Debug.Log($"Screen: ({mouseScreen.x:F0}, {mouseScreen.y:F0})");
            Debug.Log($"World:  ({mouseWorld.x:F2}, {mouseWorld.y:F2})");
            Debug.Log($"Tile:   ({tileX}, {tileY})");
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- In standard screen coordinates, which direction does Y increase? :: **Downward.** The origin is at the top-left, and Y increases toward the bottom of the screen. This is the opposite of mathematical convention.
- Why do you negate Y when converting from world to screen coordinates? :: Because world space uses Y-up (positive Y = higher) while screen space uses Y-down (positive Y = lower). The negation flips the axis.
- What are the four quadrants of the Cartesian plane? :: Quadrant I (+, +), Quadrant II (−, +), Quadrant III (−, −), Quadrant IV (+, −), numbered counter-clockwise starting from the upper-right.
- How do you convert a world position to a tile index in a grid-based game? :: $\text{tileX} = \lfloor x_{\text{world}} / \text{tileSize} \rfloor$, and similarly for Y. Use `Mathf.FloorToInt()` to handle negative coordinates correctly.
- In Unity 2D, does `Input.mousePosition` use Y-up or Y-down? :: Y-up — Unity's screen space convention has $(0, 0)$ at the **bottom-left** of the screen, unlike most OS-level APIs which use top-left.
- What does `Camera.main.ScreenToWorldPoint()` actually do? :: It converts a pixel position on screen into a position in world space, accounting for camera position, rotation, zoom (orthographic size or FOV), and the Y-axis convention.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting the Y-axis flip when doing manual coordinate conversions. You click the top of the screen expecting a high Y-world value, but get a low one.
> - **The Fix:** Always check which convention each system uses. Unity screen space is Y-up (unusual!), but HTML Canvas, SDL, and most raw OS APIs are Y-down. When in doubt, `Debug.Log` both values.
> - **Why:** Different frameworks made different convention choices decades ago, and we're stuck with the inconsistency forever.

> [!danger] **Watch Out!**
> - **The Mistake:** Using `(int)` cast instead of `Mathf.FloorToInt()` for tile conversion with negative coordinates. `(int)(-0.5f)` gives `0`, not `-1`.
> - **The Fix:** Always use `Mathf.FloorToInt()` which correctly rounds toward negative infinity.
> - **Why:** C# integer casting truncates toward zero, which gives wrong tile indices for negative coordinates.

---

## Related Topics
- [[Math/01_Foundations/coordinate_systems_3d|3D Coordinate Systems]]
- [[Math/03_Vectors/vector_fundamentals|Vector Fundamentals]]
- [[Math/01_Foundations/number_systems|Number Systems & Representation]]
- [[Math/02_Geometry_Trigonometry/polar_coordinates|Polar Coordinate Systems]]
