---
title: "Number Systems: The Invisible Foundation of Every Pixel"
tags:
  - math
  - level/Lv1
  - category/foundations
---

# Number Systems: The Invisible Foundation of Every Pixel

> [!abstract] **The Concept in a Nutshell**
> Computers don't think in numbers the way we do — they approximate. Understanding how integers, floating-point numbers (IEEE 754), and fixed-point representations work is critical because the *precision limits* of these systems create real, visible bugs in games: z-fighting, jittering objects, and desynced multiplayer physics. Knowing which number type to use and *why* is the difference between a polished game and a glitchy mess.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Z-Fighting at the Edge of the World**
> You're building an open-world RPG. Near the origin $(0, 0, 0)$, everything looks perfect — walls are crisp, shadows are clean. But the player walks 10 km from the origin to a distant fortress. Suddenly, two overlapping wall surfaces start flickering wildly — alternating which one renders in front. This is **z-fighting**, and it happens because a 32-bit `float` can only represent about 7 significant decimal digits. At position $x = 10{,}000$, the smallest representable gap between two floats is roughly $\approx 0.001$ meters. Two surfaces $0.0005$ m apart become *the same number* to the GPU. The depth buffer can't tell them apart, so it flickers between them every frame.
>
> Meanwhile, your character's sword — a small object at local scale — jitters slightly because its world position is being computed as $10{,}000.003 + 0.0001$, and that $0.0001$ simply **vanishes** in the float's limited precision.

---

## The Blueprint (Formula & Structure)

### Integers
Whole numbers with exact representation. No precision loss, but limited range.

| Type | Bits | Range |
|------|------|-------|
| `byte` | 8 | $0$ to $255$ |
| `short` | 16 | $-32{,}768$ to $32{,}767$ |
| `int` | 32 | $-2^{31}$ to $2^{31}-1$ ($\approx \pm 2.1$ billion) |
| `long` | 64 | $-2^{63}$ to $2^{63}-1$ |

### Floating-Point (IEEE 754)
A `float` stores a number as:

$$(-1)^{s} \times 1.m \times 2^{e - 127}$$

Where:
- $s$ = sign bit (1 bit)
- $e$ = exponent (8 bits for `float`, 11 for `double`)
- $m$ = mantissa/significand (23 bits for `float`, 52 for `double`)

**Key properties:**
- **Float (32-bit):** ~7 decimal digits of precision, range $\approx \pm 3.4 \times 10^{38}$
- **Double (64-bit):** ~15–16 decimal digits of precision, range $\approx \pm 1.8 \times 10^{308}$
- Precision is **relative**, not absolute — small numbers are more precise than large ones
- The gap between adjacent floats near $1.0$ is $\epsilon \approx 1.19 \times 10^{-7}$ (machine epsilon)

### Precision Degradation by Magnitude

| Value Range | Approximate Gap Between Adjacent Floats |
|-------------|------------------------------------------|
| $1$ | $\approx 0.0000001$ |
| $1{,}000$ | $\approx 0.0001$ |
| $100{,}000$ | $\approx 0.01$ |
| $10{,}000{,}000$ | $\approx 1.0$ |

### Fixed-Point Numbers
Represent a number as an integer scaled by a fixed denominator:

$$\text{value} = \frac{\text{raw\_integer}}{2^{f}}$$

Where $f$ is the number of fractional bits. For example, a **16.16 fixed-point** format uses 16 bits for the integer part and 16 bits for the fractional part, giving a precision of $\frac{1}{65536} \approx 0.0000153$.

**Advantage:** Every value at any magnitude has the *same* precision. No degradation at large distances.
**Disadvantage:** Much smaller range, no hardware acceleration on modern GPUs.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Ruler That Stretches**
> Imagine a ruler with exactly 100 tick marks. If the ruler covers 1 meter, each tick is 1 cm apart — very precise. Now stretch that same ruler to cover 1 kilometer. The same 100 ticks now represent 10-meter gaps. You can't measure anything smaller than 10 m.
>
> That's exactly how floating-point works: you always have roughly the same number of "ticks" (significant digits), but as the number gets bigger, the spacing between ticks grows. Near zero, you can distinguish incredibly tiny differences. At $x = 10{,}000$, nearby values blur together.
>
> **Fixed-point** is like having a ruler that *never stretches* — every region of the number line gets the same tick density. But the ruler is much shorter (smaller range).

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Demonstrating float precision issues and workarounds
using UnityEngine;

