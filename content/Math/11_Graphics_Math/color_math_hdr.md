---
title: "Color Math & HDR: Beyond 0 to 255"
tags:
  - math
  - level/Lv3
  - category/graphics_math
---

# Color Math & HDR: Beyond 0 to 255

> [!abstract] **The Concept in a Nutshell**
> Colors in games aren't just RGB values — they exist in specific **color spaces** with different mathematical properties. Understanding the difference between **linear** and **gamma (sRGB)** space is essential for correct lighting. **HDR** (High Dynamic Range) rendering allows light intensities far beyond the 0–1 range, enabling effects like bloom and realistic exposure. **Tone mapping** compresses HDR values back into the displayable range without losing visual quality.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A First-Person Shooter — Exiting a Dark Bunker into Bright Sunlight**
> The player runs through a dim bunker (light intensity ~0.1) and bursts outside where the sun blazes at intensity 50.0. In LDR (Low Dynamic Range), the sun would clamp to 1.0 — the sky looks the same brightness as a white wall. With **HDR rendering**, the sun is stored at 50.0 in the framebuffer. A **bloom post-process** detects pixels above 1.0 and adds a glow. **Tone mapping** (ACES) then compresses the full 0–50 range into 0–1 for the monitor, preserving the *relative* brightness relationships — the sun looks blindingly bright while the bunker wall stays dark. Meanwhile, if the artist forgot to set their albedo textures to **sRGB** mode, all the concrete textures look washed out and the metal looks too bright — because the lighting math was performed in gamma space instead of linear, double-applying the gamma curve.

---

## The Blueprint (Formula & Structure)

### Linear vs. Gamma (sRGB) Color Space

Human eyes perceive brightness **non-linearly** — we're more sensitive to differences in dark values than bright values. The sRGB standard exploits this:

**sRGB encoding (linear → gamma):**

$$C_{\text{sRGB}} = \begin{cases} 12.92 \cdot C_{\text{linear}} & \text{if } C_{\text{linear}} \leq 0.0031308 \\ 1.055 \cdot C_{\text{linear}}^{1/2.4} - 0.055 & \text{if } C_{\text{linear}} > 0.0031308 \end{cases}$$

**Simplified approximation:**
$$C_{\text{sRGB}} \approx C_{\text{linear}}^{1/2.2}$$

**sRGB decoding (gamma → linear):**
$$C_{\text{linear}} \approx C_{\text{sRGB}}^{2.2}$$

### Why Linear Space Matters for Lighting

Lighting equations assume **additive, linear** light behavior:

$$L_{\text{total}} = L_1 + L_2 \quad \text{(only correct in linear space)}$$

If you add two lights in gamma space: $0.5^{2.2} + 0.5^{2.2} = 0.217 + 0.217 = 0.434$, but decoding back gives $0.434^{1/2.2} = 0.684$ — **wrong**! In linear space: $0.5 + 0.5 = 1.0$ — correct.

**The correct workflow:**
1. **Decode** sRGB textures to linear space on load ($C^{2.2}$)
2. Perform ALL lighting/blending in **linear space**
3. **Encode** the final framebuffer back to sRGB ($C^{1/2.2}$) for display

### HDR — High Dynamic Range

Real-world luminance varies enormously:

| Scene | Luminance (cd/m²) |
|---|---|
| Starlight | 0.001 |
| Indoor lighting | 100 |
| Overcast sky | 1,000 |
| Direct sunlight | 100,000 |

HDR rendering uses **floating-point framebuffers** (RGBA16F or RGBA32F) that store values beyond $[0, 1]$:
- Sun: 50.0
- Sky: 3.0
- Indoor lamp: 1.5
- Shadow: 0.05

This preserves relative brightness relationships that LDR (8-bit per channel) cannot represent.

### Tone Mapping Operators

Tone mapping compresses HDR → LDR for display while preserving perceptual contrast.

