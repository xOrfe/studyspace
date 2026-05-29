---
title: "Reflection & Refraction Vectors: Bouncing Light & Bending Rays"
tags:
  - math
  - level/Lv3
  - category/vectors
---

# Reflection & Refraction Vectors: Bouncing Light & Bending Rays

> [!abstract] **The Concept in a Nutshell**
> **Reflection** computes the direction a ray bounces off a surface, while **refraction** computes how it bends passing through a material boundary. These operations power everything from billiard ball bouncing to realistic water rendering. The reflection formula is elegantly simple; refraction introduces Snell's law and the fascinating phenomenon of total internal reflection.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Laser Puzzle Game & Underwater Rendering**
> In *Prism Break*, the player redirects laser beams by placing mirrors. A red laser travels in direction $\vec{d} = (1, -1, 0)$ (normalized: $(0.707, -0.707, 0)$) and strikes a mirror with surface normal $\hat{n} = (0, 1, 0)$. The reflected direction is $\vec{r} = \vec{d} - 2(\vec{d} \cdot \hat{n})\hat{n} = (0.707, -0.707, 0) - 2(-0.707)(0, 1, 0) = (0.707, 0.707, 0)$ — the laser bounces upward at the same angle. In another level, the laser hits a glass prism. Now refraction bends the beam according to Snell's law, splitting white light into a rainbow. These same formulas drive real-time reflections on water, glass transparency, and PBR material rendering in AAA games.

---

## The Blueprint (Formula & Structure)

### Reflection Formula
Given an incoming direction $\vec{d}$ (pointing toward the surface) and surface normal $\hat{n}$ (unit vector, pointing away from the surface):

$$\vec{r} = \vec{d} - 2(\vec{d} \cdot \hat{n})\hat{n}$$

**Step-by-step breakdown:**
1. $\vec{d} \cdot \hat{n}$ — scalar projection of $\vec{d}$ onto the normal (negative if hitting the front face)
2. $2(\vec{d} \cdot \hat{n})\hat{n}$ — the "double-normal" component to subtract
3. $\vec{d} - (\ldots)$ — flip only the normal component, keep the tangential component

**Properties:**
- Angle of incidence = angle of reflection ($\theta_i = \theta_r$)
- The reflected vector has the same magnitude as $\vec{d}$
- Reflection preserves the tangential component and negates the normal component

### Snell's Law (Refraction)
When a ray passes from a medium with refractive index $n_1$ into a medium with index $n_2$:

$$n_1 \sin\theta_i = n_2 \sin\theta_r$$

| Medium | Refractive Index $n$ |
|---|---|
| Vacuum | 1.0 |
| Air | 1.0003 |
| Water | 1.33 |
| Glass | 1.5 |
| Diamond | 2.42 |

### Refraction Vector Formula
Given incident direction $\vec{d}$ (unit), normal $\hat{n}$ (unit), and ratio $\eta = n_1 / n_2$:

$$\vec{t} = \eta \vec{d} + \left(\eta (\hat{n} \cdot \vec{d}) - \sqrt{1 - \eta^2(1 - (\hat{n} \cdot \vec{d})^2)}\right)\hat{n}$$

**Simplified (using $\cos\theta_i = -\hat{n} \cdot \vec{d}$):**

$$\vec{t} = \eta \vec{d} + \left(\eta \cos\theta_i - \cos\theta_t\right)\hat{n}$$

where $\cos\theta_t = \sqrt{1 - \eta^2(1 - \cos^2\theta_i)}$

### Total Internal Reflection
When the discriminant under the square root becomes negative:

$$1 - \eta^2(1 - (\hat{n} \cdot \vec{d})^2) < 0$$

No refraction is possible — the ray reflects entirely. This occurs when:
- Going from a denser medium to a less dense one ($n_1 > n_2$)
- The angle of incidence exceeds the **critical angle**: $\theta_c = \arcsin(n_2 / n_1)$

This is why you see a mirror-like reflection looking up from underwater at a steep angle.

### Fresnel Effect
In reality, surfaces exhibit **both** reflection and refraction simultaneously. The **Fresnel equations** determine how much light is reflected vs transmitted based on the viewing angle:
- **Grazing angles** (looking nearly edge-on): mostly reflection
- **Head-on angles** (looking straight at surface): mostly transmission/refraction

A common approximation is **Schlick's approximation**:

$$F(\theta) = F_0 + (1 - F_0)(1 - \cos\theta)^5$$

