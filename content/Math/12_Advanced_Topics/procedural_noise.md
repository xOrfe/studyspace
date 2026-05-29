---
title: "Procedural Noise: Controlled Chaos"
tags:
  - math
  - level/Lv3
  - category/advanced_topics
---

# Procedural Noise: Controlled Chaos

> [!abstract] **The Concept in a Nutshell**
> Procedural noise functions generate smooth, coherent pseudorandom patterns from spatial coordinates — no stored textures needed. **Perlin noise** and **Simplex noise** produce natural-looking gradients. Layering multiple octaves via **fractal Brownian motion (FBM)** creates complex, self-similar detail at multiple scales. **Domain warping** distorts the input space for surreal, organic effects. These techniques power terrain generation, cloud systems, fire effects, and everything procedural in games.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Survival Game — Infinite Procedural Terrain**
> The player walks endlessly through a procedural world. Each chunk of terrain is generated on-the-fly using noise functions. **Base terrain** uses 2D Perlin noise mapped to height: $\text{height}(x, z) = \text{noise}(x \cdot 0.01, z \cdot 0.01) \times 100$. **Detail** is added via FBM — 6 octaves of noise layered together, each adding finer detail: rolling hills (octave 1), rocky ridges (octave 3), pebble-scale bumps (octave 6). **Biomes** use a separate noise layer: $\text{temperature}(x, z) = \text{noise}(x \cdot 0.005, z \cdot 0.005)$ determines desert vs. forest vs. tundra. **Caves** use 3D noise — where $\text{noise}(x, y, z) > 0.6$, carve out empty space. Domain warping makes rivers meander naturally. Same seed = same world every time. Different seed = brand new planet.

---

## The Blueprint (Formula & Structure)

### What Makes Noise "Coherent"?

Good procedural noise has these properties:
- **Deterministic:** Same input → same output (no stored state needed)
- **Continuous:** Nearby inputs produce nearby outputs (smooth gradients)
- **Bounded:** Output typically in $[-1, 1]$ or $[0, 1]$
- **No obvious patterns:** No visible grid artifacts or repetition
- **Controllable frequency:** Easily adjustable detail scale

### Perlin Noise (Classic Gradient Noise)

**1D Concept:** For input $x$:
1. Find the two integer grid points surrounding $x$: $x_0 = \lfloor x \rfloor$, $x_1 = x_0 + 1$
2. Assign a **random gradient** (slope) to each grid point via a hash function
3. Compute the **dot product** between each gradient and the vector from the grid point to $x$
4. **Interpolate** between the two dot products using a smooth fade function

**2D Algorithm:**
1. For input $(x, y)$, find the surrounding unit square with corners $(i, j)$, $(i+1, j)$, $(i, j+1)$, $(i+1, j+1)$
2. Hash each corner to select a **gradient vector** from a set (e.g., $\{(1,1), (-1,1), (1,-1), (-1,-1)\}$)
3. Compute distance vectors from each corner to $(x, y)$
4. Dot product each gradient with its distance vector:

$$d_{ij} = \vec{g}_{ij} \cdot \begin{pmatrix} x - i \\ y - j \end{pmatrix}$$

5. Interpolate using a smooth **fade curve** $f(t) = 6t^5 - 15t^4 + 10t^3$:

$$\text{noise}(x, y) = \text{lerp}_y\left(\text{lerp}_x(d_{00}, d_{10}, f(x_f)),\; \text{lerp}_x(d_{01}, d_{11}, f(x_f)),\; f(y_f)\right)$$

Where $x_f, y_f$ are fractional parts of $x, y$.

### Simplex Noise (Improved)

Ken Perlin's improved algorithm uses a **simplex grid** (triangles in 2D, tetrahedra in 3D) instead of a square grid:

| Feature | Perlin | Simplex |
|---|---|---|
| Grid | Hypercube ($2^n$ corners) | Simplex ($n+1$ corners) |
| 2D corners | 4 | 3 |
| 3D corners | 8 | 4 |
| Complexity | $O(2^n)$ | $O(n^2)$ |
| Artifacts | Visible axis-aligned patterns | No directional artifacts |

