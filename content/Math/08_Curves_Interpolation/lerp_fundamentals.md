---
title: "Lerp Fundamentals — The Smoothest Shortcut Between Two Values"
tags:
  - math
  - level/Lv2
  - category/curves_interpolation
---

# Lerp Fundamentals: The Smoothest Shortcut Between Two Values

> [!abstract] **The Concept in a Nutshell**
> Linear interpolation (lerp) blends between two values using a parameter $t \in [0, 1]$. When $t = 0$ you get the start value, when $t = 1$ you get the end value, and anything in between gives you a proportional mix. It is the single most-used math operation in game development — powering health bars, camera follow, color fades, and countless smooth transitions.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Damage Feedback Health Bar**
> Your warrior Kael takes a 40 HP hit. Instead of the health bar instantly snapping from 100 to 60, you smoothly animate it over 0.4 seconds. Each frame, you compute `currentHP = lerp(100, 60, elapsed / duration)`. At $t = 0.25$ the bar reads $100 + 0.25(60 - 100) = 90$; at $t = 0.5$ it shows $80$; at $t = 1.0$ it settles on $60$. The same technique drives your hit-flash color transitioning from white back to the character's original tint: `color = lerp(Color.white, originalColor, t)`.

---

## The Blueprint (Formula & Structure)

### Core Formula

$$\text{lerp}(a,\, b,\, t) = a + t \cdot (b - a), \quad t \in [0, 1]$$

An equivalent form that emphasizes the *blend* interpretation:

$$\text{lerp}(a,\, b,\, t) = (1 - t) \cdot a + t \cdot b$$

### Key Properties

| Property | Detail |
|---|---|
| **Boundary values** | $\text{lerp}(a, b, 0) = a$, $\quad \text{lerp}(a, b, 1) = b$ |
| **Midpoint** | $\text{lerp}(a, b, 0.5) = \frac{a + b}{2}$ |
| **Linearity** | The result changes at a constant rate as $t$ increases |
| **Commutativity** | $\text{lerp}(a, b, t) = \text{lerp}(b, a, 1 - t)$ |
| **Works on vectors** | Apply component-wise: $\text{lerp}(\vec{a}, \vec{b}, t) = \vec{a} + t(\vec{b} - \vec{a})$ |

### Clamped vs Unclamped $t$

- **Clamped lerp**: $t$ is restricted to $[0, 1]$. This is safe and predictable — you never overshoot.
- **Unclamped lerp**: $t$ can be any real number. With $t = 1.5$, the result *extrapolates* beyond $b$. Useful for overshooting effects (bouncy UI), but requires care.

$$t < 0 \implies \text{result is before } a \qquad t > 1 \implies \text{result is beyond } b$$

### Inverse Lerp

Given a value $v$ between $a$ and $b$, find the $t$ that would produce it:

$$\text{inverseLerp}(a,\, b,\, v) = \frac{v - a}{b - a}$$

This is invaluable for **remapping ranges**. For example, convert a health value in $[0, 100]$ to a fill amount in $[0, 1]$.

### Remap (Combining Lerp and Inverse Lerp)

To map a value from one range to another:

$$\text{remap}(v,\, a_1, b_1,\, a_2, b_2) = \text{lerp}\!\big(a_2,\, b_2,\, \text{inverseLerp}(a_1, b_1, v)\big)$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Number Line Slider**
> Imagine a slider on a ruler marked $a$ on the left and $b$ on the right. The parameter $t$ is the slider's position as a percentage: 0 % = all the way left ($a$), 100 % = all the way right ($b$), and 50 % = dead center. Lerp simply reads the number off the ruler at whatever position the slider is at. This works identically whether the "ruler" measures positions, colors, volumes, or any other numeric quantity.

> [!tip] **Mental Model: Mixing Paint**
> Think of lerp as mixing two paint colors. $t$ is the ratio: $t = 0$ is pure color A, $t = 1$ is pure color B, and $t = 0.3$ is a mix of 70 % A and 30 % B. The formula $(1-t) \cdot a + t \cdot b$ makes this explicit — the two weights always sum to 1.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Smooth health bar, camera follow, and color fade using Lerp
using UnityEngine;
using UnityEngine.UI;

