---
title: "Easing Functions — Making Motion Feel Alive"
tags:
  - math
  - level/Lv2
  - category/curves_interpolation
---

# Easing Functions: Making Motion Feel Alive

> [!abstract] **The Concept in a Nutshell**
> Easing functions reshape the linear parameter $t$ into a curved parameter $t'$, so that movement starts slowly and speeds up (ease-in), starts fast and slows down (ease-out), or both (ease-in-out). They are the secret behind UI animations that feel polished and game motion that feels organic rather than robotic.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Settings Panel Sliding In**
> In your mobile RPG, the player taps the gear icon. A settings panel slides in from the right. With plain lerp it slides at constant speed — functional but lifeless. Apply an **ease-out cubic** and the panel rockets in quickly then decelerates gracefully to a stop, mimicking real-world inertia. The close animation uses **ease-in quadratic** — it starts slow (giving the player a moment to register the motion) then accelerates off-screen. Jump arcs use a custom ease curve to let the character hang at the apex slightly longer, making the jump *feel* higher than it actually is.

---

## The Blueprint (Formula & Structure)

### The Core Idea

An easing function $f(t)$ takes $t \in [0, 1]$ and returns $t' \in [0, 1]$, preserving the endpoints:

$$f(0) = 0, \quad f(1) = 1$$

You then feed $t'$ into your lerp: $\text{result} = \text{lerp}(a,\, b,\, f(t))$.

### Common Easing Families

#### Power-Based Easing

| Name | Ease-In Formula | Ease-Out Formula |
|---|---|---|
| **Quadratic** | $f(t) = t^2$ | $f(t) = 1 - (1-t)^2$ |
| **Cubic** | $f(t) = t^3$ | $f(t) = 1 - (1-t)^3$ |
| **Quartic** | $f(t) = t^4$ | $f(t) = 1 - (1-t)^4$ |
| **Quintic** | $f(t) = t^5$ | $f(t) = 1 - (1-t)^5$ |

#### The Ease-Out Trick

For *any* ease-in function $f_{\text{in}}(t)$, you get the ease-out version by flipping both axes:

$$f_{\text{out}}(t) = 1 - f_{\text{in}}(1 - t)$$

#### Ease-In-Out (Combining Both)

Blend ease-in for the first half and ease-out for the second:

$$f_{\text{inout}}(t) = \begin{cases} \frac{f_{\text{in}}(2t)}{2} & \text{if } t < 0.5 \\[6pt] 1 - \frac{f_{\text{in}}(2(1-t))}{2} & \text{if } t \geq 0.5 \end{cases}$$

### Smoothstep

A classic cubic hermite ease-in-out built into shaders and engines:

$$\text{smoothstep}(t) = 3t^2 - 2t^3$$

Its smoother sibling, **smootherstep** (Ken Perlin):

$$\text{smootherstep}(t) = 6t^5 - 15t^4 + 10t^3$$

Key property: both have **zero first derivatives** at $t=0$ and $t=1$, giving buttery-smooth start and end.

### Exponential and Elastic Easing (Robert Penner)

$$\text{easeInExpo}(t) = 2^{10(t - 1)}$$

$$\text{easeInElastic}(t) = -2^{10(t-1)} \sin\!\Big(\frac{(t - 1.1) \cdot 2\pi}{0.4}\Big)$$

$$\text{easeOutBounce}(t) = \text{(piecewise quadratic segments simulating bounces)}$$

### Comparison Chart

| Easing | Start | Middle | End | Best For |
|---|---|---|---|---|
| Linear | constant | constant | constant | Debug, data |
| Ease-In | slow | accelerating | fast | Exit animations |
| Ease-Out | fast | decelerating | slow | Enter animations |
| Ease-In-Out | slow | fast | slow | Attention/emphasis |
| Smoothstep | smooth | S-curve | smooth | Blending, shaders |
| Elastic | overshoot | oscillate | settle | Bouncy UI, playful |

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Car on a Hill**
> Imagine driving a car up a steep hill (ease-in): you start slow due to gravity, then power through. For ease-out, picture rolling down the other side: you start fast but brakes bring you to a gentle stop. Ease-in-out is the full journey — slow climb, fast over the crest, slow descent. The steepness of the hill is controlled by the power ($t^2$ = gentle hill, $t^5$ = cliff face).

> [!tip] **Mental Model: The t → t' Remapper**
> Think of the easing function as a *warped ruler*. Regular lerp reads positions on a straight ruler. An easing function bends that ruler: scrunching marks together where motion should be slow, spreading them apart where motion should be fast. The physical distance (output) stays the same, but the *pace* along it changes.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Easing function library and UI panel animation
using UnityEngine;

