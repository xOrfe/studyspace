---
title: "The Rendering Pipeline: From Vertices to Pixels"
tags:
  - math
  - level/Lv3
  - category/graphics_math
---

# The Rendering Pipeline: From Vertices to Pixels

> [!abstract] **The Concept in a Nutshell**
> The rendering pipeline is the sequence of stages that transforms 3D mesh data into 2D colored pixels on your screen. Understanding each stage — vertex processing, primitive assembly, rasterization, fragment processing, and output merging — lets you write efficient shaders, debug visual artifacts, and push every frame closer to your performance budget.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Fantasy RPG Village at Sunset**
> Your scene has 200,000 triangles: thatched roofs, cobblestone roads, an animated dragon flying overhead. When the player presses "Play," every single triangle goes through the pipeline. The **vertex shader** transforms the dragon's wing vertices from local bone space into screen coordinates. **Clipping** discards the houses behind the camera. **Rasterization** determines which screen pixels each triangle covers. The **fragment shader** paints each pixel with textures, lighting, and shadows. Finally, **depth testing** ensures the dragon's wing renders in front of the distant mountains, not behind them. A single bug in any stage — wrong matrix, inverted normals, bad depth write — and the entire illusion shatters.

---

## The Blueprint (Formula & Structure)

The modern GPU pipeline flows through these stages:

### 1. Vertex Processing (Vertex Shader)
Each vertex is transformed through the **MVP matrix chain**:

$$\vec{p}_{\text{clip}} = M_{\text{projection}} \cdot M_{\text{view}} \cdot M_{\text{model}} \cdot \vec{p}_{\text{local}}$$

The vertex shader also computes per-vertex data: normals, tangents, UVs, and any custom attributes passed to later stages.

### 2. Primitive Assembly
Vertices are grouped into **primitives** (triangles, lines, points) according to the index buffer. A triangle list with indices `[0,1,2, 2,1,3, ...]` assembles vertex triplets.

### 3. Clipping
Primitives partially or fully outside the **view frustum** are clipped against 6 planes. Clip-space coordinates satisfy:

$$-w \leq x \leq w, \quad -w \leq y \leq w, \quad 0 \leq z \leq w$$

Triangles straddling a clip plane are split into smaller triangles.

### 4. Perspective Divide & Viewport Transform
Clip coordinates become **Normalized Device Coordinates (NDC)**:

$$x_{\text{ndc}} = \frac{x_{\text{clip}}}{w_{\text{clip}}}, \quad y_{\text{ndc}} = \frac{y_{\text{clip}}}{w_{\text{clip}}}, \quad z_{\text{ndc}} = \frac{z_{\text{clip}}}{w_{\text{clip}}}$$

NDC are then mapped to screen pixel coordinates via the viewport transform:

$$x_{\text{screen}} = \frac{(x_{\text{ndc}} + 1)}{2} \cdot \text{width}, \quad y_{\text{screen}} = \frac{(1 - y_{\text{ndc}})}{2} \cdot \text{height}$$

### 5. Rasterization
The rasterizer determines which **fragments** (potential pixels) lie inside each triangle, using edge functions or scanline algorithms. For each fragment, vertex attributes are **interpolated** using barycentric coordinates:

$$A_{\text{frag}} = \alpha \cdot A_0 + \beta \cdot A_1 + \gamma \cdot A_2, \quad \alpha + \beta + \gamma = 1$$

### 6. Fragment Processing (Fragment/Pixel Shader)
Each fragment receives interpolated data and computes a final color. This is where **texture sampling**, **lighting calculations**, **normal mapping**, and **shadow lookups** happen.

### 7. Output Merging
The final stage performs:
- **Depth Test:** Compare fragment depth against the depth buffer; discard if behind existing geometry.
- **Stencil Test:** Mask rendering to specific regions.
- **Blending:** Combine fragment color with framebuffer color for transparency:

