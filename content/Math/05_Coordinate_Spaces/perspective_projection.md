---
title: "Perspective Projection: The Art of Making 3D Look 3D"
tags:
  - math
  - level/Lv3
  - category/coordinate_spaces
---

# Perspective Projection: The Art of Making 3D Look 3D

> [!abstract] **The Concept in a Nutshell**
> The perspective projection matrix transforms 3D camera-space geometry into a normalized clip volume while encoding **perspective foreshortening** — far objects appear smaller. It is parameterized by **field of view (FOV)**, **aspect ratio**, and **near/far clipping planes**. Changing these parameters mimics real camera lenses: narrow FOV = telephoto zoom, wide FOV = fisheye. The $w$ component produced by the matrix enables the perspective divide that creates the illusion of depth.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Sniper Scope in a Tactical Shooter**
> You're playing a tactical shooter. Normally, your camera has a $60°$ FOV — a comfortable default. You press the right mouse button to aim through the sniper scope, and the FOV smoothly narrows to $8°$. Distant enemies that were tiny specks now fill the screen. The world appears to "flatten" — objects at different depths look closer together (telephoto compression).
>
> When you switch to a security camera view with a fisheye lens ($120°$ FOV), everything near the edges stretches and curves. The room looks huge, but distances are exaggerated.
>
> All of this is controlled by a single parameter in the perspective projection matrix: the **field of view**. The matrix itself handles all the math to make closer things bigger, farther things smaller, and map the visible frustum-shaped volume into a rectangular image.

---

## The Blueprint (Formula & Structure)

### The Perspective Projection Matrix

Given:
- $\theta$ = vertical field of view (in radians)
- $a$ = aspect ratio ($\text{width} / \text{height}$)
- $n$ = near clipping plane distance
- $f$ = far clipping plane distance

**OpenGL convention** (NDC $z \in [-1, 1]$):