where $F_0 = \left(\frac{n_1 - n_2}{n_1 + n_2}\right)^2$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Bouncing Ball and the Swimming Pool**
> **Reflection** is a rubber ball bouncing off a wall — it bounces away at the same angle it arrived, like a perfect mirror. The wall's normal acts as the "fold line." Imagine folding the incoming path over the normal — the outgoing path is the crease.
>
> **Refraction** is a straw in a glass of water — it appears to bend at the surface. Light slows down in denser materials and bends toward the normal (like a car driving from pavement onto sand: the wheel that hits sand first slows down, turning the car). The bigger the density difference, the more the bending.
>
> **Total internal reflection** is like being underwater and looking up at the surface at a steep angle — instead of seeing the sky, you see a mirror! Past the critical angle, all light bounces back.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Reflection for laser bouncing and Fresnel for water rendering
using UnityEngine;

public class ReflectionRefraction : MonoBehaviour
{
    [Header("Laser Reflection")]
    public int maxBounces = 5;
    public float maxDistance = 100f;
    public LayerMask mirrorLayer;

    [Header("Fresnel")]
    public float refractiveIndex = 1.33f; // Water

    void Update()
    {
        // === MULTI-BOUNCE LASER REFLECTION ===
        TraceLaser(transform.position, transform.forward);
    }

    void TraceLaser(Vector3 origin, Vector3 direction)
    {
        for (int i = 0; i < maxBounces; i++)
        {
            if (Physics.Raycast(origin, direction, out RaycastHit hit,
                                maxDistance, mirrorLayer))
            {
                // Draw the laser segment
                Debug.DrawLine(origin, hit.point, Color.red);

                // REFLECTION: r = d - 2(d·n)n
                direction = Vector3.Reflect(direction, hit.normal);

                // Offset origin slightly to avoid self-intersection
                origin = hit.point + hit.normal * 0.001f;
            }
            else
            {
                // No hit — draw ray to infinity and stop
                Debug.DrawRay(origin, direction * maxDistance, Color.red);
                break;
            }
        }
    }

    // === SCHLICK'S FRESNEL APPROXIMATION ===
    float SchlickFresnel(Vector3 viewDir, Vector3 normal, float n1, float n2)
    {
        float f0 = Mathf.Pow((n1 - n2) / (n1 + n2), 2f);
        float cosTheta = Mathf.Max(0f, Vector3.Dot(-viewDir, normal));
        return f0 + (1f - f0) * Mathf.Pow(1f - cosTheta, 5f);
        // Returns 0-1: how much light is reflected vs transmitted
    }

    // === REFRACTION DIRECTION ===
    bool Refract(Vector3 incident, Vector3 normal, float eta, out Vector3 refracted)
    {
        float cosI = -Vector3.Dot(incident, normal);
        float sinT2 = eta * eta * (1f - cosI * cosI);

        if (sinT2 > 1f)
        {
            refracted = Vector3.zero;
            return false; // Total internal reflection!
        }

        float cosT = Mathf.Sqrt(1f - sinT2);
        refracted = eta * incident + (eta * cosI - cosT) * normal;
        return true;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the reflection formula? :: $\vec{r} = \vec{d} - 2(\vec{d} \cdot \hat{n})\hat{n}$. Subtract twice the normal component from the incoming direction to flip it across the surface.
- What does Snell's law state? :: $n_1 \sin\theta_i = n_2 \sin\theta_r$. The product of refractive index and sine of angle is preserved across a material boundary.
- When does total internal reflection occur? :: When light travels from a denser medium to a less dense one ($n_1 > n_2$) at an angle exceeding the critical angle $\theta_c = \arcsin(n_2/n_1)$. The discriminant in the refraction formula becomes negative.
- What is the Fresnel effect and why is it important in games? :: Surfaces reflect more light at grazing angles (edge-on) and transmit more at head-on angles. It's crucial for realistic water, glass, and PBR materials. Schlick's approximation: $F(\theta) = F_0 + (1 - F_0)(1 - \cos\theta)^5$.
- What Unity method computes a reflected vector? :: `Vector3.Reflect(inDirection, normal)` — it implements $\vec{r} = \vec{d} - 2(\vec{d} \cdot \hat{n})\hat{n}$ internally.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Getting the sign convention wrong for the incident vector. Some formulas expect $\vec{d}$ pointing TOWARD the surface; others expect it pointing AWAY. Unity's `Vector3.Reflect` expects the direction pointing toward the surface.
> - **The Fix:** Be consistent. In the formula $\vec{r} = \vec{d} - 2(\vec{d} \cdot \hat{n})\hat{n}$, $\vec{d}$ points toward the surface, so $\vec{d} \cdot \hat{n} < 0$. If your dot product is positive, either the direction or the normal is flipped.
> - **Why:** Mixing conventions produces reflections that go through the surface instead of bouncing off, or rays that refract in the wrong direction — subtle bugs that are hard to track down visually.

---

## Related Topics
- [[Math/03_Vectors/dot_product|Dot Product]]
- [[Math/11_Graphics_Math/ray_tracing_math|Ray Tracing Mathematics]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
