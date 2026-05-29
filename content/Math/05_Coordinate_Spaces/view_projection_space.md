---
title: "View & Projection Space: From 3D World to 2D Screen"
tags:
  - math
  - level/Lv2
  - category/coordinate_spaces
---

# View & Projection Space: From 3D World to 2D Screen

> [!abstract] **The Concept in a Nutshell**
> Getting a 3D scene onto your 2D monitor involves a pipeline of coordinate space transformations: **World Space → View Space → Clip Space → NDC → Screen Space**. The **view matrix** repositions everything relative to the camera, the **projection matrix** applies perspective (or orthographic) foreshortening and maps the visible volume to a normalized cube, and the **viewport transform** maps that cube to actual pixel coordinates.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: An Open-World RPG Camera System**
> The player is exploring a vast fantasy world. Their character stands at world position $(200, 5, -340)$. The camera hovers behind them at $(195, 8, -345)$, looking toward the character. A distant mountain at $(2000, 300, -1000)$ looms in the background; a tiny firefly at $(200.5, 5.5, -339)$ flickers near the character's shoulder.
>
> The rendering pipeline must determine: Where does each of these objects appear on the player's 1920×1080 monitor? The mountain is far away and should appear small. The firefly is close and should appear near the center. Objects behind the camera should not be drawn at all.
>
> Every vertex in the scene passes through the **MVP pipeline** — Model (local→world), View (world→camera), Projection (camera→clip) — and out come 2D pixel coordinates. Understanding this pipeline is essential for shader programming, UI rendering, mouse picking, and custom camera systems.

---

## The Blueprint (Formula & Structure)

### The Full Transformation Pipeline

$$\mathbf{p}_\text{clip} = \mathbf{P} \cdot \mathbf{V} \cdot \mathbf{M} \cdot \mathbf{p}_\text{local}$$

| Stage | Matrix | From → To | What It Does |
|---|---|---|---|
| Model | $\mathbf{M}$ | Local → World | Places object in the world |
| View | $\mathbf{V}$ | World → Camera/Eye | Moves world so camera is at origin |
| Projection | $\mathbf{P}$ | Camera → Clip | Applies perspective, maps frustum to cube |
| *Perspective Divide* | — | Clip → NDC | Divides by $w$ component |
| *Viewport* | — | NDC → Screen | Maps $[-1,1]$ to pixel coordinates |

### Step 1: View Matrix (World → Camera Space)

The camera has a position $\mathbf{e}$ (eye), a forward direction, and an up vector. The view matrix is the **inverse of the camera's model matrix**:

$$\mathbf{V} = \mathbf{M}_\text{camera}^{-1}$$

For a LookAt camera with eye $\mathbf{e}$, target $\mathbf{t}$, and world-up $\hat{\mathbf{u}}$:

$$\hat{\mathbf{f}} = \text{normalize}(\mathbf{t} - \mathbf{e}) \quad \text{(forward)}$$
$$\hat{\mathbf{r}} = \text{normalize}(\hat{\mathbf{f}} \times \hat{\mathbf{u}}) \quad \text{(right)}$$
$$\hat{\mathbf{u}}_\text{cam} = \hat{\mathbf{r}} \times \hat{\mathbf{f}} \quad \text{(camera up)}$$

$$\mathbf{V} = \begin{bmatrix} r_x & r_y & r_z & -\hat{\mathbf{r}} \cdot \mathbf{e} \\ u_x & u_y & u_z & -\hat{\mathbf{u}}_\text{cam} \cdot \mathbf{e} \\ -f_x & -f_y & -f_z & \hat{\mathbf{f}} \cdot \mathbf{e} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

*(Note: Unity uses a left-handed coordinate system where camera looks down $+z$ in view space, so the forward sign may differ.)*

### Step 2: Projection Matrix (Camera → Clip Space)

The projection matrix maps the viewing volume (frustum) into a **clip-space cube**. After perspective division, coordinates are in **Normalized Device Coordinates (NDC)**:

- **OpenGL NDC:** $x, y, z \in [-1, 1]$
- **DirectX/Vulkan NDC:** $x, y \in [-1, 1]$, $z \in [0, 1]$

### Step 3: Perspective Divide (Clip → NDC)

After the projection matrix, we have homogeneous clip coordinates $(x_c, y_c, z_c, w_c)$. The perspective divide produces NDC:

$$x_\text{ndc} = \frac{x_c}{w_c}, \quad y_\text{ndc} = \frac{y_c}{w_c}, \quad z_\text{ndc} = \frac{z_c}{w_c}$$

This is where perspective foreshortening happens — far-away objects get divided by a larger $w$, making them appear smaller.

### Step 4: Viewport Transform (NDC → Screen)

Given a viewport of width $W$ and height $H$:

$$x_\text{screen} = \frac{(x_\text{ndc} + 1)}{2} \cdot W$$
$$y_\text{screen} = \frac{(1 - y_\text{ndc})}{2} \cdot H$$

