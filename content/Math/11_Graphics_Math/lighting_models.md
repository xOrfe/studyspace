---
title: "Lighting Models: Painting Reality with Math"
tags:
  - math
  - level/Lv3
  - category/graphics_math
---

# Lighting Models: Painting Reality with Math

> [!abstract] **The Concept in a Nutshell**
> Lighting models are mathematical approximations of how light interacts with surfaces. By combining ambient, diffuse (Lambert), and specular (Phong/Blinn-Phong) components, we create the illusion that flat polygons are three-dimensional objects reacting to light sources — the fundamental trick that makes 3D graphics convincing.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Dungeon Crawler Torch Corridor**
> The player holds a flickering torch as they walk through a stone corridor. Without lighting math, every wall polygon would be a flat, uniformly colored rectangle — the scene would look like a colored floor plan. **Ambient light** ensures the distant corners aren't pitch black. **Lambert diffuse** makes the wall facing the torch bright while the perpendicular wall stays dark, revealing the corridor's 3D geometry. **Phong specular** adds a bright gleam on the wet stone floor where the torch reflects directly toward the camera. When the player turns, the specular highlight slides across the surface because it depends on the view direction — this single detail makes the stone look wet rather than dry.

---

## The Blueprint (Formula & Structure)

### The Complete Lighting Equation (Phong Model)

$$I = I_a + I_d + I_s = k_a \cdot L_a + k_d \cdot L_d \cdot \max(0, \vec{N} \cdot \vec{L}) + k_s \cdot L_s \cdot \max(0, \vec{R} \cdot \vec{V})^{n}$$

Where:
- $I$ = final intensity at the surface point
- $k_a, k_d, k_s$ = material ambient, diffuse, specular coefficients
- $L_a, L_d, L_s$ = light ambient, diffuse, specular colors/intensities
- $\vec{N}$ = surface normal (unit vector)
- $\vec{L}$ = direction from surface to light (unit vector)
- $\vec{V}$ = direction from surface to viewer/camera (unit vector)
- $\vec{R}$ = reflection of $-\vec{L}$ about $\vec{N}$
- $n$ = shininess exponent (higher = tighter highlight)

### Component 1: Ambient

$$I_a = k_a \cdot L_a$$

A constant base illumination that prevents surfaces facing away from all lights from being completely black. It's a crude approximation of indirect bounced light.

### Component 2: Diffuse (Lambert's Cosine Law)

$$I_d = k_d \cdot L_d \cdot \max(0, \vec{N} \cdot \vec{L})$$

The dot product $\vec{N} \cdot \vec{L} = \cos\theta$ measures how directly the surface faces the light. A surface facing the light head-on ($\theta = 0°$, $\cos\theta = 1$) receives maximum illumination. A surface parallel to the light direction ($\theta = 90°$) receives none. The $\max(0, \ldots)$ clamp prevents negative light (surfaces facing away).

**Key property:** Diffuse lighting is **view-independent** — it looks the same regardless of where the camera is.

### Component 3: Specular (Phong)

$$\vec{R} = 2(\vec{N} \cdot \vec{L})\vec{N} - \vec{L}$$

$$I_s = k_s \cdot L_s \cdot \max(0, \vec{R} \cdot \vec{V})^{n}$$

The specular highlight appears where the reflection vector $\vec{R}$ aligns with the view direction $\vec{V}$. The exponent $n$ controls the tightness:
- $n = 1$: very broad, matte-like highlight
- $n = 32$: moderately sharp plastic highlight
- $n = 256$: very tight metal/glass highlight

### Blinn-Phong Optimization

Instead of computing the reflection vector, Blinn-Phong uses the **half-vector**:

$$\vec{H} = \frac{\vec{L} + \vec{V}}{|\vec{L} + \vec{V}|}$$