public static class Easing
{
    // --- Power-based ---
    public static float InQuad(float t)  => t * t;
    public static float OutQuad(float t) => 1f - (1f - t) * (1f - t);
    public static float InOutQuad(float t) =>
        t < 0.5f ? 2f * t * t : 1f - Mathf.Pow(-2f * t + 2f, 2f) / 2f;

    public static float InCubic(float t)  => t * t * t;
    public static float OutCubic(float t) => 1f - Mathf.Pow(1f - t, 3f);
    public static float InOutCubic(float t) =>
        t < 0.5f ? 4f * t * t * t : 1f - Mathf.Pow(-2f * t + 2f, 3f) / 2f;

    // --- Smoothstep ---
    public static float SmoothStep(float t)   => t * t * (3f - 2f * t);
    public static float SmootherStep(float t) => t * t * t * (t * (6f * t - 15f) + 10f);

    // --- Exponential ---
    public static float InExpo(float t) =>
        t <= 0f ? 0f : Mathf.Pow(2f, 10f * (t - 1f));
    public static float OutExpo(float t) =>
        t >= 1f ? 1f : 1f - Mathf.Pow(2f, -10f * t);

    // --- Elastic ---
    public static float OutElastic(float t)
    {
        const float c4 = (2f * Mathf.PI) / 3f;
        return t <= 0f ? 0f : t >= 1f ? 1f :
            Mathf.Pow(2f, -10f * t) * Mathf.Sin((t * 10f - 0.75f) * c4) + 1f;
    }
}

// --- Usage: Animate a UI panel sliding in ---
public class PanelSlideIn : MonoBehaviour
{
    public RectTransform panel;
    public float duration = 0.5f;

    private Vector2 hiddenPos = new Vector2(800f, 0f);
    private Vector2 shownPos  = Vector2.zero;
    private float elapsed = 0f;
    private bool isAnimating = false;

    public void Show()
    {
        elapsed = 0f;
        isAnimating = true;
    }

    void Update()
    {
        if (!isAnimating) return;

        elapsed += Time.unscaledDeltaTime;
        float t = Mathf.Clamp01(elapsed / duration);
        float easedT = Easing.OutCubic(t);                       // Ease-out for "enter"

        panel.anchoredPosition = Vector2.Lerp(hiddenPos, shownPos, easedT);

        if (t >= 1f) isAnimating = false;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What does an easing function do to the parameter $t$? :: It remaps $t$ to $t' = f(t)$, reshaping the speed curve while keeping $f(0) = 0$ and $f(1) = 1$.
- How do you convert any ease-in $f(t)$ to ease-out? :: Use $f_{\text{out}}(t) = 1 - f_{\text{in}}(1-t)$.
- What is the smoothstep formula? :: $3t^2 - 2t^3$, a cubic hermite with zero derivatives at $t=0$ and $t=1$.
- When should you use ease-out vs ease-in? :: Ease-out for elements *entering* (fast start, gentle arrival). Ease-in for elements *leaving* (slow start, fast exit).
- What is the formula for quadratic ease-in? :: $f(t) = t^2$.
- What makes smootherstep different from smoothstep? :: Smootherstep ($6t^5 - 15t^4 + 10t^3$) also has a zero *second* derivative at the endpoints, giving an even smoother transition.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using ease-in for UI panels *entering* the screen, making them feel sluggish.
> - **The Fix:** Use ease-out (fast start, slow finish) for entering elements. Use ease-in for *exiting* elements.
> - **Why:** Human perception associates deceleration with arrival and settling, and acceleration with departure. Swapping them feels unnatural.

> [!danger] **Watch Out!**
> - **The Mistake:** Applying easing to `Time.deltaTime * speed` lerp patterns instead of a tracked $t \in [0,1]$.
> - **The Fix:** Track a normalized $t = \text{elapsed} / \text{duration}$, apply the easing function to $t$, and pass that eased value into lerp with fixed start/end values.
> - **Why:** The frame-dependent lerp pattern already has its own (exponential) curve. Layering an easing function on top produces unpredictable results.

---

## Related Topics
- [[Math/08_Curves_Interpolation/lerp_fundamentals|Lerp Fundamentals]]
- [[Math/08_Curves_Interpolation/bezier_curves|Bézier Curves]]
- [[Math/01_Algebra_Foundations/algebra_fundamentals|Algebra Fundamentals]]