public class LerpExamples : MonoBehaviour
{
    [Header("Health Bar")]
    public Image healthFill;
    private float displayedHP = 100f;
    private float actualHP = 100f;
    public float lerpSpeed = 5f;

    [Header("Camera Follow")]
    public Transform target;
    public float followSmooth = 0.1f;

    [Header("Color Fade")]
    public SpriteRenderer sprite;
    private Color hitColor = Color.white;
    private Color normalColor = Color.green;
    private float colorT = 1f;

    void Update()
    {
        // --- Smooth Health Bar (lerp toward target each frame) ---
        displayedHP = Mathf.Lerp(displayedHP, actualHP, Time.deltaTime * lerpSpeed);
        healthFill.fillAmount = Mathf.InverseLerp(0f, 100f, displayedHP);

        // --- Camera Follow (smooth damp-like behavior) ---
        transform.position = Vector3.Lerp(
            transform.position,
            target.position + Vector3.back * 10f,
            followSmooth
        );

        // --- Color Flash Recovery ---
        colorT = Mathf.Clamp01(colorT + Time.deltaTime * 3f);
        sprite.color = Color.Lerp(hitColor, normalColor, colorT);
    }

    public void TakeDamage(float amount)
    {
        actualHP = Mathf.Clamp(actualHP - amount, 0f, 100f);
        colorT = 0f; // Reset flash
    }

    // --- Remap utility ---
    public static float Remap(float value, float fromMin, float fromMax, float toMin, float toMax)
    {
        float t = Mathf.InverseLerp(fromMin, fromMax, value);
        return Mathf.Lerp(toMin, toMax, t);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the lerp formula? :: $\text{lerp}(a, b, t) = a + t(b - a)$, equivalently $(1-t)a + tb$.
- What does lerp return when $t = 0.5$? :: The midpoint: $\frac{a + b}{2}$.
- What is inverse lerp used for? :: Given a value $v$ between $a$ and $b$, it returns the $t$ that would produce $v$: $t = \frac{v - a}{b - a}$.
- What happens when $t > 1$ in an unclamped lerp? :: The result extrapolates beyond $b$, overshooting the end value.
- Why is `Mathf.Lerp(current, target, Time.deltaTime * speed)` frame-rate dependent? :: Each frame moves a percentage of the *remaining* distance, which is exponential decay — different frame rates reach the target at different speeds.
- How do you remap a value from range $[a_1, b_1]$ to $[a_2, b_2]$? :: Compute $t = \text{inverseLerp}(a_1, b_1, v)$ then return $\text{lerp}(a_2, b_2, t)$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `Lerp(current, target, Time.deltaTime * speed)` and thinking it reaches the target in a fixed time.
> - **The Fix:** This is *exponential decay*, not linear interpolation. Each frame moves a fraction of the **remaining** gap. For true timed interpolation, track elapsed time: `t = elapsed / duration; result = Lerp(start, end, t);`.
> - **Why:** When you feed the current value back as the start, the start value changes every frame, breaking the linear relationship. The result feels smooth but is frame-rate dependent and technically never reaches the target.

> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to clamp $t$ before calling lerp, leading to extrapolation artifacts.
> - **The Fix:** Use `Mathf.Clamp01(t)` or Unity's `Mathf.Lerp` (which clamps automatically). Only use `Mathf.LerpUnclamped` when you intentionally want overshooting.
> - **Why:** Accumulated time or computed ratios can easily exceed $[0, 1]$, producing values outside the expected range.

---

## Related Topics
- [[Math/08_Curves_Interpolation/easing_functions|Easing Functions]]
- [[Math/08_Curves_Interpolation/slerp_interpolation|Slerp Interpolation]]
- [[Math/01_Algebra_Foundations/algebra_fundamentals|Algebra Fundamentals]]
