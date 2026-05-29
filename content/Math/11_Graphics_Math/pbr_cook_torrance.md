---
title: "PBR & Cook-Torrance: Rendering Reality"
tags:
  - math
  - level/Lv4
  - category/graphics_math
---

# PBR & Cook-Torrance: Rendering Reality

> [!abstract] **The Concept in a Nutshell**
> Physically Based Rendering (PBR) replaces ad-hoc lighting hacks with a mathematically rigorous model grounded in real physics. The Cook-Torrance BRDF (Bidirectional Reflectance Distribution Function) describes how light bounces off a surface using three components: a **microfacet distribution** (D), a **Fresnel** term (F), and a **geometry/shadowing** term (G). This approach ensures materials look correct under any lighting condition — the foundation of modern AAA game visuals.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Unreal Engine Material Editor — Crafting a Battle-Worn Shield**
> An artist adjusts two sliders: **Metallic** (0.0 → 1.0) and **Roughness** (0.0 → 1.0). At Metallic=1, Roughness=0, the shield looks like polished chrome — a tight, bright specular reflection with colored metal tinting. At Metallic=0, Roughness=0.8, it looks like worn leather — broad, faint highlights with no Fresnel color shift. The artist doesn't tweak 15 arbitrary sliders — the PBR model's physics-based constraints ensure that **energy is conserved** (a surface can't reflect more light than it receives), and materials look equally correct under a sunset, a fluorescent office light, or a flashlight in a cave. This consistency across lighting conditions is why every major engine (Unity URP/HDRP, Unreal, Godot) adopted PBR as the default.

---

## The Blueprint (Formula & Structure)

### The Rendering Equation (Simplified)

$$L_o(\vec{v}) = L_e + \int_{\Omega} f_r(\vec{l}, \vec{v}) \cdot L_i(\vec{l}) \cdot (\vec{N} \cdot \vec{l}) \, d\vec{l}$$

Where:
- $L_o$ = outgoing radiance toward viewer $\vec{v}$
- $L_e$ = emitted light (for emissive surfaces)
- $f_r$ = BRDF — the core function describing surface reflectance
- $L_i$ = incoming radiance from direction $\vec{l}$
- $\vec{N} \cdot \vec{l}$ = Lambert's cosine factor

### The Cook-Torrance BRDF

The BRDF splits into **diffuse** and **specular** components:

$$f_r = k_d \cdot f_{\text{Lambert}} + k_s \cdot f_{\text{Cook-Torrance}}$$

$$f_{\text{Lambert}} = \frac{c}{\pi}$$

$$f_{\text{Cook-Torrance}} = \frac{D(\vec{H}) \cdot F(\vec{V}, \vec{H}) \cdot G(\vec{L}, \vec{V}, \vec{H})}{4 \cdot (\vec{N} \cdot \vec{L}) \cdot (\vec{N} \cdot \vec{V})}$$

Where $c$ is the surface albedo color, and $k_d + k_s = 1$ (energy conservation).

### D — Normal Distribution Function (NDF): GGX/Trowbridge-Reitz

$$D_{\text{GGX}}(\vec{H}) = \frac{\alpha^2}{\pi \cdot \left[(\vec{N} \cdot \vec{H})^2 \cdot (\alpha^2 - 1) + 1\right]^2}$$

Where $\alpha = \text{roughness}^2$. This function describes the statistical distribution of microfacet orientations. At low roughness, most microfacets align with the macro-normal → tight specular. At high roughness, they're scattered → broad, dim specular.

**Why GGX?** It has a longer "tail" than Blinn-Phong or Beckmann distributions, producing highlights that fade more gradually — matching real-world materials better.

### F — Fresnel: Schlick's Approximation

$$F_{\text{Schlick}}(\vec{V}, \vec{H}) = F_0 + (1 - F_0)(1 - \vec{V} \cdot \vec{H})^5$$

Where $F_0$ is the base reflectivity at normal incidence:
- **Dielectrics** (plastic, wood, stone): $F_0 \approx 0.04$ (grayscale)
- **Metals** (gold, copper, iron): $F_0 = \text{albedo color}$ (colored reflection)

At grazing angles ($\vec{V} \cdot \vec{H} \to 0$), $F \to 1$ — all surfaces become mirrors at steep angles. This is why lakes look reflective at sunset and transparent looking straight down.

### G — Geometry Function: Smith's Method with Schlick-GGX

$$G_{\text{SchlickGGX}}(\vec{N}, \vec{X}, k) = \frac{\vec{N} \cdot \vec{X}}{(\vec{N} \cdot \vec{X})(1 - k) + k}$$

$$G_{\text{Smith}}(\vec{N}, \vec{V}, \vec{L}) = G_{\text{SchlickGGX}}(\vec{N}, \vec{V}, k) \cdot G_{\text{SchlickGGX}}(\vec{N}, \vec{L}, k)$$

Where $k = \frac{(\text{roughness} + 1)^2}{8}$ for direct lighting. This models **self-shadowing** — rough microfacets block each other, reducing the effective specular at grazing angles.

### The Metallic/Roughness Workflow

| Property | Metallic = 0 (Dielectric) | Metallic = 1 (Metal) |
|---|---|---|
| $F_0$ | $(0.04, 0.04, 0.04)$ | Albedo color |
| Diffuse | Albedo × $(1 - F)$ | **Zero** (metals don't diffuse) |
| Specular | White/gray highlight | Colored highlight |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Bumpy Mirror at the Microscopic Level**
> Imagine zooming into a surface with an electron microscope. Even a "smooth" surface is covered in tiny mirror-like facets (microfacets), each oriented slightly differently. **D** describes how many facets point in the "right" direction to bounce light toward your eye. **F** describes how much light each facet reflects vs. absorbs (more at grazing angles, like skipping a stone on water). **G** describes how many facets block each other's light. **Roughness** is simply how chaotic the facet orientations are: low roughness = facets mostly aligned = sharp mirror reflection; high roughness = facets randomly tilted = blurry, scattered reflection. PBR doesn't invent new physics — it models what's actually happening at the micro-scale.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Computing Cook-Torrance BRDF terms manually
// (Educational — in practice, shaders handle this on the GPU)
using UnityEngine;

public class PBRCalculator : MonoBehaviour
{
    // GGX Normal Distribution Function
    static float DistributionGGX(Vector3 N, Vector3 H, float roughness)
    {
        float a = roughness * roughness;
        float a2 = a * a;
        float NdotH = Mathf.Max(Vector3.Dot(N, H), 0f);
        float NdotH2 = NdotH * NdotH;

        float denom = NdotH2 * (a2 - 1f) + 1f;
        denom = Mathf.PI * denom * denom;

        return a2 / Mathf.Max(denom, 0.0001f);
    }

    // Schlick-GGX Geometry sub-function
    static float GeometrySchlickGGX(float NdotX, float roughness)
    {
        float r = roughness + 1f;
        float k = (r * r) / 8f;
        return NdotX / (NdotX * (1f - k) + k);
    }

    // Smith's Geometry function (combines view + light masking)
    static float GeometrySmith(Vector3 N, Vector3 V, Vector3 L, float roughness)
    {
        float NdotV = Mathf.Max(Vector3.Dot(N, V), 0f);
        float NdotL = Mathf.Max(Vector3.Dot(N, L), 0f);
        return GeometrySchlickGGX(NdotV, roughness) * GeometrySchlickGGX(NdotL, roughness);
    }

    // Fresnel-Schlick approximation
    static Vector3 FresnelSchlick(float cosTheta, Vector3 F0)
    {
        float t = Mathf.Pow(1f - cosTheta, 5f);
        return F0 + (Vector3.one - F0) * t;
    }

    public static Color ComputePBR(Vector3 N, Vector3 V, Vector3 L,
                                     Color albedo, float metallic, float roughness)
    {
        Vector3 H = (V + L).normalized;
        Vector3 F0 = Vector3.Lerp(new Vector3(0.04f, 0.04f, 0.04f),
            new Vector3(albedo.r, albedo.g, albedo.b), metallic);

        float D = DistributionGGX(N, H, roughness);
        float G = GeometrySmith(N, V, L, roughness);
        Vector3 F = FresnelSchlick(Mathf.Max(Vector3.Dot(H, V), 0f), F0);

        // Specular BRDF
        float NdotL = Mathf.Max(Vector3.Dot(N, L), 0f);
        float NdotV = Mathf.Max(Vector3.Dot(N, V), 0f);
        float denom = 4f * NdotV * NdotL + 0.0001f;
        Vector3 specular = (D * G) * F / denom;

        // Diffuse (metals have no diffuse)
        Vector3 kD = (Vector3.one - F) * (1f - metallic);
        Vector3 albedoVec = new Vector3(albedo.r, albedo.g, albedo.b);
        Vector3 diffuse = kD * albedoVec / Mathf.PI;

        Vector3 result = (diffuse + specular) * NdotL;
        return new Color(result.x, result.y, result.z, 1f);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What are the three terms of the Cook-Torrance specular BRDF? :: **D** (Normal Distribution Function) — how many microfacets align with the half-vector. **F** (Fresnel) — how much light reflects vs. refracts at the surface. **G** (Geometry) — how much microfacet self-shadowing occurs.
- What does the Fresnel effect look like in everyday life? :: At steep/grazing angles, surfaces become more reflective. A lake looks transparent looking straight down but reflects the sky at the horizon. Road surfaces appear to "mirror" at far distances.
- Why do metals have no diffuse component in PBR? :: Metals absorb all refracted light immediately (free electrons absorb photons). Only surface reflection remains, which is the specular term with colored $F_0$ = albedo. Dielectrics refract light that scatters internally and exits as diffuse color.
- What does the roughness parameter physically represent? :: It describes the variance of microfacet surface orientations. Low roughness = microfacets are mostly aligned with the macro surface = mirror-like reflections. High roughness = chaotic orientations = scattered, blurry reflections.
- Why is energy conservation important in PBR? :: Without it, a surface could reflect more light than it receives, looking unnaturally glowing. PBR enforces $k_d + k_s \leq 1$, and the Fresnel term naturally partitions energy between specular reflection and diffuse refraction.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Setting roughness to exactly 0.0, causing the GGX denominator to approach zero and producing firefly artifacts (infinitely bright specular pixels).
> - **The Fix:** Clamp roughness to a minimum value (e.g., 0.04) and add a small epsilon to denominators: `denom = 4 * NdotV * NdotL + 0.0001`.
> - **Why:** At roughness = 0, the NDF becomes a Dirac delta function — mathematically valid but numerically unstable in discrete pixel sampling.

> [!danger] **Watch Out!**
> - **The Mistake:** Performing PBR lighting calculations in gamma/sRGB space instead of linear space.
> - **The Fix:** Always convert textures to linear space before lighting, then apply gamma correction (or tone mapping) as a final post-process.
> - **Why:** PBR equations assume linear light addition. In gamma space, light values are non-linearly compressed, making addition physically incorrect — dark surfaces appear washed out and highlights look too harsh.

---

## Related Topics
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
- [[Math/03_Vectors/reflection_refraction|Reflection & Refraction Vectors]]
- [[Math/11_Graphics_Math/color_math_hdr|Color Math & HDR]]
