---
title: "Probability & Statistics: The Math of Chance"
tags:
  - math
  - level/Lv3
  - category/advanced_topics
---

# Probability & Statistics: The Math of Chance

> [!abstract] **The Concept in a Nutshell**
> Probability and statistics govern everything random in games — from loot drops and critical hits to procedural generation and AI decision-making. Understanding **pseudo-random number generators (PRNGs)**, **probability distributions**, **expected value**, and **Monte Carlo methods** lets you design fair, engaging, and reproducible random systems that feel genuinely unpredictable while being mathematically controlled.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: An Action RPG Loot System — "Why Won't This Legendary Drop?!"**
> A boss has a 5% legendary drop rate. A player kills it 40 times and gets nothing — they're furious. The math says: the probability of NO legendary in 40 kills is $(0.95)^{40} = 0.129$ — a 12.9% chance. One in eight players experiences this! A smart designer implements a **pity system** using the negative binomial distribution: after 20 kills without a drop, the rate increases by 2% per kill. They also use **seeded PRNGs** so that "daily challenge" loot is identical for all players, enabling fair competition. Meanwhile, the critical hit system uses **weighted random selection** — a warrior with 25% crit chance shouldn't go 20 attacks without critting (that's $(0.75)^{20} = 0.3\%$ — rare but player-feelingly frustrating).

---

## The Blueprint (Formula & Structure)

### Probability Fundamentals

**Probability of event A:**
$$P(A) = \frac{\text{favorable outcomes}}{\text{total outcomes}}, \quad 0 \leq P(A) \leq 1$$

**Complement:**
$$P(\text{not } A) = 1 - P(A)$$

**Independent events (both occur):**
$$P(A \cap B) = P(A) \times P(B)$$

**Union (at least one):**
$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$

**Conditional probability:**
$$P(A|B) = \frac{P(A \cap B)}{P(B)}$$

### Probability of At Least One Success in N Trials

"What's the probability of getting at least one legendary in 40 kills?"

$$P(\text{at least 1}) = 1 - (1 - p)^n = 1 - (0.95)^{40} \approx 0.871$$

### Expected Value

The average outcome over many trials:

$$E[X] = \sum_{i} x_i \cdot P(x_i) \quad \text{(discrete)}$$

**Example:** A loot chest contains: 70% common (worth 10 gold), 25% rare (50 gold), 5% legendary (500 gold).

$$E[\text{gold}] = 0.70 \times 10 + 0.25 \times 50 + 0.05 \times 500 = 7 + 12.5 + 25 = 44.5 \text{ gold}$$

### Pseudo-Random Number Generators (PRNGs)

PRNGs are deterministic algorithms that produce sequences **appearing** random:

**Linear Congruential Generator (LCG):**
$$X_{n+1} = (a \cdot X_n + c) \mod m$$

Where $a$, $c$, $m$ are carefully chosen constants. The **seed** $X_0$ determines the entire sequence — same seed = same sequence (reproducibility).

**Properties of good PRNGs:**
- Long period (before repeating)
- Uniform distribution
- No detectable patterns
- Fast computation

**Common choices:** Mersenne Twister (high quality), xorshift128+ (fast, good), PCG (excellent).

### Key Distributions

**Uniform distribution** — all values equally likely:
$$P(X = k) = \frac{1}{n} \quad \text{for } k \in \{1, 2, \ldots, n\}$$

**Bernoulli** — single yes/no trial with probability $p$:
$$P(X = 1) = p, \quad P(X = 0) = 1 - p$$

**Binomial** — $n$ independent trials, count of successes:
$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

**Normal (Gaussian)** — bell curve, common in nature:
$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x - \mu)^2}{2\sigma^2}}$$

Used for: random scatter patterns, NPC stat generation, accuracy cone for weapons.

**Box-Muller transform** (uniform → normal):
$$Z_0 = \sqrt{-2\ln U_1} \cos(2\pi U_2)$$
$$Z_1 = \sqrt{-2\ln U_1} \sin(2\pi U_2)$$

Where $U_1, U_2$ are uniform random in $(0, 1]$.

### Weighted Random Selection

Given items with weights $w_1, w_2, \ldots, w_n$:

1. Compute total: $W = \sum w_i$
2. Generate uniform random $r \in [0, W)$
3. Walk through items, accumulating weight until $\text{cumulative} > r$

$$P(\text{item } i) = \frac{w_i}{W}$$

### The Law of Large Numbers

As the number of trials $n \to \infty$, the sample average converges to the expected value:

$$\bar{X}_n = \frac{1}{n}\sum_{i=1}^{n} X_i \xrightarrow{n \to \infty} E[X]$$

This is why Monte Carlo methods work — take enough random samples and the average converges to the true answer.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Weighted Dice Bag**
> Imagine a bag of colored marbles: 70 gray (common), 25 blue (rare), 5 gold (legendary). **Probability** is the fraction of gold marbles. **Expected value** is what you'd win *on average* if each color had a prize. A **PRNG** is like a marble-dispensing machine with a specific internal gear arrangement — give it the same starting position (seed), and it dispenses the exact same sequence. **Weighted selection** is making the bag — more gray marbles = higher chance of drawing gray. The **Law of Large Numbers** says: draw 10 marbles and you might get 3 gold (luck!), but draw 10,000 and you'll get very close to 5% gold. The individual draws are unpredictable, but the pattern is mathematically guaranteed.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Complete weighted random loot system with seeded PRNG
using UnityEngine;
using System.Collections.Generic;