*(The $y$-flip is because screen space has $y=0$ at the **top**.)*

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Photographer's Process**
> Imagine you're a photographer shooting a scene:
>
> 1. **World Space** → The real, physical world. Objects are where they are.
> 2. **View Space** → You position yourself and aim your camera. Now everything is relative to YOUR eyes — left, right, near, far.
> 3. **Clip Space** → Your viewfinder has a rectangular frame. Things outside the frame are cropped (clipped). Things farther away appear smaller in the frame (perspective).
> 4. **NDC** → You normalize the frame to a standard size. Whether your print is 4×6 or poster-sized, the *proportions* are the same.
> 5. **Screen Space** → The final print: actual pixels on actual paper (or monitor).
>
> The view matrix is *"where am I standing and looking?"*. The projection matrix is *"what lens am I using?"*. The viewport is *"what size is my print?"*.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Understanding the full MVP pipeline
// Manual world-to-screen conversion vs Unity's built-in methods

using UnityEngine;

public class ViewProjectionDemo : MonoBehaviour
{
    public Camera mainCam;
    public Transform targetObject;

    void Update()
    {
        // === METHOD 1: Unity's built-in ===
        // WorldToScreenPoint does the entire MVP + viewport transform
        Vector3 screenPos = mainCam.WorldToScreenPoint(targetObject.position);
        Debug.Log($"Screen pos (Unity): {screenPos}");
        // screenPos.z = distance from camera (depth)

        // === METHOD 2: Manual pipeline ===
        // Step 1: Get the view and projection matrices
        Matrix4x4 viewMatrix = mainCam.worldToCameraMatrix;
        Matrix4x4 projMatrix = mainCam.projectionMatrix;

        // Step 2: Transform world position to view space
        Vector3 worldPos = targetObject.position;
        Vector4 viewPos = viewMatrix * new Vector4(worldPos.x, worldPos.y,
                                                    worldPos.z, 1f);

        // Step 3: Transform to clip space
        Vector4 clipPos = projMatrix * viewPos;

        // Step 4: Perspective divide → NDC
        Vector3 ndc = new Vector3(clipPos.x / clipPos.w,
                                   clipPos.y / clipPos.w,
                                   clipPos.z / clipPos.w);

        // Step 5: Viewport transform → screen pixels
        float screenX = (ndc.x + 1f) * 0.5f * Screen.width;
        float screenY = (ndc.y + 1f) * 0.5f * Screen.height;
        Debug.Log($"Screen pos (manual): ({screenX}, {screenY})");

        // Check if object is in front of the camera
        // In Unity's view space, camera looks down -Z, so objects
        // in front have NEGATIVE z in view space
        bool inFrontOfCamera = viewPos.z < 0f;
        Debug.Log($"In front of camera: {inFrontOfCamera}");

        // Reverse: screen point to world ray (for mouse picking)
        Ray pickRay = mainCam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(pickRay, out RaycastHit hit, 100f))
        {
            Debug.Log($"Mouse is over: {hit.collider.name}");
        }
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What are the five coordinate spaces in the rendering pipeline (in order)? :: **Local/Object → World → View/Camera → Clip → NDC → Screen**. The Model matrix handles step 1, View matrix step 2, Projection matrix step 3, perspective divide step 4, and viewport transform step 5.
- What is the view matrix and how is it constructed? :: The view matrix transforms world space to camera space. It is the **inverse of the camera's world transform** ($\mathbf{V} = \mathbf{M}_\text{camera}^{-1}$). It effectively "moves the universe" so the camera is at the origin looking down an axis.
- What does the perspective divide do? :: Divides clip-space coordinates $(x_c, y_c, z_c)$ by the $w_c$ component. This creates **perspective foreshortening** — objects farther from the camera have a larger $w$, so they appear smaller after division.
- What is NDC and what is its range? :: **Normalized Device Coordinates** — the coordinate system after projection and perspective divide. In OpenGL: $[-1, 1]$ for all axes. In DirectX: $[-1,1]$ for $x,y$ and $[0,1]$ for $z$. It's a device-independent representation of what's visible.
- In Unity, which built-in function converts a world position to screen pixels? :: `Camera.WorldToScreenPoint(worldPos)` performs the full View → Projection → Perspective Divide → Viewport pipeline and returns pixel coordinates (with $z$ = distance from camera).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting that `WorldToScreenPoint` returns a $z$ component that represents depth, then using it as a 2D position and getting confused by the third value.
> - **The Fix:** The $z$ component is the **distance from the camera** along its forward axis. For 2D screen placement, use only $x$ and $y$. The $z$ is useful for depth sorting.
> - **Why:** The function packs depth info into the return value for convenience. If $z < 0$, the object is **behind** the camera and the $x, y$ values are meaningless.

> [!danger] **Watch Out!**
> - **The Mistake:** Confusing view-space axes across APIs. In Unity, the camera looks down $-z$ in view space (OpenGL convention), but in DirectX the camera typically looks down $+z$.
> - **The Fix:** Always check which convention your engine uses. In Unity, objects in front of the camera have **negative** $z$ in view space.
> - **Why:** The handedness of the coordinate system differs between graphics APIs. Code that works in one convention will place objects behind the camera in another.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/object_world_space|Object & World Space]]
- [[Math/05_Coordinate_Spaces/perspective_projection|Perspective Projection]]
- [[Math/11_Graphics_Math/rendering_pipeline|Rendering Pipeline]]
