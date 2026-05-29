---
title: "Shadow Mathematics: Darkness by Design"
tags:
  - math
  - level/Lv4
  - category/graphics_math
---

# Shadow Mathematics: Darkness by Design

> [!abstract] **The Concept in a Nutshell**
> Shadows are computed by asking a deceptively simple question: "Can this surface point see the light?" Shadow mapping answers this by rendering the scene from the light's perspective into a depth buffer, then comparing each fragment's light-space depth against the stored value. If the fragment is farther than what the light "sees," it's in shadow. Artifacts like shadow acne and peter-panning require careful bias tuning, and large open worlds demand Cascaded Shadow Maps (CSM) to maintain quality across distances.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Horror Game Flashlight in an Abandoned Hospital**
> The player's flashlight casts a cone of light down a dark hallway. Without shadows, every object in the cone is uniformly lit — the gurney, the overturned wheelchair, the figure standing in the doorway all blend into a flat, non-threatening mess. With shadow mapping, the gurney casts a long shadow behind it, the wheelchair's spokes create intricate shadow patterns on the floor, and the figure in the doorway casts a looming shadow stretching toward the player. But implementation details matter: **shadow acne** creates an eerie zebra-stripe pattern on the floor (a bug, not a feature), and the **shadow map resolution** determines whether the figure's shadow is a crisp silhouette or a blurry blob. Getting shadows right is the difference between "horror" and "broken tech demo."

---

## The Blueprint (Formula & Structure)

### Shadow Mapping Algorithm

**Pass 1: Render Depth from Light's POV**