public class LootSystem : MonoBehaviour
{
    [System.Serializable]
    public struct LootEntry
    {
        public string name;
        public float weight;
        public int goldValue;
    }

    [SerializeField] private LootEntry[] lootTable =
    {
        new LootEntry { name = "Iron Sword", weight = 50, goldValue = 10 },
        new LootEntry { name = "Silver Shield", weight = 30, goldValue = 50 },
        new LootEntry { name = "Gold Amulet", weight = 15, goldValue = 200 },
        new LootEntry { name = "Legendary Staff", weight = 5, goldValue = 1000 },
    };

    private System.Random seededRng;

    void Start()
    {
        // Seeded PRNG — same seed = same loot sequence (reproducible)
        int dailySeed = System.DateTime.Now.DayOfYear * 1000 + 42;
        seededRng = new System.Random(dailySeed);
        Debug.Log($"Daily seed: {dailySeed}");

        // Simulate 1000 drops and check distribution
        Dictionary<string, int> counts = new Dictionary<string, int>();
        float totalGold = 0;

        for (int i = 0; i < 1000; i++)
        {
            LootEntry drop = WeightedRandomPick();
            counts[drop.name] = counts.GetValueOrDefault(drop.name) + 1;
            totalGold += drop.goldValue;
        }

        // Law of Large Numbers: actual frequencies should approach weights
        foreach (var kvp in counts)
            Debug.Log($"{kvp.Key}: {kvp.Value}/1000 ({kvp.Value / 10f}%)");
        Debug.Log($"Average gold per drop: {totalGold / 1000f:F1}");
    }

    LootEntry WeightedRandomPick()
    {
        float totalWeight = 0;
        foreach (var entry in lootTable) totalWeight += entry.weight;

        float roll = (float)(seededRng.NextDouble() * totalWeight);
        float cumulative = 0;

        foreach (var entry in lootTable)
        {
            cumulative += entry.weight;
            if (roll < cumulative) return entry;
        }

        return lootTable[lootTable.Length - 1]; // fallback
    }

    // Gaussian random for weapon accuracy spread
    public Vector2 GaussianSpread(float standardDev)
    {
        // Box-Muller transform
        float u1 = 1f - (float)seededRng.NextDouble(); // (0, 1]
        float u2 = (float)seededRng.NextDouble();
        float mag = standardDev * Mathf.Sqrt(-2f * Mathf.Log(u1));
        float angle = 2f * Mathf.PI * u2;
        return new Vector2(mag * Mathf.Cos(angle), mag * Mathf.Sin(angle));
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the probability of getting at least one success in N independent trials with probability p? :: $P(\text{at least 1}) = 1 - (1-p)^N$. For example, at least one crit in 10 attacks with 20% crit rate: $1 - 0.8^{10} = 0.893$ or 89.3%.
- Why are PRNGs called "pseudo-random"? :: They're deterministic algorithms — given the same seed, they produce the exact same sequence. They only *appear* random because the output has no detectable pattern, but they are fully reproducible.
- What does the expected value represent in a loot system? :: The average reward per drop over many trials. If items are worth 10, 50, and 500 gold with probabilities 70%, 25%, and 5%, the expected value is $0.7(10) + 0.25(50) + 0.05(500) = 44.5$ gold per drop.
- How does the Box-Muller transform work? :: It converts two uniform random variables $U_1, U_2 \in (0, 1]$ into two independent standard normal variables using $Z = \sqrt{-2\ln U_1} \cos(2\pi U_2)$. This is useful for weapon accuracy cones and natural-feeling randomness.
- What is the Law of Large Numbers and why does it matter for game balance? :: As the number of trials increases, the sample average converges to the expected value. This means your loot tables, crit rates, and drop percentages ARE accurate — but only over many trials. Individual player experiences can vary wildly in the short term.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `System.Random` for multiplayer games where each client needs identical random sequences but initializes with different seeds.
> - **The Fix:** Synchronize a seed from the server (e.g., room creation timestamp). All clients use the same seeded PRNG instance for game-critical randomness.
> - **Why:** `System.Random` with different seeds produces completely different sequences. In networked games, this causes desynchronization — one client sees a critical hit while another sees a normal attack.

> [!danger] **Watch Out!**
> - **The Mistake:** Assuming $P = 5\%$ means you'll get the item within 20 attempts ("it's 1 in 20!").
> - **The Fix:** Calculate: $P(\text{at least 1 in 20}) = 1 - 0.95^{20} = 0.642$ — only 64%! For 95% confidence, you need $\frac{\ln(0.05)}{\ln(0.95)} \approx 59$ attempts.
> - **Why:** Each trial is independent. The probability doesn't "build up" — the coin doesn't remember previous flips. This misconception (the Gambler's Fallacy) leads to frustrated players and poorly designed pity systems.

---

## Related Topics
- [[Math/12_Advanced_Topics/procedural_noise|Procedural Noise]]
- [[Math/11_Graphics_Math/ray_tracing_math|Ray Tracing Mathematics]]
- [[Math/01_Foundations/algebra_fundamentals|Algebra Fundamentals]]