$$C_{\text{final}} = \alpha_{\text{src}} \cdot C_{\text{src}} + (1 - \alpha_{\text{src}}) \cdot C_{\text{dst}}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Assembly Line Factory**
> Think of the GPU as a factory assembly line. Raw materials (vertices) enter one end. Station 1 (vertex shader) shapes them and stamps coordinates. Station 2 (primitive assembly) groups parts into products (triangles). Station 3 (clipping) removes defective items that fall off the conveyor. Station 4 (rasterization) is a cookie-cutter that stamps triangle shapes onto a pixel grid. Station 5 (fragment shader) is the paint shop — each pixel gets its final color. Station 6 (output merging) is quality control — only the closest, correctly masked, properly blended pixels make it to the final display. The key insight: **every stage runs in parallel across thousands of cores**, processing millions of items simultaneously.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Visualizing where a vertex ends up on screen (the pipeline in code)
using UnityEngine;

public class PipelineVisualizer : MonoBehaviour
{
    [SerializeField] private Camera mainCam;

    void Update()
    {
        // Stage 1: Model Space → World Space (Model matrix)
        Vector3 localPos = Vector3.zero; // vertex at mesh origin
        Vector3 worldPos = transform.TransformPoint(localPos);

        // Stage 2-4: World Space → Screen Space (View + Projection + Viewport)
        Vector3 screenPos = mainCam.WorldToScreenPoint(worldPos);

        // screenPos.x, screenPos.y = pixel coordinates
        // screenPos.z = depth from camera (used in depth test)

        Debug.Log($"World: {worldPos} → Screen: ({screenPos.x:F0}, {screenPos.y:F0}) Depth: {screenPos.z:F2}");

        // Checking if vertex is within the view frustum (clipping stage)
        Vector3 viewportPos = mainCam.WorldToViewportPoint(worldPos);
        bool insideFrustum = viewportPos.x >= 0 && viewportPos.x <= 1 &&
                             viewportPos.y >= 0 && viewportPos.y <= 1 &&
                             viewportPos.z > 0; // in front of camera

        if (!insideFrustum)
            Debug.Log("Vertex is CLIPPED (outside frustum)");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What transformation does the vertex shader primarily perform? :: It transforms vertices from local/model space to clip space using the MVP matrix: $\vec{p}_{\text{clip}} = P \cdot V \cdot M \cdot \vec{p}_{\text{local}}$.
- What is the purpose of the perspective divide? :: It converts 4D clip coordinates to 3D NDC by dividing $x$, $y$, $z$ by $w$, which produces the foreshortening effect where distant objects appear smaller.
- How does rasterization determine which pixels a triangle covers? :: It uses edge functions or scanline algorithms to test each pixel sample against the triangle's edges, then interpolates vertex attributes via barycentric coordinates.
- What happens during the depth test in output merging? :: Each fragment's depth is compared to the value already in the depth buffer; if the fragment is farther away (greater depth), it is discarded; otherwise it overwrites the buffer.
- Why is alpha blending order-dependent for transparency? :: Because the blending equation $C = \alpha \cdot C_{\text{src}} + (1 - \alpha) \cdot C_{\text{dst}}$ depends on which fragment is processed first; rendering back-to-front produces correct results, while arbitrary order creates visual errors.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting that matrix multiplication order matters — writing `Model * View * Projection` instead of `Projection * View * Model`.
> - **The Fix:** Always multiply in the order **P × V × M × vertex**. In Unity's shader code: `UnityObjectToClipPos(v.vertex)` handles this automatically.
> - **Why:** Matrix multiplication is not commutative. The vertex must first go to world space (M), then camera space (V), then clip space (P). Reversing the order applies the wrong transformation at each stage.

> [!danger] **Watch Out!**
> - **The Mistake:** Rendering transparent objects without sorting them back-to-front.
> - **The Fix:** Use Unity's render queue system (`Transparent` queue at 3000+) which sorts by distance. For complex cases, use Order-Independent Transparency (OIT) techniques.
> - **Why:** The depth buffer works on a binary pass/fail basis — it cannot blend partially transparent fragments correctly when rendered out of order.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/view_projection_space|View & Projection Space]]
- [[Math/07_Geometric_Primitives/triangles_meshes|Triangles & Meshes]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