Simplex noise scales much better to higher dimensions and produces isotropic (direction-independent) patterns.

### Fractal Brownian Motion (FBM)

Layer multiple octaves of noise at increasing frequency and decreasing amplitude:

$$\text{FBM}(x) = \sum_{i=0}^{n-1} \text{amplitude}_i \cdot \text{noise}(\text{frequency}_i \cdot x)$$

Where:
$$\text{frequency}_i = \text{lacunarity}^i, \quad \text{amplitude}_i = \text{persistence}^i$$

**Key parameters:**
- **Octaves** ($n$): number of noise layers (more = finer detail, more expensive)
- **Lacunarity**: frequency multiplier per octave (typically 2.0 — each octave is twice the frequency)
- **Persistence** (gain): amplitude multiplier per octave (typically 0.5 — each octave is half the amplitude)

**Example with 4 octaves, lacunarity=2, persistence=0.5:**

| Octave | Frequency | Amplitude | Contribution |
|---|---|---|---|
| 0 | 1× | 1.0 | Broad hills |
| 1 | 2× | 0.5 | Medium ridges |
| 2 | 4× | 0.25 | Small bumps |
| 3 | 8× | 0.125 | Fine detail |

### Domain Warping

Feed the output of one noise function as the input offset of another:

$$\text{warped}(x, y) = \text{noise}\left(x + a \cdot \text{noise}(x, y),\; y + b \cdot \text{noise}(x + 5.2, y + 1.3)\right)$$

This creates swirling, organic patterns — excellent for marble textures, lava flow, magical energy effects.

**Multi-level warping** (warping the warp):

$$\vec{q} = \begin{pmatrix} \text{noise}(x, y) \\ \text{noise}(x + 5.2, y + 1.3) \end{pmatrix}$$

$$\vec{r} = \begin{pmatrix} \text{noise}(x + a \cdot q_x, y + a \cdot q_y) \\ \text{noise}(x + b \cdot q_x + 1.7, y + b \cdot q_y + 9.2) \end{pmatrix}$$

$$\text{result}(x, y) = \text{noise}(x + c \cdot r_x, y + c \cdot r_y)$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Mountain Seen from Different Distances**
> FBM is like describing a mountain range at different zoom levels. From an airplane (octave 0), you see broad peaks and valleys — low frequency, high amplitude. Walking closer (octave 1), you notice ridges on the slopes. Closer still (octave 2), you see individual boulders. On your hands and knees (octave 3), you see pebbles and cracks. **Each zoom level adds finer detail at smaller amplitude.** The mountain's overall shape comes from the first octave; the surface texture comes from the later ones. **Lacunarity** controls how much you zoom in each time (2× = double the detail). **Persistence** controls how much influence each zoom level has (0.5× = each is half as prominent). Domain warping is like viewing the mountain through a fun-house mirror — the underlying detail is the same, but the coordinate system is distorted.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Procedural terrain generation using FBM noise
using UnityEngine;

public class ProceduralTerrain : MonoBehaviour
{
    [SerializeField] private int width = 256;
    [SerializeField] private int depth = 256;
    [SerializeField] private float heightScale = 30f;
    [SerializeField] private float noiseScale = 0.02f;
    [SerializeField] private int octaves = 6;
    [SerializeField] private float lacunarity = 2f;
    [SerializeField] private float persistence = 0.5f;
    [SerializeField] private int seed = 42;

    void Start()
    {
        Terrain terrain = GetComponent<Terrain>();
        terrain.terrainData = GenerateTerrain(terrain.terrainData);
    }

    TerrainData GenerateTerrain(TerrainData data)
    {
        data.heightmapResolution = width + 1;
        data.size = new Vector3(width, heightScale, depth);
        data.SetHeights(0, 0, GenerateHeights());
        return data;
    }