**Reinhard (simple):**
$$C_{\text{out}} = \frac{C_{\text{HDR}}}{1 + C_{\text{HDR}}}$$

Properties: maps $[0, \infty) \to [0, 1)$, never clips, but can look flat/washed out.

**Reinhard (extended, with white point):**
$$C_{\text{out}} = \frac{C_{\text{HDR}} \cdot (1 + C_{\text{HDR}} / W^2)}{1 + C_{\text{HDR}}}$$

Where $W$ = the smallest luminance that maps to pure white.

**ACES Filmic (industry standard):**
$$C_{\text{out}} = \frac{C(aC + b)}{C(cC + d) + e}$$

With fitted constants: $a = 2.51$, $b = 0.03$, $c = 2.43$, $d = 0.59$, $e = 0.14$.

$$C_{\text{ACES}} = \frac{C(2.51C + 0.03)}{C(2.43C + 0.59) + 0.14}$$

ACES provides a cinematic look with a natural shoulder (bright values compress gracefully) and toe (dark values maintain contrast).

### Bloom from HDR

Bloom detects **super-bright** fragments (above a threshold, e.g., $> 1.0$) and adds a Gaussian-blurred glow:

1. Extract bright pixels: $C_{\text{bright}} = \max(0, C_{\text{HDR}} - \text{threshold})$
2. Downsample and blur (multi-pass Gaussian)
3. Add back to the original image: $C_{\text{final}} = C_{\text{HDR}} + C_{\text{bloom}}$

This only works with HDR — in LDR, there's no information above 1.0 to extract.

### Color Blending Operations

$$C_{\text{multiply}} = C_A \times C_B$$
$$C_{\text{screen}} = 1 - (1 - C_A)(1 - C_B)$$
$$C_{\text{overlay}} = \begin{cases} 2 \cdot C_A \cdot C_B & \text{if } C_A < 0.5 \\ 1 - 2(1-C_A)(1-C_B) & \text{otherwise} \end{cases}$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Camera Exposure Analogy**
> Think of your game's rendering pipeline as a camera. **Linear space** is how much actual light hits the sensor — physical, measurable, additive. **Gamma space** is how the photo is saved to JPEG — compressed so dark areas get more data bits (because your eyes care more about shadows than highlights). If you try to do math on the JPEG values directly (adding two photos), the result is wrong because the values aren't real light anymore — they're perceptually compressed. **HDR** is like shooting in RAW format — you capture the full dynamic range, from pitch black to blinding sun. **Tone mapping** is the "develop" step where you choose how to map that enormous range into a printable photo. ACES tone mapping is like using a professional film stock that handles highlights and shadows beautifully.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Color space conversions and tone mapping
using UnityEngine;

public class ColorMathDemo : MonoBehaviour
{
    // Convert sRGB [0,1] to Linear space
    static float SRGBToLinear(float srgb)
    {
        if (srgb <= 0.04045f)
            return srgb / 12.92f;
        return Mathf.Pow((srgb + 0.055f) / 1.055f, 2.4f);
    }

    // Convert Linear to sRGB [0,1]
    static float LinearToSRGB(float linear)
    {
        if (linear <= 0.0031308f)
            return 12.92f * linear;
        return 1.055f * Mathf.Pow(linear, 1f / 2.4f) - 0.055f;
    }

    // Reinhard tone mapping
    static Color ToneMapReinhard(Color hdr)
    {
        return new Color(
            hdr.r / (1f + hdr.r),
            hdr.g / (1f + hdr.g),
            hdr.b / (1f + hdr.b),
            hdr.a
        );
    }

    // ACES Filmic tone mapping
    static Color ToneMapACES(Color hdr)
    {
        float a = 2.51f, b = 0.03f, c = 2.43f, d = 0.59f, e = 0.14f;
        return new Color(
            Mathf.Clamp01((hdr.r * (a * hdr.r + b)) / (hdr.r * (c * hdr.r + d) + e)),
            Mathf.Clamp01((hdr.g * (a * hdr.g + b)) / (hdr.g * (c * hdr.g + d) + e)),
            Mathf.Clamp01((hdr.b * (a * hdr.b + b)) / (hdr.b * (c * hdr.b + d) + e)),
            hdr.a
        );
    }