Construct a **light-space matrix** (view + projection from the light's position/direction):

$$M_{\text{lightSpace}} = P_{\text{light}} \cdot V_{\text{light}}$$

For a **directional light**: use orthographic projection.
For a **spot light**: use perspective projection.
For a **point light**: use 6 perspective projections (cubemap).

Render the scene using $M_{\text{lightSpace}}$, storing only depth values in a texture (the **shadow map**).

**Pass 2: Shadow Test per Fragment**

For each visible fragment at world position $\vec{p}$:

1. Transform to light space: $\vec{p}_{\text{light}} = M_{\text{lightSpace}} \cdot \vec{p}$
2. Perform perspective divide: $(u, v, d_{\text{frag}}) = (x/w, y/w, z/w)$
3. Map to texture coordinates: $u' = 0.5u + 0.5, \quad v' = 0.5v + 0.5$
4. Sample shadow map: $d_{\text{map}} = \text{ShadowMap}(u', v')$
5. Compare: if $d_{\text{frag}} > d_{\text{map}} + \text{bias}$, the fragment is **in shadow**

$$\text{shadow} = \begin{cases} 0 & \text{if } d_{\text{frag}} > d_{\text{map}} + \text{bias (in shadow)} \\ 1 & \text{if } d_{\text{frag}} \leq d_{\text{map}} + \text{bias (lit)} \end{cases}$$

### Shadow Acne and Bias

**Shadow acne** occurs because the shadow map has finite resolution. A surface lit at an angle self-shadows due to quantization — adjacent texels alternate between "lit" and "shadowed," creating striped artifacts.

**Depth bias** offsets the comparison:

$$\text{bias} = \max\left(\text{maxBias} \cdot (1.0 - \vec{N} \cdot \vec{L}), \text{minBias}\right)$$

Surfaces at steep angles to the light need more bias. Too much bias causes **peter-panning** — shadows detach from their casters, making objects appear to float.

### PCF — Percentage Closer Filtering

Instead of a binary shadow test, sample multiple neighboring texels and average:

$$\text{shadow}_{\text{PCF}} = \frac{1}{N} \sum_{i=1}^{N} \text{compare}(d_{\text{frag}}, d_{\text{map}}(u + \delta_i, v + \delta_i))$$

Common kernel sizes: $3 \times 3$ (9 samples), $5 \times 5$ (25 samples). This produces soft shadow edges. Larger kernels = softer but more expensive.

### Cascaded Shadow Maps (CSM)

For large open worlds with directional lights (the sun), a single shadow map can't cover the entire view — nearby objects need high resolution, distant objects can be lower.

CSM splits the view frustum into $K$ cascades (typically 2-4):

$$\text{Cascade } k: z_{\text{near}}^{(k)} \text{ to } z_{\text{far}}^{(k)}$$

Each cascade gets its own shadow map rendered from the light's perspective, covering only its depth range. Common split schemes:

**Logarithmic split:**
$$z_i = z_{\text{near}} \cdot \left(\frac{z_{\text{far}}}{z_{\text{near}}}\right)^{i/K}$$

**Practical split (PSSM):**
$$z_i = \lambda \cdot z_{\text{log}} + (1 - \lambda) \cdot z_{\text{uniform}}$$

Where $\lambda \in [0, 1]$ blends between logarithmic and uniform distribution.

### Shadow Volumes (Stencil Shadows)

An alternative approach: extrude silhouette edges of shadow casters to infinity, forming volumes. Use the stencil buffer to track how many shadow volume surfaces are between the camera and each pixel:

- Front-face of shadow volume: increment stencil
- Back-face of shadow volume: decrement stencil
- Stencil ≠ 0: pixel is in shadow

**Pros:** Pixel-perfect hard shadows, no resolution artifacts.
**Cons:** Fill-rate intensive, complex silhouette detection, no soft shadows.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Photographer's Light Meter**
> Imagine placing a camera exactly where the light source is and taking a depth photo. Every pixel in that photo records "how far away is the nearest surface the light can see?" Now, for each pixel on YOUR screen, you ask: "If I trace a line from this pixel to the light, how far is it?" If it's farther than what the light's camera recorded, something is blocking the light — you're in shadow. Shadow bias is like adjusting the light-camera's focus — too precise and you get artifacts from rounding errors; too loose and shadows "detach" from objects. CSM is like the photographer using a telephoto lens for nearby detail and a wide-angle lens for the background, each with its own exposure settings.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Shadow map visualization and bias adjustment
using UnityEngine;

public class ShadowMapDebug : MonoBehaviour
{
    [SerializeField] private Light directionalLight;
    [SerializeField] private Camera shadowCamera; // camera simulating light's view
    [SerializeField] private RenderTexture shadowMap;
    [Range(0.001f, 0.05f)]
    [SerializeField] private float shadowBias = 0.005f;

    void Start()
    {
        // Configure the shadow camera to match the directional light
        shadowCamera.orthographic = true;          // directional light uses ortho
        shadowCamera.orthographicSize = 20f;       // covers 40x40 world units
        shadowCamera.targetTexture = shadowMap;    // render depth to texture
        shadowCamera.depthTextureMode = DepthTextureMode.Depth;
    }

    // Simulated shadow test for a world-space point
    bool IsInShadow(Vector3 worldPoint)
    {
        // Transform world point to light's clip space
        Vector3 lightSpacePos = shadowCamera.WorldToViewportPoint(worldPoint);

        // Check if point is within shadow map bounds
        if (lightSpacePos.x < 0 || lightSpacePos.x > 1 ||
            lightSpacePos.y < 0 || lightSpacePos.y > 1 ||
            lightSpacePos.z < 0)
            return false; // outside light's view

        // Fragment depth from light's perspective (normalized)
        float fragDepth = lightSpacePos.z / shadowCamera.farClipPlane;

        // In a real implementation, you'd sample the shadow map texture here:
        // float mapDepth = shadowMap.Sample(lightSpacePos.xy);
        // return fragDepth > mapDepth + shadowBias;

        Debug.Log($"Light-space UV: ({lightSpacePos.x:F2}, {lightSpacePos.y:F2}), Depth: {fragDepth:F4}");
        return false; // placeholder
    }

    void Update()
    {
        // Sync shadow camera to light orientation
        shadowCamera.transform.rotation = directionalLight.transform.rotation;
        shadowCamera.transform.position = directionalLight.transform.position;

        // Visualize: configure Unity's built-in shadow settings
        QualitySettings.shadowDistance = 150f;       // CSM max distance
        QualitySettings.shadowCascades = 4;          // 4 cascade levels
        QualitySettings.shadowResolution = ShadowResolution.VeryHigh;

        // Test a point
        IsInShadow(Vector3.zero);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What are the two main passes in shadow mapping? :: **Pass 1:** Render the scene from the light's perspective, storing depth values in a shadow map texture. **Pass 2:** For each visible fragment, compare its light-space depth to the shadow map value — if farther, it's in shadow.
- What causes shadow acne, and how is it fixed? :: Shadow acne occurs when a surface self-shadows due to depth map quantization at steep angles. It's fixed with a **depth bias** that offsets the comparison threshold, preventing false self-shadowing.
- What is peter-panning and what causes it? :: Peter-panning is when shadows visibly detach from their casting objects, making them appear to float. It's caused by using **too much shadow bias** — the offset pushes the shadow start point too far from the actual surface.
- How do Cascaded Shadow Maps improve quality for large scenes? :: CSM splits the view frustum into distance ranges (cascades), each with its own shadow map. Near objects get high-resolution shadow maps for sharp detail, while distant objects use lower resolution, efficiently distributing the shadow budget.
- What is PCF and why is it used? :: Percentage Closer Filtering samples multiple shadow map texels around the lookup point and averages the comparison results. This produces soft shadow edges instead of hard, aliased boundaries.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using a fixed shadow bias value for all surfaces regardless of their angle to the light.
> - **The Fix:** Use a slope-scaled bias: `bias = max(maxBias * (1.0 - NdotL), minBias)`. Surfaces at steep angles (low $\vec{N} \cdot \vec{L}$) get more bias.
> - **Why:** Surfaces nearly parallel to the light direction have extreme depth changes across a single shadow map texel, requiring larger bias. Surfaces facing the light directly need minimal bias.

> [!danger] **Watch Out!**
> - **The Mistake:** Shadow map resolution too low for the scene, causing blocky, pixelated shadow edges.
> - **The Fix:** For directional lights, use CSM. For spot/point lights, increase shadow map resolution or reduce the light's coverage area. Consider using PCF to soften aliased edges.
> - **Why:** Each shadow map texel covers a finite world area. If that area is large (low resolution or wide coverage), shadow edges jump between texels, creating visible staircase patterns.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/perspective_projection|Perspective Projection]]
- [[Math/05_Coordinate_Spaces/orthographic_projection|Orthographic Projection]]
- [[Math/11_Graphics_Math/rendering_pipeline|The Rendering Pipeline]]
