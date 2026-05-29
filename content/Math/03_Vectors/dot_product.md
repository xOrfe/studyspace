---
title: "Dot Product: The Alignment Detector"
tags:
  - math
  - level/Lv2
  - category/vectors
---

# Dot Product: The Alignment Detector

> [!abstract] **The Concept in a Nutshell**
> The **dot product** (or scalar product) takes two vectors and returns a single number that measures how much they point in the same direction. It's arguably the most versatile operation in game math — used for angle checks, field-of-view detection, surface lighting, projection, and dozens of other applications. If vectors are the alphabet of game math, the dot product is the most common word.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Stealth Game Enemy FOV Detection**
> In *Shadow Protocol*, enemy guard Hank faces direction $\hat{f} = (0, 0, 1)$ (forward). The spy Maya is at position $(3, 0, 8)$ and Hank is at $(0, 0, 5)$. The direction from Hank to Maya is $\vec{d} = (3, 0, 3)$, normalized to $\hat{d} = (0.707, 0, 0.707)$. The dot product $\hat{f} \cdot \hat{d} = 0 \cdot 0.707 + 0 \cdot 0 + 1 \cdot 0.707 = 0.707$. Since $\cos(45°) = 0.707$, Maya is 45° off Hank's forward axis. If Hank's FOV cone is 60° (half-angle), then $\cos(60°) = 0.5$, and since $0.707 > 0.5$, **Maya is detected!** This single dot product replaces expensive trigonometry calls every frame.

---

## The Blueprint (Formula & Structure)

### Algebraic Definition
$$\vec{a} \cdot \vec{b} = a_x b_x + a_y b_y + a_z b_z$$

Multiply corresponding components, then sum. The result is a **scalar** (a single number), not a vector.

### Geometric Definition
$$\vec{a} \cdot \vec{b} = \|\vec{a}\| \, \|\vec{b}\| \cos\theta$$

Where $\theta$ is the angle between the two vectors.

### Angle Between Two Vectors
Rearranging the geometric formula:

$$\cos\theta = \frac{\vec{a} \cdot \vec{b}}{\|\vec{a}\| \, \|\vec{b}\|}$$

If both vectors are **unit vectors** (normalized), this simplifies beautifully:

$$\cos\theta = \hat{a} \cdot \hat{b}$$

### Sign Meaning — The Alignment Test
| Dot Product Value | Angle $\theta$ | Meaning |
|---|---|---|
| $\hat{a} \cdot \hat{b} = 1$ | $0°$ | Perfectly aligned (same direction) |
| $\hat{a} \cdot \hat{b} > 0$ | $0° < \theta < 90°$ | Mostly facing the same way |
| $\hat{a} \cdot \hat{b} = 0$ | $90°$ | **Perpendicular** (orthogonal) |
| $\hat{a} \cdot \hat{b} < 0$ | $90° < \theta < 180°$ | Mostly facing opposite ways |
| $\hat{a} \cdot \hat{b} = -1$ | $180°$ | Perfectly opposed (opposite direction) |

### Scalar Projection
The scalar projection of $\vec{a}$ onto $\vec{b}$ tells you how much of $\vec{a}$ lies along $\vec{b}$:

$$\text{comp}_{\vec{b}} \vec{a} = \frac{\vec{a} \cdot \vec{b}}{\|\vec{b}\|}$$

If $\vec{b}$ is a unit vector: $\text{comp}_{\hat{b}} \vec{a} = \vec{a} \cdot \hat{b}$

### Key Properties
- **Commutative:** $\vec{a} \cdot \vec{b} = \vec{b} \cdot \vec{a}$
- **Distributive:** $\vec{a} \cdot (\vec{b} + \vec{c}) = \vec{a} \cdot \vec{b} + \vec{a} \cdot \vec{c}$
- **Self-dot:** $\vec{a} \cdot \vec{a} = \|\vec{a}\|^2$ (squared magnitude!)

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Shadow on a Wall**
> Imagine shining a flashlight straight down a line (vector $\vec{b}$). The **dot product** is like measuring the length of the **shadow** that vector $\vec{a}$ casts on that line. If $\vec{a}$ is parallel to the line, the shadow is at maximum length (dot = full magnitude). If $\vec{a}$ is perpendicular, the shadow has zero length (dot = 0). If $\vec{a}$ points away from the light, the shadow would fall on the "negative" side (dot < 0). The sign tells you which side of the line the shadow falls on.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — FOV detection and Lambert lighting using the dot product
using UnityEngine;

public class DotProductDemo : MonoBehaviour
{
    [Header("FOV Detection")]
    public Transform target;
    public float fovAngle = 60f; // Half-angle of the FOV cone

    [Header("Lighting")]
    public Transform lightSource;

    void Update()
    {
        // === ENEMY FOV DETECTION ===
        Vector3 forward = transform.forward; // Already a unit vector
        Vector3 toTarget = (target.position - transform.position).normalized;

        float dot = Vector3.Dot(forward, toTarget);
        float fovThreshold = Mathf.Cos(fovAngle * Mathf.Deg2Rad);

        if (dot > fovThreshold)
        {
            Debug.Log("Target is within field of view!");
            Debug.DrawRay(transform.position, toTarget * 10f, Color.red);
        }

        // === LAMBERT'S COSINE LAW (simplified surface lighting) ===
        Vector3 surfaceNormal = transform.up; // Normal of a surface
        Vector3 toLightDir = (lightSource.position - transform.position).normalized;

        float lightIntensity = Mathf.Max(0f, Vector3.Dot(surfaceNormal, toLightDir));
        // lightIntensity: 1.0 = fully lit, 0.0 = in shadow

        // === FACING CHECK: Is the target in front or behind? ===
        bool isInFront = dot > 0f;
        bool isBehind  = dot < 0f;
        bool isSideways = Mathf.Abs(dot) < 0.01f; // Nearly perpendicular
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does a dot product of 0 between two vectors mean? :: The vectors are **perpendicular** (orthogonal) — at exactly 90° to each other.
- How do you check if an enemy is within a 60° FOV cone using dot product? :: Compute $\hat{f} \cdot \hat{d}$ (forward dot direction-to-target, both normalized). If the result is greater than $\cos(60°) \approx 0.5$, the target is within the cone.
- What is $\vec{a} \cdot \vec{a}$? :: The squared magnitude $\|\vec{a}\|^2$. This is a useful identity: the self-dot product equals the length squared.
- What is Lambert's cosine law? :: Surface brightness = $\max(0, \hat{n} \cdot \hat{l})$ where $\hat{n}$ is the surface normal and $\hat{l}$ is the direction to the light. Surfaces facing the light are bright; surfaces turned away receive no light.
- Is the dot product commutative? :: Yes. $\vec{a} \cdot \vec{b} = \vec{b} \cdot \vec{a}$. Unlike the cross product, order doesn't matter.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using the dot product to find angles without normalizing first. If $\vec{a}$ and $\vec{b}$ aren't unit vectors, $\vec{a} \cdot \vec{b}$ does NOT directly equal $\cos\theta$.
> - **The Fix:** Always normalize both vectors before using the dot product for angle checks: `Vector3.Dot(a.normalized, b.normalized)`. Or divide by both magnitudes manually.
> - **Why:** The full formula is $\vec{a} \cdot \vec{b} = \|\vec{a}\|\|\vec{b}\|\cos\theta$. The magnitudes scale the result, so a raw dot product of 50 could mean "parallel short vectors" or "nearly perpendicular long vectors."

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/03_Vectors/vector_projection|Vector Projection]]
- [[Math/11_Graphics_Math/lighting_models|Lighting Models]]
