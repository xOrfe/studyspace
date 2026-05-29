---
title: "Texture & Normal Mapping: Detail Without Geometry"
tags:
  - math
  - level/Lv3
  - category/graphics_math
---

# Texture & Normal Mapping: Detail Without Geometry

> [!abstract] **The Concept in a Nutshell**
> Texture mapping wraps 2D images onto 3D surfaces using UV coordinates — like applying a sticker to a model. Normal mapping takes this further: instead of adding actual geometric detail (expensive polygons), it stores perturbed surface normals in a texture, tricking the lighting system into shading flat surfaces as if they had bumps, grooves, and ridges. The TBN matrix (Tangent, Bitangent, Normal) bridges the gap between the normal map's tangent space and world space lighting.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Medieval Castle Wall in an Open-World RPG**
> The castle wall is a single flat quad — just 2 triangles, 4 vertices. Yet it looks like intricately carved stone blocks with deep mortar grooves. A **diffuse texture** provides the color variation (gray stone, dark mortar lines). A **normal map** stores the brick-by-brick surface angle deviations encoded as an RGB image (that familiar purple-blue color). When the sun crosses the sky, shadows shift between bricks — the grooves darken and the raised surfaces brighten — all from a simple dot product between the perturbed normal and the light direction. Without normal mapping, this level of detail would require 50,000+ polygons per wall segment. With it: 2 triangles + 2 textures. That's the power of faking geometry with math.

---

## The Blueprint (Formula & Structure)

### UV Coordinate Mapping

Every vertex stores a 2D coordinate $(u, v)$ where $u, v \in [0, 1]$ that maps to a position in the texture image:

$$\text{pixel}_{x} = u \times \text{width}, \quad \text{pixel}_{y} = v \times \text{height}$$

UV coordinates are interpolated across the triangle surface via barycentric coordinates during rasterization. The GPU then **samples** the texture at the interpolated UV position.

### Texture Filtering: Bilinear Interpolation

When a UV coordinate falls between texel centers, bilinear filtering blends the 4 nearest texels:

$$C = (1-s)(1-t) \cdot C_{00} + s(1-t) \cdot C_{10} + (1-s)t \cdot C_{01} + st \cdot C_{11}$$

Where $s, t$ are the fractional offsets within the texel grid. This prevents the blocky/pixelated look of nearest-neighbor sampling.

### Normal Map Encoding

A normal map stores perturbed normals in **tangent space** as RGB colors:

$$\vec{N}_{\text{tangent}} = 2.0 \times (R, G, B) - 1.0$$

- Red channel → X component (left/right deviation)
- Green channel → Y component (up/down deviation)
- Blue channel → Z component (outward from surface)

A flat surface with no perturbation encodes as $(0.5, 0.5, 1.0)$ — the characteristic purple/blue color of normal maps, which decodes to $(0, 0, 1)$ — straight outward.

### The TBN Matrix (Tangent-Bitangent-Normal)

To use tangent-space normals in world-space lighting, we construct the **TBN matrix** per-vertex:

$$\text{TBN} = \begin{bmatrix} T_x & B_x & N_x \\ T_y & B_y & N_y \\ T_z & B_z & N_z \end{bmatrix}$$

Where:
- $\vec{T}$ = tangent vector (aligned with the U texture direction)
- $\vec{B}$ = bitangent vector (aligned with the V texture direction)
- $\vec{N}$ = geometric surface normal

**Computing T and B from triangle edges and UV deltas:**

Given triangle edges $\vec{E}_1 = P_1 - P_0$ and $\vec{E}_2 = P_2 - P_0$, and UV deltas $\Delta UV_1, \Delta UV_2$:

$$\begin{bmatrix} \vec{T} \\ \vec{B} \end{bmatrix} = \frac{1}{\Delta u_1 \Delta v_2 - \Delta u_2 \Delta v_1} \begin{bmatrix} \Delta v_2 & -\Delta v_1 \\ -\Delta u_2 & \Delta u_1 \end{bmatrix} \begin{bmatrix} \vec{E}_1 \\ \vec{E}_2 \end{bmatrix}$$

**Transforming the sampled normal to world space:**

$$\vec{N}_{\text{world}} = \text{TBN} \times \vec{N}_{\text{tangent}}$$

This perturbed world-space normal is then used in all lighting calculations (Lambert, Phong, PBR) as a drop-in replacement for the geometric normal.

### Wrap Modes and Mipmapping

**Wrap modes** control what happens at UV boundaries:
- **Repeat:** UV wraps around (tiling textures)
- **Clamp:** UV is clamped to $[0, 1]$ (edge pixels stretch)

