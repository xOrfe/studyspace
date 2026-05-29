---
title: "Ray Tracing Mathematics: Following the Light"
tags:
  - math
  - level/Lv4
  - category/graphics_math
---

# Ray Tracing Mathematics: Following the Light

> [!abstract] **The Concept in a Nutshell**
> Ray tracing simulates light transport by shooting mathematical rays from the camera through each pixel, testing for intersections with scene geometry, and recursively tracing reflected, refracted, and shadow rays to compute physically accurate lighting. With Monte Carlo integration and techniques like Russian roulette termination, ray tracing can solve the full rendering equation — producing photorealistic global illumination, reflections, and soft shadows that rasterization can only approximate.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Cyberpunk City — RTX Reflections in Rain-Soaked Streets**
> Neon signs reflect off wet asphalt. Each puddle shows a distorted image of the buildings above, with the reflection blurring based on the water's roughness. A glass storefront shows both the interior (refraction) and a faint reflection of the player. Traditional rasterization would use screen-space reflections (limited to what's already on screen) and cubemaps (static, pre-baked). **Ray tracing** fires a reflection ray from each wet pixel, correctly intersecting off-screen geometry. The glass storefront traces both a reflection ray AND a refraction ray, splitting energy via the Fresnel equation. Shadow rays from each point toward area lights produce naturally soft shadows under the neon signs. This is now real-time on RTX GPUs — the math hasn't changed since 1980, but the hardware finally caught up.

---

## The Blueprint (Formula & Structure)

### Ray Definition

A ray is defined parametrically:

$$\vec{r}(t) = \vec{O} + t \cdot \vec{D}, \quad t \geq 0$$

Where:
- $\vec{O}$ = ray origin (camera position or surface point)
- $\vec{D}$ = ray direction (unit vector)
- $t$ = parameter (distance along the ray)

### Primary Ray Generation

For each pixel $(i, j)$ on a screen of size $(W, H)$ with vertical FOV $\theta$:

$$\text{aspect} = W / H$$
$$\text{scale} = \tan(\theta / 2)$$

$$\vec{D} = \text{normalize}\left(\begin{bmatrix} (2 \cdot \frac{i + 0.5}{W} - 1) \cdot \text{aspect} \cdot \text{scale} \\ (1 - 2 \cdot \frac{j + 0.5}{H}) \cdot \text{scale} \\ -1 \end{bmatrix}\right)$$

Transform $\vec{D}$ by the camera's rotation matrix to get world-space direction.

### Ray-Sphere Intersection

For a sphere centered at $\vec{C}$ with radius $r$:

$$|\vec{r}(t) - \vec{C}|^2 = r^2$$

Expand:
$$|\vec{D}|^2 t^2 + 2(\vec{O} - \vec{C}) \cdot \vec{D} \cdot t + |\vec{O} - \vec{C}|^2 - r^2 = 0$$

With $\vec{d} = \vec{O} - \vec{C}$:
- $a = \vec{D} \cdot \vec{D} = 1$ (if normalized)
- $b = 2(\vec{d} \cdot \vec{D})$
- $c = \vec{d} \cdot \vec{d} - r^2$

$$\Delta = b^2 - 4ac$$

- $\Delta < 0$: no intersection (ray misses)
- $\Delta = 0$: tangent hit (one point)
- $\Delta > 0$: two hits, take smallest positive $t$:

$$t = \frac{-b - \sqrt{\Delta}}{2a}$$

### Ray-Triangle Intersection (Möller-Trumbore)

For a triangle with vertices $V_0, V_1, V_2$:

$$\vec{E_1} = V_1 - V_0, \quad \vec{E_2} = V_2 - V_0, \quad \vec{T} = \vec{O} - V_0$$

$$\vec{P} = \vec{D} \times \vec{E_2}, \quad \vec{Q} = \vec{T} \times \vec{E_1}$$

$$\det = \vec{P} \cdot \vec{E_1}$$

$$t = \frac{\vec{Q} \cdot \vec{E_2}}{\det}, \quad u = \frac{\vec{P} \cdot \vec{T}}{\det}, \quad v = \frac{\vec{Q} \cdot \vec{D}}{\det}$$