    float[,] GenerateHeights()
    {
        float[,] heights = new float[depth, width];
        System.Random prng = new System.Random(seed);
        Vector2[] octaveOffsets = new Vector2[octaves];

        for (int i = 0; i < octaves; i++)
            octaveOffsets[i] = new Vector2(prng.Next(-10000, 10000), prng.Next(-10000, 10000));

        for (int z = 0; z < depth; z++)
        {
            for (int x = 0; x < width; x++)
            {
                heights[z, x] = FBM(x, z, octaveOffsets);
            }
        }
        return heights;
    }

    float FBM(float x, float z, Vector2[] offsets)
    {
        float amplitude = 1f;
        float frequency = 1f;
        float value = 0f;
        float maxPossible = 0f; // for normalization

        for (int i = 0; i < octaves; i++)
        {
            float sampleX = (x + offsets[i].x) * noiseScale * frequency;
            float sampleZ = (z + offsets[i].y) * noiseScale * frequency;

            // Mathf.PerlinNoise returns [0,1], remap to [-1,1]
            float noise = Mathf.PerlinNoise(sampleX, sampleZ) * 2f - 1f;
            value += noise * amplitude;

            maxPossible += amplitude;
            amplitude *= persistence;
            frequency *= lacunarity;
        }

        // Normalize to [0, 1]
        return (value / maxPossible + 1f) * 0.5f;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What makes procedural noise "coherent" vs. pure random? :: Coherent noise is continuous — nearby inputs produce nearby outputs, creating smooth gradients. Pure random (white noise) has no spatial correlation, producing static-like patterns with no structure.
- What do lacunarity and persistence control in FBM? :: **Lacunarity** = frequency multiplier per octave (typically 2.0 — each layer has double the frequency/detail). **Persistence** (gain) = amplitude multiplier (typically 0.5 — each layer has half the influence). Together they control the "roughness" character.
- How does Perlin noise avoid grid-aligned artifacts? :: It uses a smooth fade function $f(t) = 6t^5 - 15t^4 + 10t^3$ (improved Perlin) for interpolation instead of linear lerp. This ensures the first and second derivatives are zero at grid boundaries, eliminating visible grid lines.
- What is domain warping? :: Feeding the output of one noise function as a coordinate offset for another: $\text{noise}(x + a \cdot \text{noise}(x,y), ...)$. This creates swirling, organic distortions — perfect for marble, lava, clouds, and magical effects.
- Why is Simplex noise preferred over classic Perlin in higher dimensions? :: Simplex uses $n+1$ vertices per cell (vs. $2^n$ for Perlin's hypercube), making it $O(n^2)$ vs. $O(2^n)$. In 3D: 4 points vs. 8. In 4D: 5 points vs. 16. It also produces no directional artifacts.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using `Mathf.PerlinNoise` with integer coordinates — you get the same value at every integer because Perlin noise is defined to be 0 at grid intersections.
> - **The Fix:** Always multiply coordinates by a non-integer scale factor: `noise(x * 0.1f, z * 0.1f)`. Never pass pure integers.
> - **Why:** Perlin noise computes dot products between gradient vectors and distance vectors. At integer coordinates, the distance to the grid corner is zero → the dot product is zero → the value is always 0 (or close to 0 with improved implementations).

> [!danger] **Watch Out!**
> - **The Mistake:** Using too many octaves for the visible detail level — wasting performance on sub-pixel noise.
> - **The Fix:** Calculate max useful octaves: if the highest frequency creates detail smaller than one pixel, skip that octave. Generally 4-6 octaves suffice for terrain, 2-3 for clouds.
> - **Why:** Each additional octave adds computation but produces detail at half the amplitude and double the frequency. Beyond the pixel resolution, this detail is invisible but still costs full computation time.

---

## Related Topics
- [[Math/12_Advanced_Topics/probability_statistics|Probability & Statistics]]
- [[Math/01_Foundations/algebra_fundamentals|Algebra Fundamentals]]
- [[Math/11_Graphics_Math/texture_normal_mapping|Texture & Normal Mapping]]
