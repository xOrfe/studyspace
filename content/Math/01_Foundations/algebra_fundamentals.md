---
title: "Algebra Fundamentals: The Language Every Game System Speaks"
tags:
  - math
  - level/Lv1
  - category/foundations
---

# Algebra Fundamentals: The Language Every Game System Speaks

> [!abstract] **The Concept in a Nutshell**
> Algebra is the grammar of game systems. Every damage formula, XP curve, difficulty ramp, and economy model is an algebraic function — a rule that maps inputs to outputs. Understanding linear, quadratic, and exponential functions lets you *design* how your game feels, not just stumble into it.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Balancing a Dungeon Crawler's Progression**
> You're designing "Crypt Delver," an action RPG. The warrior Kael is level 12, wielding a +3 Flame Sword (base damage 40). You need to answer:
>
> - **Damage formula:** `finalDamage = (baseDamage + weaponBonus) * levelMultiplier - enemyArmor`. With `baseDamage = 40`, `weaponBonus = 3`, `levelMultiplier = 1 + 0.05 * level = 1.6`, and `enemyArmor = 15`: $\text{finalDamage} = (40 + 3) \times 1.6 - 15 = 68.8 - 15 = 53.8$.
> - **XP to next level:** Should it be linear ($100 \times \text{level}$), quadratic ($50 \times \text{level}^2$), or exponential ($100 \times 1.5^{\text{level}}$)? Each creates a dramatically different pacing feel.
> - **Enemy scaling:** If enemies grow linearly but player power grows quadratically, the player will eventually trivialize all content. If it's the reverse, the game becomes impossibly hard.
>
> Every design decision here is an algebraic function choice.

---

## The Blueprint (Formula & Structure)

### Variables and Expressions
A **variable** is a named placeholder for a value. An **expression** combines variables with operations:

$$\text{damage} = \text{base} \times (1 + \text{critMultiplier}) - \text{armor}$$

### Functions: Input → Output
A **function** $f(x)$ maps every input $x$ to exactly one output. In games, the "input" is often level, distance, time, or a stat.

$$f(x) = mx + b \quad \text{(linear)}$$

### The Three Essential Growth Models

**1. Linear:** $f(x) = mx + b$
- Constant rate of change (slope $m$)
- Example: "Each level grants +5 HP" → $\text{HP}(level) = 5 \times level + 100$
- Graph: straight line

**2. Quadratic:** $f(x) = ax^2 + bx + c$
- Accelerating growth (or deceleration if $a < 0$)
- Example: XP cost curve → $\text{XP}(level) = 50 \times level^2$
- Level 5 costs 1,250 XP; Level 20 costs 20,000 XP — steep ramp
- Graph: parabola

**3. Exponential:** $f(x) = a \cdot b^x$
- Multiplicative growth — each step multiplies by a constant factor $b$
- Example: Gold inflation → $\text{goldReward}(zone) = 10 \times 2^{zone}$
- Zone 1 = 20 gold, Zone 10 = 10,240 gold
- Graph: hockey stick curve

### Key Operations Refresher

| Operation | Formula | Example |
|-----------|---------|---------|
| Solving for $x$ | $ax + b = c \Rightarrow x = \frac{c-b}{a}$ | "At what level does HP reach 200?" |
| Substitution | Replace variable with value | $f(5) = 3(5)^2 + 2 = 77$ |
| Composition | $f(g(x))$ | `ApplyArmor(CalculateDamage(stats))` |
| Inverse | $f^{-1}(y)$ finds $x$ given output $y$ | "What level gives 10,000 XP?" |

### Inverse Functions (Solving Backwards)
If $\text{XP}(level) = 50 \times level^2$, what level corresponds to $4{,}500$ XP?

$$4500 = 50 \times level^2 \quad\Rightarrow\quad level^2 = 90 \quad\Rightarrow\quad level = \sqrt{90} \approx 9.49$$

Round down: the player is level **9** with some progress toward 10.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Three Hills**
> Imagine three hikers climbing hills:
>
> - **Linear:** A steady, flat slope — every step gains the same height. Predictable but boring. This is "+5 damage per level."
> - **Quadratic:** The hill gets steeper as you climb. Early levels are easy; later levels demand much more effort. This is most RPG XP curves.
> - **Exponential:** The hill is nearly flat at first, then suddenly becomes a vertical cliff. This is idle-game currency or unchecked compound interest.
>
> When designing game systems, ask: "Which hill do I want my players climbing?" Linear feels fair but flat. Quadratic creates a satisfying difficulty ramp. Exponential creates dramatic late-game scaling but can spiral out of control.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Game system functions: damage, XP curves, difficulty scaling
using UnityEngine;

