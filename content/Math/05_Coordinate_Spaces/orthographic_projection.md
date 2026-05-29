---
title: "Orthographic Projection: Flat Is a Feature, Not a Bug"
tags:
  - math
  - level/Lv3
  - category/coordinate_spaces
---

# Orthographic Projection: Flat Is a Feature, Not a Bug

> [!abstract] **The Concept in a Nutshell**
> An orthographic projection maps 3D geometry to 2D without any perspective foreshortening — objects look the **same size** regardless of distance from the camera. It defines a **rectangular box** (not a frustum) as the visible volume. This is essential for 2D games, isometric views, UI elements, shadow map rendering, and any situation where you need parallel projection lines.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: An Isometric ARPG (Diablo-Style)**
> You're building an isometric action RPG. The camera hangs above the world at a $45°$ angle, looking down at the hero, monsters, and loot. A chest 50 meters away should appear the **exact same size** as a chest 5 meters away — there's no depth-based shrinking. Characters at the top of the screen aren't "smaller" because they're farther from the camera; they're just higher up in the world.
>
> Separately, your UI system renders health bars, inventory slots, and text. These are all flat 2D elements that must stay pixel-perfect regardless of the 3D camera. An orthographic camera with size matched to screen resolution handles this perfectly.
>
> And behind the scenes, your directional light renders a **shadow map** using orthographic projection — because sunlight rays are parallel, an ortho camera perfectly models the sun's "view" of the scene.

---

## The Blueprint (Formula & Structure)

### The Orthographic Projection Matrix

Given a viewing box defined by:
- $l, r$ = left and right boundaries
- $b, t$ = bottom and top boundaries
- $n, f$ = near and far clipping planes

**OpenGL convention** (NDC $z \in [-1, 1]$):

$$\mathbf{P}_\text{ortho} = \begin{bmatrix} \frac{2}{r - l} & 0 & 0 & -\frac{r + l}{r - l} \\ 0 & \frac{2}{t - b} & 0 & -\frac{t + b}{t - b} \\ 0 & 0 & -\frac{2}{f - n} & -\frac{f + n}{f - n} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

### Symmetric Orthographic (Centered)

When the view box is centered ($l = -r$, $b = -t$), the matrix simplifies:

$$\mathbf{P}_\text{ortho} = \begin{bmatrix} \frac{1}{r} & 0 & 0 & 0 \\ 0 & \frac{1}{t} & 0 & 0 \\ 0 & 0 & -\frac{2}{f - n} & -\frac{f + n}{f - n} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

### Key Difference from Perspective

| Property | Perspective | Orthographic |
|---|---|---|
| Bottom row of matrix | $[0, 0, -1, 0]$ | $[0, 0, 0, 1]$ |
| Output $w$ component | $w = -z_\text{view}$ (depth-dependent) | $w = 1$ (constant) |
| Perspective divide effect | Divides by depth → foreshortening | Divides by 1 → no effect |
| Visible volume shape | Truncated pyramid (frustum) | Rectangular box |
| Parallel lines | Converge at vanishing point | Remain parallel |

### Unity's Orthographic Size

In Unity, `Camera.orthographicSize` defines **half the vertical height** of the visible area in world units:

$$\text{height}_\text{visible} = 2 \times \text{orthographicSize}$$
$$\text{width}_\text{visible} = 2 \times \text{orthographicSize} \times \text{aspect}$$

For a pixel-perfect 2D game at 1920×1080:

$$\text{orthographicSize} = \frac{1080}{2 \times \text{PPU}}$$