Hit condition: $u \geq 0, \quad v \geq 0, \quad u + v \leq 1, \quad t > 0$

### Recursive Ray Tracing

At each hit point, spawn secondary rays:

**Reflection ray:**
$$\vec{D}_{\text{reflect}} = \vec{D} - 2(\vec{D} \cdot \vec{N})\vec{N}$$

**Refraction ray (Snell's law):**
$$\vec{D}_{\text{refract}} = \frac{\eta_1}{\eta_2}\vec{D} + \left(\frac{\eta_1}{\eta_2}\cos\theta_i - \cos\theta_t\right)\vec{N}$$

Where $\cos\theta_t = \sqrt{1 - \left(\frac{\eta_1}{\eta_2}\right)^2(1 - \cos^2\theta_i)}$

**Shadow ray:** Cast from the hit point toward each light source. If any geometry intersects the ray before reaching the light → the point is in shadow.

### Color Accumulation

$$C_{\text{pixel}} = C_{\text{local}} + k_r \cdot C_{\text{reflect}} + k_t \cdot C_{\text{refract}}$$

Where $k_r$ and $k_t$ come from Fresnel equations, and $C_{\text{reflect}}, C_{\text{refract}}$ are recursively traced.

### Monte Carlo Integration

For indirect illumination, sample random directions over the hemisphere:

$$L_o \approx \frac{1}{N} \sum_{i=1}^{N} \frac{f_r(\vec{l}_i, \vec{v}) \cdot L_i(\vec{l}_i) \cdot (\vec{N} \cdot \vec{l}_i)}{p(\vec{l}_i)}$$

Where $p(\vec{l}_i)$ is the probability density of sampling direction $\vec{l}_i$.

**Cosine-weighted hemisphere sampling** ($p(\vec{l}) = \frac{\cos\theta}{\pi}$) reduces variance by sampling directions proportional to the Lambert cosine factor.

### Russian Roulette Termination

Instead of capping recursion at a fixed depth, probabilistically terminate rays:

- At each bounce, continue with probability $p$ (e.g., $p$ = surface albedo magnitude)
- If continuing, scale the result by $1/p$ to maintain an unbiased estimator
- This allows arbitrary bounce depth while keeping computation finite in expectation

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Laser Pointer Investigation**
> Imagine standing in a room and pointing a laser through a tiny hole in a card (one pixel). The laser hits a wall — that's your **primary ray intersection**. You note the wall's color. Then you bounce the laser off the wall (reflection) — if it hits a mirror, you trace the bounce. You also check: "can I draw a straight line from this wall spot to the light bulb without hitting anything?" That's a **shadow ray**. For diffuse surfaces, you'd need to shoot lasers in thousands of random directions to see how light arrives from everywhere (Monte Carlo). Russian roulette is like saying "with each bounce, flip a coin — heads I keep tracing, tails I stop and scale up my answer." On average, this gives the right result without infinite computation.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simple ray tracer demonstrating sphere intersection and reflection
using UnityEngine;

public class SimpleRayTracer : MonoBehaviour
{
    [SerializeField] private Transform sphereTarget;
    [SerializeField] private float sphereRadius = 1f;
    [SerializeField] private Light sceneLight;
    [SerializeField] private int maxBounces = 3;

    // Ray-sphere intersection — returns t or -1 if no hit
    float RaySphereIntersect(Vector3 origin, Vector3 dir, Vector3 center, float radius)
    {
        Vector3 oc = origin - center;
        float b = 2f * Vector3.Dot(oc, dir);
        float c = Vector3.Dot(oc, oc) - radius * radius;
        float discriminant = b * b - 4f * c;

        if (discriminant < 0) return -1f;

        float t = (-b - Mathf.Sqrt(discriminant)) / 2f;
        return t > 0.001f ? t : -1f; // small epsilon to avoid self-intersection
    }

    // Trace a ray recursively
    Color TraceRay(Vector3 origin, Vector3 direction, int depth)
    {
        if (depth >= maxBounces) return Color.black;

        float t = RaySphereIntersect(origin, direction, sphereTarget.position, sphereRadius);
        if (t < 0) return new Color(0.1f, 0.1f, 0.2f); // sky color (miss)

        // Hit point and normal
        Vector3 hitPoint = origin + direction * t;
        Vector3 normal = (hitPoint - sphereTarget.position).normalized;

        // Shadow ray — check if light is visible
        Vector3 toLightDir = (sceneLight.transform.position - hitPoint).normalized;
        float shadow = 1f; // 1 = lit, 0 = shadowed

        // Diffuse shading (Lambert)
        float diffuse = Mathf.Max(0, Vector3.Dot(normal, toLightDir)) * shadow;

        // Reflection ray (recursive)
        Vector3 reflectDir = Vector3.Reflect(direction, normal);
        Color reflectedColor = TraceRay(hitPoint + normal * 0.001f, reflectDir, depth + 1);

        // Combine: 70% diffuse + 30% reflection
        float reflectivity = 0.3f;
        Color surfaceColor = Color.red * diffuse * (1f - reflectivity);
        return surfaceColor + reflectedColor * reflectivity;
    }

    void Update()
    {
        // Cast a ray from camera through screen center
        Ray camRay = Camera.main.ScreenPointToRay(new Vector3(Screen.width / 2, Screen.height / 2));
        Color result = TraceRay(camRay.origin, camRay.direction, 0);
        Debug.Log($"Center pixel color: {result}");
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the parametric ray equation? :: $\vec{r}(t) = \vec{O} + t \cdot \vec{D}$, where $\vec{O}$ is the origin, $\vec{D}$ is the direction, and $t \geq 0$ is the distance parameter. A hit point is found by solving for $t$ where the ray intersects geometry.
- What does the discriminant tell you in a ray-sphere intersection? :: $\Delta < 0$: ray misses the sphere entirely. $\Delta = 0$: ray is tangent (grazes the surface at one point). $\Delta > 0$: ray passes through the sphere at two points; take the smallest positive $t$ for the nearest hit.
- What is a shadow ray and how does it work? :: A shadow ray is cast from a hit point toward a light source. If any geometry intersects this ray before reaching the light, the original hit point is in shadow from that light source.
- What is Russian roulette in path tracing? :: Instead of terminating rays at a fixed depth, each bounce has a random probability $p$ of continuing. If it continues, the result is scaled by $1/p$ to maintain an unbiased estimate. This allows infinite theoretical depth while keeping expected computation finite.
- Why is Monte Carlo integration used for global illumination? :: The rendering equation requires integrating incoming light over the entire hemisphere — an analytically intractable integral for complex scenes. Monte Carlo approximates it by averaging random samples: $L_o \approx \frac{1}{N}\sum \frac{f_r \cdot L_i \cdot \cos\theta}{p(\omega)}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Self-intersection — a reflected ray immediately hits the surface it originated from, causing dark spots or infinite recursion.
> - **The Fix:** Offset the reflected ray origin slightly along the normal: `origin = hitPoint + normal * 0.001f`. This is the "epsilon offset" or "shadow bias."
> - **Why:** Floating-point imprecision means the hit point may be slightly inside the surface. Without offset, the next intersection test immediately detects $t \approx 0$ against the same surface.

> [!danger] **Watch Out!**
> - **The Mistake:** Not importance-sampling — shooting rays uniformly over the hemisphere, leading to extremely noisy renders.
> - **The Fix:** Use cosine-weighted hemisphere sampling ($p(\omega) \propto \cos\theta$) for diffuse surfaces, or GGX importance sampling for specular. Match the sampling distribution to the BRDF shape.
> - **Why:** Uniform sampling wastes most rays on directions that contribute little to the integral (grazing angles for diffuse). Importance sampling concentrates rays where the integrand is large, dramatically reducing variance for the same sample count.

---

## Related Topics
- [[Math/07_Geometric_Primitives/intersection_testing|Intersection Testing]]
- [[Math/03_Vectors/reflection_refraction|Reflection & Refraction Vectors]]
- [[Math/12_Advanced_Topics/probability_statistics|Probability & Statistics]]