$$I_s^{\text{Blinn}} = k_s \cdot L_s \cdot \max(0, \vec{N} \cdot \vec{H})^{n'}$$

**Why it's better:**
- $\vec{H}$ is cheaper to compute than $\vec{R}$
- For directional lights, $\vec{H}$ is constant across the entire surface
- Produces slightly wider, more physically plausible highlights
- $n'$ is typically $\approx 4n$ to match Phong visually

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Flashlight and the Basketball**
> Hold a flashlight and point it at a basketball. The **ambient** is the dim room light letting you see the ball's outline even in the shadow side. The **diffuse** is the smooth gradient from bright (facing the flashlight) to dark (turned away) — it depends only on the ball's surface angle relative to the flashlight. The **specular** is the bright white hotspot — it moves as you walk around the ball because it depends on your eye position relative to the reflection angle. A matte rubber ball has strong diffuse, weak specular. A glossy billiard ball has strong, tight specular. The dot product $\vec{N} \cdot \vec{L}$ is literally "how much is the surface pointing toward the light?" — a perfect, intuitive geometric measure.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Manual Blinn-Phong lighting calculation per-vertex
// (Normally done in shaders, but shown in C# for learning)
using UnityEngine;

public class BlinnPhongDemo : MonoBehaviour
{
    [SerializeField] private Light sceneLight;
    [SerializeField] private float ambientStrength = 0.1f;
    [SerializeField] private float specularStrength = 0.5f;
    [SerializeField] private float shininess = 32f;

    void Update()
    {
        Vector3 surfacePoint = transform.position;
        Vector3 surfaceNormal = transform.up; // simplified: using object's up as normal

        // Direction vectors (must be normalized!)
        Vector3 lightDir = (sceneLight.transform.position - surfacePoint).normalized;
        Vector3 viewDir = (Camera.main.transform.position - surfacePoint).normalized;

        // Ambient component
        Color ambient = ambientStrength * sceneLight.color;

        // Diffuse component (Lambert's cosine law)
        float NdotL = Mathf.Max(0f, Vector3.Dot(surfaceNormal, lightDir));
        Color diffuse = NdotL * sceneLight.color;

        // Specular component (Blinn-Phong with half-vector)
        Vector3 halfVector = (lightDir + viewDir).normalized;
        float NdotH = Mathf.Max(0f, Vector3.Dot(surfaceNormal, halfVector));
        float specAmount = Mathf.Pow(NdotH, shininess);
        Color specular = specularStrength * specAmount * sceneLight.color;

        // Final combined color
        Color finalColor = ambient + diffuse + specular;
        GetComponent<Renderer>().material.color = finalColor;

        Debug.Log($"N·L={NdotL:F2} | N·H={NdotH:F2} | Spec={specAmount:F3}");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does Lambert's cosine law compute, and what's its formula? :: It computes the diffuse light intensity based on the angle between the surface normal and light direction: $I_d = k_d \cdot \max(0, \vec{N} \cdot \vec{L})$. The cosine factor means surfaces facing the light are brightest.
- Why is diffuse lighting view-independent while specular is view-dependent? :: Diffuse depends only on $\vec{N} \cdot \vec{L}$ (surface vs. light direction). Specular depends on $\vec{R} \cdot \vec{V}$ or $\vec{N} \cdot \vec{H}$, which both involve the view/camera direction, so the highlight moves as the viewer moves.
- What is the half-vector in Blinn-Phong, and why is it preferred? :: $\vec{H} = \text{normalize}(\vec{L} + \vec{V})$. It's preferred because it avoids computing the full reflection vector, and for directional lights $\vec{H}$ is constant across the surface, saving computation.
- What does the shininess exponent $n$ control? :: It controls the tightness of the specular highlight. Low $n$ (e.g., 1-8) creates broad, matte highlights. High $n$ (e.g., 128-256) creates tight, sharp highlights typical of polished or metallic surfaces.
- Why do we clamp the dot product with $\max(0, \ldots)$? :: A negative dot product means the surface faces away from the light (angle > 90°). Negative light has no physical meaning, so we clamp to zero to prevent darkening surfaces that should simply receive no light from that source.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to normalize vectors before computing dot products in the fragment shader.
> - **The Fix:** Always normalize $\vec{N}$, $\vec{L}$, and $\vec{V}$ per-fragment. Interpolated normals from vertex data are NOT unit length after interpolation.
> - **Why:** If $|\vec{N}| \neq 1$, the dot product $\vec{N} \cdot \vec{L}$ no longer equals $\cos\theta$, producing incorrect brightness values — surfaces appear too bright or too dark with visible interpolation artifacts along triangle edges.

> [!danger] **Watch Out!**
> - **The Mistake:** Computing lighting in the wrong space — e.g., normals in world space but light direction in object space.
> - **The Fix:** Transform all vectors into the same coordinate space (world space is most common) before any dot product.
> - **Why:** The dot product only gives a meaningful geometric result when both vectors exist in the same coordinate frame.

---

## Related Topics
- [[Math/03_Vectors/dot_product|Dot Product]]
- [[Math/07_Geometric_Primitives/normal_vectors|Normal Vectors]]
- [[Math/11_Graphics_Math/pbr_cook_torrance|PBR & Cook-Torrance]]