    void Start()
    {
        // Demonstrate why linear space matters
        Color srgbGray = new Color(0.5f, 0.5f, 0.5f); // 50% gray in sRGB
        float linearValue = SRGBToLinear(0.5f);
        Debug.Log($"sRGB 0.5 → Linear: {linearValue:F3} (not 0.5!)");

        // HDR sun color (way above 1.0)
        Color hdrSun = new Color(50f, 45f, 40f);

        // Tone map to displayable range
        Color reinhard = ToneMapReinhard(hdrSun);
        Color aces = ToneMapACES(hdrSun);

        Debug.Log($"HDR Sun: ({hdrSun.r}, {hdrSun.g}, {hdrSun.b})");
        Debug.Log($"Reinhard: ({reinhard.r:F3}, {reinhard.g:F3}, {reinhard.b:F3})");
        Debug.Log($"ACES:     ({aces.r:F3}, {aces.g:F3}, {aces.b:F3})");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Why must lighting calculations be performed in linear space? :: Light is physically additive — two light sources of intensity 0.5 produce intensity 1.0. In gamma/sRGB space, values are non-linearly compressed, so adding them produces incorrect results (e.g., $0.5_{\gamma} + 0.5_{\gamma} \neq 1.0_{\gamma}$ in terms of actual light).
- What does the approximate sRGB gamma correction formula look like? :: Linear to sRGB: $C_{\text{sRGB}} \approx C_{\text{linear}}^{1/2.2}$. sRGB to linear: $C_{\text{linear}} \approx C_{\text{sRGB}}^{2.2}$. The exponent 2.2 approximates the precise sRGB transfer function.
- What is tone mapping and why is it needed? :: Tone mapping compresses HDR values (which can exceed 1.0, e.g., sun at 50.0) into the displayable $[0, 1]$ range. Without it, any value above 1.0 clips to white, losing all brightness differentiation.
- Why does bloom only work with HDR rendering? :: Bloom extracts pixels brighter than a threshold (e.g., > 1.0). In LDR, all values are clamped to $[0, 1]$, so there's no "super-bright" information to extract — everything at 1.0 would bloom, including regular white surfaces.
- What's the difference between Reinhard and ACES tone mapping? :: Reinhard ($C/(1+C)$) is simple but can look flat/washed out. ACES uses a filmic S-curve with a toe (preserves dark contrast) and shoulder (graceful highlight rolloff), producing a more cinematic, visually appealing result favored by the film and game industry.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Applying gamma correction twice — once in the shader and once by the display hardware, resulting in overly bright, washed-out images.
> - **The Fix:** Let the engine handle the final gamma correction. In Unity, use "Linear" color space in Player Settings (not "Gamma"). Mark color textures as sRGB and data textures (normal maps, masks) as Linear.
> - **Why:** When color space is set to "Linear," Unity automatically decodes sRGB textures on read and encodes the final framebuffer on write. Manual gamma correction on top of this doubles the correction.

> [!danger] **Watch Out!**
> - **The Mistake:** Performing color lerp/blending in sRGB space, which creates a dark "dip" in the middle of gradients.
> - **The Fix:** Convert to linear space first, blend, then convert back: `result = LinearToSRGB(lerp(SRGBToLinear(a), SRGBToLinear(b), t))`.
> - **Why:** In sRGB, the midpoint between two colors is perceptually darker than expected because the gamma curve compresses mid-tones. Linear-space blending produces physically correct gradients.

---

## Related Topics
- [[Math/11_Graphics_Math/rendering_pipeline|The Rendering Pipeline]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
- [[Math/11_Graphics_Math/pbr_cook_torrance|PBR & Cook-Torrance]]