public class AlgebraInAction : MonoBehaviour
{
    // --- Linear function: flat stat growth ---
    // f(level) = base + growth * level
    public static float LinearHP(int level, float baseHP = 100f, float hpPerLevel = 5f)
    {
        return baseHP + hpPerLevel * level;
        // Level 1: 105, Level 20: 200, Level 100: 600
    }

    // --- Quadratic function: XP cost curve ---
    // f(level) = a * level^2 + b * level + c
    public static int XPToNextLevel(int level, float a = 50f, float b = 100f)
    {
        return Mathf.RoundToInt(a * level * level + b * level);
        // Level 1: 150, Level 10: 6000, Level 50: 130000
    }

    // --- Exponential function: enemy scaling ---
    // f(zone) = base * multiplier^zone
    public static float EnemyHP(int zone, float baseHP = 100f, float multiplier = 1.3f)
    {
        return baseHP * Mathf.Pow(multiplier, zone);
        // Zone 0: 100, Zone 5: 371, Zone 10: 1379, Zone 20: 19005
    }

    // --- Inverse function: what level am I at given total XP? ---
    public static int LevelFromTotalXP(int totalXP, float a = 50f)
    {
        // Inverting f(level) = a * level^2  →  level = sqrt(totalXP / a)
        return Mathf.FloorToInt(Mathf.Sqrt(totalXP / a));
    }

    // --- Composition: full damage pipeline ---
    public static float CalculateFinalDamage(float baseDmg, int level, float armor)
    {
        float levelMultiplier = 1f + 0.05f * level; // linear scaling
        float rawDamage = baseDmg * levelMultiplier;   // apply level
        float mitigated = Mathf.Max(rawDamage - armor, 0f); // subtract armor, floor at 0
        return mitigated;
    }

    void Start()
    {
        Debug.Log($"Level 10 HP: {LinearHP(10)}");                // 150
        Debug.Log($"XP for Level 15: {XPToNextLevel(15)}");       // 12750
        Debug.Log($"Zone 8 Enemy HP: {EnemyHP(8):F0}");           // ~816
        Debug.Log($"Level from 5000 XP: {LevelFromTotalXP(5000)}"); // 10
        Debug.Log($"Damage (base40, lv12, armor15): {CalculateFinalDamage(40, 12, 15)}"); // 49
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What makes a function "linear"? :: It has the form $f(x) = mx + b$ — a constant rate of change (slope $m$), producing a straight line graph.
- Why do most RPGs use quadratic XP curves instead of linear? :: Quadratic curves create a natural difficulty ramp — early levels come quickly (keeping new players engaged), while later levels require significantly more effort, extending endgame longevity.
- How do you find the inverse of $f(x) = 50x^2$? :: Set $y = 50x^2$, solve for $x$: $x = \sqrt{y / 50}$. This tells you "what level corresponds to $y$ total XP?"
- What is function composition in game terms? :: Feeding the output of one function into another — e.g., `ApplyArmor(ApplyLevelScaling(baseDamage))`. The order matters!
- What growth model does $f(x) = 100 \cdot 1.5^x$ represent? :: Exponential growth — each increment of $x$ multiplies the output by $1.5$. It starts slow but escalates rapidly.
- If enemy HP grows exponentially but player damage grows linearly, what happens? :: The player eventually hits a wall where enemies become unkillable — a common balancing mistake that requires either capping enemy scaling or introducing multiplicative player power (gear upgrades, prestige systems).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using exponential scaling without testing late-game values. A designer sets enemy HP to $100 \times 1.5^{\text{level}}$ and only tests levels 1–10. By level 30, enemies have $191{,}751$ HP — completely unplayable.
> - **The Fix:** Always plot your function across the *entire* expected input range. Use a spreadsheet or graphing tool before committing to a formula.
> - **Why:** Exponential growth is deceptively slow at first. The "hockey stick" inflection point often lands right where you stopped testing.

> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to clamp outputs. A damage formula like `baseDmg - armor` can go negative, "healing" the enemy.
> - **The Fix:** Always apply `Mathf.Max(result, 0f)` or a minimum damage floor.
> - **Why:** Algebraic formulas don't know about game logic constraints. You must enforce them explicitly.

---

## Related Topics
- [[Math/02_Geometry_Trigonometry/trigonometric_functions|Trigonometric Functions]]
- [[Math/08_Curves_Interpolation/lerp_fundamentals|Linear Interpolation (LERP)]]
- [[Math/01_Foundations/number_systems|Number Systems & Representation]]
- [[Math/08_Curves_Interpolation/easing_functions|Easing Functions]]