**Mipmaps** are pre-computed downscaled versions of the texture (512 → 256 → 128 → ...). The GPU selects the appropriate mip level based on how many texels cover a single screen pixel, preventing aliasing (shimmering patterns on distant surfaces).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Gift Wrapping vs. Embossed Wallpaper**
> **UV mapping** is like gift-wrapping a box — you cut a flat piece of wrapping paper (the texture) and fold it around the 3D shape, matching corners to UV coordinates. **Normal mapping** is like using embossed wallpaper — the wallpaper is flat, but the embossed bumps catch light differently, creating an illusion of depth. The TBN matrix is the instruction manual that tells you: "the bumps pointing 'up' in the wallpaper pattern correspond to THIS direction on the actual wall." Without TBN, the bumps would light incorrectly — as if the wallpaper was applied upside-down or sideways.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Manually sampling a normal map and computing TBN
// (Demonstrates what the GPU does automatically in shader)
using UnityEngine;

public class NormalMapDemo : MonoBehaviour
{
    [SerializeField] private Texture2D normalMap;
    [SerializeField] private Light sceneLight;

    void Update()
    {
        // Simulate UV coordinates at the center of the object
        Vector2 uv = new Vector2(0.5f, 0.5f);

        // Sample the normal map (returns color in [0,1] range)
        Color normalColor = normalMap.GetPixelBilinear(uv.x, uv.y);

        // Decode from [0,1] to [-1,1] tangent-space normal
        Vector3 tangentNormal = new Vector3(
            normalColor.r * 2f - 1f,
            normalColor.g * 2f - 1f,
            normalColor.b * 2f - 1f
        ).normalized;

        // Build TBN matrix from the object's orientation
        // In real shaders, T and B come from mesh tangent data
        Vector3 N = transform.up;       // surface normal
        Vector3 T = transform.right;    // tangent (U direction)
        Vector3 B = transform.forward;  // bitangent (V direction)

        // Transform tangent-space normal to world space: TBN * normalTangent
        Vector3 worldNormal = (T * tangentNormal.x +
                               B * tangentNormal.y +
                               N * tangentNormal.z).normalized;

        // Use the perturbed normal for lighting (Lambert diffuse)
        Vector3 lightDir = (sceneLight.transform.position - transform.position).normalized;
        float NdotL = Mathf.Max(0f, Vector3.Dot(worldNormal, lightDir));

        Debug.Log($"Tangent Normal: {tangentNormal} → World Normal: {worldNormal} | NdotL: {NdotL:F2}");
        Debug.DrawRay(transform.position, worldNormal * 2f, Color.blue);
        Debug.DrawRay(transform.position, N * 2f, Color.green); // geometric normal for comparison
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What color does a flat (unperturbed) normal map appear, and why? :: Purple/blue — RGB $(0.5, 0.5, 1.0)$. Decoded: $(2 \times 0.5 - 1, 2 \times 0.5 - 1, 2 \times 1.0 - 1) = (0, 0, 1)$, pointing straight outward in tangent space — no deviation from the surface normal.
- What does the TBN matrix do? :: It transforms normals from **tangent space** (the coordinate system of the normal map texture) to **world space** (where lighting calculations happen). The columns are the tangent, bitangent, and normal vectors.
- Why is bilinear filtering important for texture sampling? :: When a UV coordinate falls between texel centers, nearest-neighbor sampling creates blocky artifacts. Bilinear filtering blends the 4 nearest texels for a smooth result, essential for surfaces viewed at non-integer texel alignment.
- Why are normal maps stored in tangent space rather than world or object space? :: Tangent space normals are relative to the surface orientation, making them **reusable across different meshes** and independent of object rotation. A brick normal map works on walls, floors, or ceilings without modification.
- What are mipmaps and why do they prevent aliasing? :: Mipmaps are pre-downscaled versions of a texture. When a surface is far from the camera (many texels per pixel), the GPU samples a lower-resolution mip, preventing the shimmering Moiré patterns that occur when high-frequency texture detail is undersampled.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using a normal map as a regular color/diffuse texture, or importing a normal map without setting the texture type to "Normal Map" in Unity.
> - **The Fix:** Always mark normal map textures as "Normal Map" type in the import settings. Unity will handle the correct compression format (e.g., DXT5nm which stores only X and Y, reconstructing Z).
> - **Why:** Normal maps need special treatment: they must not be gamma-corrected (they're data, not color), and engine-specific compression formats preserve the precision needed for accurate lighting.

> [!danger] **Watch Out!**
> - **The Mistake:** Flipped green channel — normals appear inverted (bumps look like dents).
> - **The Fix:** Check whether your normal map was authored for OpenGL (Y-up) or DirectX (Y-down) convention. Unity uses OpenGL-style (green = up). Flip the green channel if importing DirectX-convention maps.
> - **Why:** Different tools and engines use different tangent-space conventions for the Y axis. Using the wrong convention inverts the bitangent direction, making raised areas shade as indentations.

---

## Related Topics
- [[Math/07_Geometric_Primitives/normal_vectors|Normal Vectors]]
- [[Math/07_Geometric_Primitives/triangles_meshes|Triangles & Meshes]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