public class PrecisionDemo : MonoBehaviour
{
    void Start()
    {
        // --- Float precision loss at large values ---
        float bigX = 100000f;
        float smallOffset = 0.001f;
        float result = bigX + smallOffset;
        // result is 100000.0f — the 0.001 is LOST!
        Debug.Log($"100000 + 0.001 = {result:F6}"); // prints 100000.000000

        // --- Equality comparison trap ---
        float a = 0.1f + 0.2f;
        float b = 0.3f;
        Debug.Log($"0.1 + 0.2 == 0.3? {a == b}"); // FALSE!

        // --- Correct float comparison using epsilon ---
        bool isClose = Mathf.Approximately(a, b); // true
        Debug.Log($"Approximately equal? {isClose}");

        // --- Relative-to-origin technique for large worlds ---
        // Instead of storing world positions as absolute floats,
        // use a "floating origin" that recenters the world near the player.
        Vector3 playerWorldPos = new Vector3(100000f, 0f, 100000f);
        Vector3 floatingOrigin = playerWorldPos; // recenter!
        Vector3 localPos = playerWorldPos - floatingOrigin; // (0, 0, 0) — maximum precision
        Debug.Log($"Local position after recentering: {localPos}");
    }

    // --- Fixed-point simulation for deterministic multiplayer ---
    // Represent values as integers with an implicit scale factor
    struct FixedPoint
    {
        public long rawValue; // 32.32 fixed-point
        const int FractionalBits = 32;
        const long Scale = 1L << FractionalBits;

        public FixedPoint(double value) => rawValue = (long)(value * Scale);
        public double ToDouble() => (double)rawValue / Scale;

        public static FixedPoint operator +(FixedPoint a, FixedPoint b)
            => new FixedPoint { rawValue = a.rawValue + b.rawValue };

        public static FixedPoint operator *(FixedPoint a, FixedPoint b)
            => new FixedPoint { rawValue = (a.rawValue * b.rawValue) >> FractionalBits };
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- How many significant decimal digits does a 32-bit float provide? :: Approximately 7 significant decimal digits.
- Why does z-fighting occur at large distances from the origin? :: Float precision decreases with magnitude — distant surfaces become indistinguishable in the depth buffer because the gap between representable floats exceeds the surface separation.
- What is machine epsilon for a 32-bit float? :: $\epsilon \approx 1.19 \times 10^{-7}$, meaning two floats near $1.0$ must differ by at least this much to be distinct.
- When should you use fixed-point instead of float? :: When you need **deterministic** results across different machines (e.g., lockstep multiplayer), or when uniform precision across all magnitudes matters more than range.
- Why should you never compare floats with `==`? :: Floating-point arithmetic introduces tiny rounding errors, so values that *should* be equal (like $0.1 + 0.2$ and $0.3$) often differ by a small amount. Use `Mathf.Approximately()` or an epsilon threshold.
- What is the "floating origin" technique? :: Periodically recentering the world origin to the player's position so all nearby calculations happen close to zero, where float precision is highest.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Comparing floats with `==` (e.g., `if (health == 0.0f)`).
> - **The Fix:** Use `Mathf.Approximately(a, b)` or `Mathf.Abs(a - b) < epsilon`.
> - **Why:** Float arithmetic rounds at every step. After multiple operations, values drift from their "expected" results by tiny amounts that break exact equality.

> [!danger] **Watch Out!**
> - **The Mistake:** Accumulating small float values over time (e.g., `timer += Time.deltaTime` running for hours).
> - **The Fix:** Use a `double` for long-running accumulators, or periodically reset and track elapsed segments.
> - **Why:** Adding a small float to a large float loses the small value once the magnitudes differ by more than ~7 orders of magnitude. After hours of play, your timer literally stops advancing.

---

## Related Topics
- [[Math/01_Foundations/coordinate_systems_2d|2D Coordinate Systems]]
- [[Math/01_Foundations/coordinate_systems_3d|3D Coordinate Systems]]
- [[Math/01_Foundations/algebra_fundamentals|Algebra Fundamentals]]