Where PPU = pixels per unit (typically 100 in Unity's default 2D settings), giving $\text{orthographicSize} = 5.4$.

### When Ortho Beats Perspective

1. **2D games** — No depth distortion, pixel-perfect alignment.
2. **Isometric/top-down** — Consistent scale across the map.
3. **UI rendering** — Overlay elements must be resolution-dependent, not depth-dependent.
4. **Shadow maps** — Directional light has parallel rays, perfectly modeled by ortho.
5. **Technical/CAD views** — Architects and engineers need true-scale drawings.
6. **Minimap rendering** — A secondary ortho camera looking straight down.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Stamp Press vs a Flashlight**
> **Perspective** is like a flashlight: rays diverge from a single point (the camera). Objects farther from the source cast smaller shadows.
>
> **Orthographic** is like a stamp press: the "light" comes from an infinitely large, flat panel. All rays are **perfectly parallel**. A coin held 1 meter away casts the same shadow as a coin held 10 meters away. No convergence, no vanishing points, no foreshortening.
>
> The matrix difference is elegant: perspective puts $-1$ in the bottom row, shoving depth into $w$ for the divide. Orthographic puts $1$ in $w$, so the divide does nothing — every object maintains its true proportional size.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Setting up orthographic cameras for different use cases
using UnityEngine;

public class OrthoSetup : MonoBehaviour
{
    [Header("Main Game Camera (Isometric)")]
    public Camera gameCamera;
    public float isoSize = 10f; // Half-height in world units

    [Header("UI Camera")]
    public Camera uiCamera;

    [Header("Minimap Camera")]
    public Camera minimapCamera;
    public float minimapWorldSize = 100f;

    [Header("Shadow Map Simulation")]
    public Light directionalLight;

    void Start()
    {
        // === Isometric Game Camera ===
        SetupIsometricCamera();

        // === Pixel-Perfect UI Camera ===
        SetupUICamera();

        // === Minimap Camera ===
        SetupMinimapCamera();
    }

    void SetupIsometricCamera()
    {
        gameCamera.orthographic = true;
        gameCamera.orthographicSize = isoSize;
        gameCamera.nearClipPlane = 0.1f;
        gameCamera.farClipPlane = 100f;

        // Classic isometric angle: 30° around X, 45° around Y
        gameCamera.transform.rotation = Quaternion.Euler(30f, 45f, 0f);
        gameCamera.transform.position = new Vector3(0f, 20f, 0f)
            - gameCamera.transform.forward * 30f;
    }

    void SetupUICamera()
    {
        uiCamera.orthographic = true;
        // For pixel-perfect at 1080p with PPU = 100
        uiCamera.orthographicSize = Screen.height / (2f * 100f);
        uiCamera.nearClipPlane = -10f;
        uiCamera.farClipPlane = 10f;
        // Clear only depth so the game camera's image shows through
        uiCamera.clearFlags = CameraClearFlags.Depth;
    }

    void SetupMinimapCamera()
    {
        minimapCamera.orthographic = true;
        minimapCamera.orthographicSize = minimapWorldSize * 0.5f;
        minimapCamera.transform.position = new Vector3(0f, 50f, 0f);
        minimapCamera.transform.rotation = Quaternion.Euler(90f, 0f, 0f);

        // Render to a corner of the screen
        minimapCamera.rect = new Rect(0.75f, 0.75f, 0.24f, 0.24f);
    }

    // Build orthographic matrix manually (educational)
    static Matrix4x4 BuildOrthoMatrix(float left, float right,
                                       float bottom, float top,
                                       float near, float far)
    {
        Matrix4x4 m = Matrix4x4.identity;
        m[0, 0] = 2f / (right - left);
        m[1, 1] = 2f / (top - bottom);
        m[2, 2] = -2f / (far - near);
        m[0, 3] = -(right + left) / (right - left);
        m[1, 3] = -(top + bottom) / (top - bottom);
        m[2, 3] = -(far + near) / (far - near);
        return m;
    }

    void Update()
    {
        // Zoom in/out with scroll wheel
        float scroll = Input.GetAxis("Mouse ScrollWheel");
        if (Mathf.Abs(scroll) > 0.01f)
        {
            gameCamera.orthographicSize -= scroll * 3f;
            gameCamera.orthographicSize = Mathf.Clamp(
                gameCamera.orthographicSize, 2f, 30f);
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the fundamental difference between orthographic and perspective projection? :: Orthographic uses **parallel** projection lines — objects don't shrink with distance. Perspective uses **converging** projection lines from a single point, creating foreshortening. Mathematically, ortho keeps $w = 1$; perspective sets $w = -z$.
- What does Unity's `Camera.orthographicSize` represent? :: It is **half the vertical height** of the visible area in world units. The total visible height = $2 \times \text{orthographicSize}$, and the visible width = $2 \times \text{orthographicSize} \times \text{aspect ratio}$.
- Name three game dev use cases where orthographic projection is preferred. :: 1) **2D / isometric games** (no depth distortion), 2) **UI rendering** (pixel-perfect overlays), 3) **Shadow maps** for directional lights (parallel sun rays). Also: minimaps, CAD/technical views.
- Why are shadow maps for directional lights rendered with orthographic projection? :: Directional lights model **infinitely distant** light sources (like the sun), meaning all light rays are **parallel**. Orthographic projection perfectly captures parallel rays, while perspective would incorrectly model them as diverging from a point.
- How do you calculate `orthographicSize` for pixel-perfect 2D rendering? :: $\text{orthographicSize} = \frac{\text{screenHeight}}{2 \times \text{PPU}}$, where PPU is pixels per world unit. For 1080p with PPU=100: $\text{size} = 1080 / 200 = 5.4$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using perspective projection for a 2D game and then wondering why sprites at different Z depths appear different sizes or have subpixel jitter.
> - **The Fix:** Switch to `Camera.orthographic = true` and set `orthographicSize` appropriately for your target resolution and PPU.
> - **Why:** Perspective projection divides by depth. Even small Z differences cause visible size changes. Orthographic guarantees consistent sizing at any depth.

> [!danger] **Watch Out!**
> - **The Mistake:** Setting the orthographic camera's near/far planes too tightly around $z = 0$ and then placing objects with varying Z depths (for sorting) that fall outside the clipping range.
> - **The Fix:** Ensure your near/far range encompasses all Z layers. In 2D, a range like $[-10, 10]$ or $[0.1, 100]$ usually covers typical sorting layers.
> - **Why:** Unlike perspective, ortho near/far don't affect visual quality (no precision issues). So make them generous enough to include all your objects.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/perspective_projection|Perspective Projection]]
- [[Math/05_Coordinate_Spaces/view_frustum|View Frustum]]
- [[Math/10_Physics_Math/shadow_math|Shadow Math]]