$$\mathbf{P} = \begin{bmatrix} \frac{1}{a \cdot \tan(\theta/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(\theta/2)} & 0 & 0 \\ 0 & 0 & -\frac{f + n}{f - n} & -\frac{2fn}{f - n} \\ 0 & 0 & -1 & 0 \end{bmatrix}$$

**DirectX/Unity convention** (NDC $z \in [0, 1]$):

$$\mathbf{P} = \begin{bmatrix} \frac{1}{a \cdot \tan(\theta/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(\theta/2)} & 0 & 0 \\ 0 & 0 & \frac{f}{n - f} & \frac{nf}{n - f} \\ 0 & 0 & -1 & 0 \end{bmatrix}$$

### Key Insight: The $w$ Component

The bottom row $[0, 0, -1, 0]$ copies the **negative $z$** (depth) into the output $w$ component:

$$w_\text{clip} = -z_\text{view}$$

After the perspective divide $(x/w, y/w, z/w)$, everything is divided by the distance from the camera. This is why far objects become smaller — they're divided by a bigger number.

### FOV and Focal Length Relationship

$$f_\text{focal} = \frac{1}{\tan(\theta / 2)}$$

| FOV | Effect | Lens Equivalent |
|---|---|---|
| $120°$ | Wide angle, extreme distortion | Fisheye |
| $90°$ | Common for FPS games | Wide angle |
| $60°$ | Natural look, comfortable | Standard lens |
| $30°$ | Noticeable zoom | Telephoto |
| $8°$ | Extreme zoom, flat look | Super telephoto / Sniper scope |

### Depth Buffer Precision

The mapping of depth values is **non-linear** — more precision is concentrated near the near plane:

$$z_\text{ndc} = \frac{A \cdot z_\text{view} + B}{-z_\text{view}}$$

This means:
- Choosing $n = 0.001$ and $f = 10000$ wastes almost all depth precision on the first few meters.
- **Z-fighting** occurs when distant surfaces share nearly the same depth buffer value.
- **Rule of thumb:** keep the ratio $f/n$ as small as possible. $f/n < 1000$ is good.

### Reversed Depth Buffer (Modern Best Practice)

Many modern engines use a **reversed-Z** buffer ($z_\text{ndc} = 1$ at near, $z_\text{ndc} = 0$ at far) with a floating-point depth buffer. This distributes precision more evenly because float precision is also concentrated near zero.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: A Flashlight Cone Projected onto a Wall**
> Imagine holding a flashlight (your camera) and shining it on a wall (your screen).
>
> - Objects close to the flashlight cast **big shadows** on the wall. Objects far away cast **tiny shadows**. That's perspective.
> - **FOV** is how wide you open the flashlight cone. A narrow beam (small FOV) = telephoto zoom. A wide flood light (large FOV) = fisheye.
> - The **near plane** is like a glass pane right in front of the flashlight — anything closer than the glass isn't lit. The **far plane** is how far the light reaches.
> - The projection matrix is the mathematical description of how the cone-shaped beam maps onto the flat wall. The **frustum** (a truncated pyramid) is the shape of the lit volume.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Sniper scope zoom effect and custom projection
using UnityEngine;

public class SniperScope : MonoBehaviour
{
    public Camera mainCam;

    [Header("FOV Settings")]
    public float normalFOV = 60f;
    public float scopeFOV = 8f;
    public float zoomSpeed = 5f;

    [Header("Clip Planes")]
    public float nearPlane = 0.3f;
    public float farPlane = 2000f;

    private float currentFOV;
    private bool isScoped = false;

    void Start()
    {
        currentFOV = normalFOV;
        mainCam.nearClipPlane = nearPlane;
        mainCam.farClipPlane = farPlane;
    }

    void Update()
    {
        // Toggle scope
        if (Input.GetMouseButtonDown(1))
            isScoped = !isScoped;

        // Smooth FOV transition
        float targetFOV = isScoped ? scopeFOV : normalFOV;
        currentFOV = Mathf.Lerp(currentFOV, targetFOV, Time.deltaTime * zoomSpeed);
        mainCam.fieldOfView = currentFOV;

        // Show the current focal length equivalent
        float focalLength = 1f / Mathf.Tan(currentFOV * Mathf.Deg2Rad * 0.5f);
        Debug.Log($"FOV: {currentFOV:F1}° | Focal length: {focalLength:F2}");

        // Demonstrate building a projection matrix manually
        Matrix4x4 customProj = BuildPerspectiveMatrix(
            currentFOV * Mathf.Deg2Rad,
            mainCam.aspect,
            nearPlane,
            farPlane
        );

        // Compare with Unity's built-in
        Matrix4x4 unityProj = mainCam.projectionMatrix;
        // These should be nearly identical
    }

    // Manual perspective matrix construction (OpenGL convention)
    static Matrix4x4 BuildPerspectiveMatrix(float fovRad, float aspect,
                                             float near, float far)
    {
        float tanHalfFov = Mathf.Tan(fovRad * 0.5f);
        Matrix4x4 m = Matrix4x4.zero;

        m[0, 0] = 1f / (aspect * tanHalfFov);
        m[1, 1] = 1f / tanHalfFov;
        m[2, 2] = -(far + near) / (far - near);
        m[2, 3] = -(2f * far * near) / (far - near);
        m[3, 2] = -1f;

        return m;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What four parameters define a perspective projection matrix? :: **Vertical FOV** ($\theta$), **aspect ratio** ($a = W/H$), **near plane** ($n$), and **far plane** ($f$).
- Why does the perspective matrix put $-1$ in the bottom row? :: It copies $-z_\text{view}$ into the $w$ component of clip space. The subsequent perspective divide ($x/w, y/w$) divides by depth, creating perspective foreshortening — far objects become smaller.
- What happens when you decrease the FOV from 60° to 8°? :: The visible cone narrows dramatically, magnifying distant objects (zoom effect). The scene also appears "flatter" due to telephoto compression — depth differences are minimized visually.
- What is z-fighting and what causes it? :: Visible flickering between two surfaces that have nearly identical depth values. Caused by insufficient depth buffer precision, which is worsened by a large $f/n$ ratio or surfaces that are very far from the camera.
- What is reversed-Z and why is it used? :: A technique where $z_\text{ndc} = 1$ at the near plane and $0$ at the far plane. Combined with a floating-point depth buffer (which has more precision near 0), it distributes depth precision more evenly across the entire range.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Setting the near plane to a very small value (like $0.001$) "just to be safe" so nothing clips.
> - **The Fix:** Use the **largest near plane value you can tolerate** (typically $0.1$ to $1.0$ for most games). Push the near plane out as far as gameplay allows.
> - **Why:** Depth buffer precision degrades catastrophically with a small near plane. The ratio $f/n$ determines precision distribution: $f/n = 10{,}000{,}000$ (with $n=0.001, f=10{,}000$) means 99.9% of your depth buffer values are wasted in the first meter.

> [!danger] **Watch Out!**
> - **The Mistake:** Mixing up degrees and radians when constructing a custom projection matrix.
> - **The Fix:** Always convert FOV to radians before passing to $\tan()$. Use `Mathf.Deg2Rad` in Unity.
> - **Why:** $\tan(60)$ in radians vs degrees gives completely different results. In radians, $\tan(60) \approx 0.32$; in degrees, it should be $\tan(60° \times \pi/180) = \tan(1.047) \approx 1.73$.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/orthographic_projection|Orthographic Projection]]
- [[Math/05_Coordinate_Spaces/view_frustum|View Frustum]]
- [[Math/04_Matrices_Transforms/homogeneous_coordinates|Homogeneous Coordinates]]
