---
title: "View Frustum: The Camera's Invisible Boundary"
tags:
  - math
  - level/Lv3
  - category/coordinate_spaces
---

# View Frustum: The Camera's Invisible Boundary

> [!abstract] **The Concept in a Nutshell**
> The **view frustum** is the 3D volume that represents everything a camera can see — shaped like a truncated pyramid for perspective cameras, or a box for orthographic cameras. It is bounded by **6 planes**: near, far, left, right, top, bottom. **Frustum culling** — testing whether objects lie inside this volume — is one of the most impactful performance optimizations in real-time rendering, allowing the engine to skip drawing thousands of invisible objects every frame.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Sprawling Open-World City**
> Your open-world game has 50,000 objects in the city: buildings, cars, NPCs, street lights, trash cans. The player's camera can only see maybe 2,000 of them at any moment. Without frustum culling, the GPU would process all 50,000 objects — vertices, shaders, textures — only to discover most of them are behind the camera or off to the side.
>
> With frustum culling, the engine tests each object's bounding volume against the 6 frustum planes *before* sending anything to the GPU. The building behind the camera? Culled. The skyscraper 10km away beyond the far plane? Culled. The hot dog cart off-screen to the left? Culled. Only the ~2,000 actually visible objects get rendered.
>
> This single optimization can turn a 10 FPS slideshow into a smooth 60 FPS experience. Unity, Unreal, and every serious engine do frustum culling automatically, but understanding the math lets you write custom culling for particles, LOD systems, and occlusion queries.

---

## The Blueprint (Formula & Structure)

### Frustum Shape

A perspective frustum has 6 bounding planes forming a truncated pyramid:

```
        Near Plane (small rectangle)
       /    |    \
      /     |     \
     /      |      \    ← Left, Right, Top, Bottom planes
    /       |       \      are the slanted sides
   /________|________\
     Far Plane (large rectangle)
```

### Representing a Plane

Each frustum plane is defined in the form:

$$ax + by + cz + d = 0$$

Where $\hat{\mathbf{n}} = (a, b, c)$ is the **inward-facing normal** (pointing toward the interior of the frustum) and $d$ is the signed distance from the origin.

### Extracting Frustum Planes from the VP Matrix

Given the combined **View-Projection** matrix $\mathbf{VP}$, with rows $\mathbf{r}_0, \mathbf{r}_1, \mathbf{r}_2, \mathbf{r}_3$ (where each row is a 4-component vector):

| Plane | Extraction Formula |
|---|---|
| **Left** | $\mathbf{r}_3 + \mathbf{r}_0$ |
| **Right** | $\mathbf{r}_3 - \mathbf{r}_0$ |
| **Bottom** | $\mathbf{r}_3 + \mathbf{r}_1$ |
| **Top** | $\mathbf{r}_3 - \mathbf{r}_1$ |
| **Near** | $\mathbf{r}_3 + \mathbf{r}_2$ |
| **Far** | $\mathbf{r}_3 - \mathbf{r}_2$ |

Each result gives $(a, b, c, d)$. **Normalize** by dividing all four components by $\|\hat{\mathbf{n}}\| = \sqrt{a^2 + b^2 + c^2}$ to get true distances.

### Frustum Tests

**Point vs Frustum:**
A point $\mathbf{p}$ is inside the frustum if and only if it is on the **positive side** of all 6 planes:

$$\hat{\mathbf{n}}_i \cdot \mathbf{p} + d_i \geq 0 \quad \text{for all } i \in \{0..5\}$$

If any plane gives a negative result, the point is outside.

**Sphere vs Frustum:**
A sphere with center $\mathbf{c}$ and radius $r$ is outside the frustum if, for any plane:

$$\hat{\mathbf{n}}_i \cdot \mathbf{c} + d_i < -r$$

It is fully inside if:

$$\hat{\mathbf{n}}_i \cdot \mathbf{c} + d_i \geq r \quad \text{for all } i$$

Otherwise, it **intersects** the frustum boundary.

**AABB vs Frustum (N/P vertex test):**
For each frustum plane, find the AABB vertex most in the direction of the plane normal (**P-vertex**) and the vertex most opposite (**N-vertex**):

1. If the **P-vertex** is outside the plane → the entire AABB is outside → **cull**.
2. If the **N-vertex** is inside → the AABB is fully inside this plane.
3. Otherwise → the AABB intersects this plane.

If the AABB passes all 6 planes, it's (at least partially) visible.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Looking Through a Cardboard Tube**
> Hold an empty paper towel tube up to your eye. What you see through it is your **frustum** (well, a cylindrical version).
>
> - **Near plane:** The opening closest to your eye — things pressed against it are visible but weird.
> - **Far plane:** The far end of the tube — beyond it, nothing exists.
> - **Side planes:** The tube walls — anything outside the tube's cone of vision is invisible.
>
> Now imagine the tube is a truncated **pyramid** instead of a cylinder (wider at the far end). That's the perspective frustum. Objects must fit inside this pyramid to be rendered.
>
> **Frustum culling** is simply asking: "Is this object inside the tube?" If not, don't bother drawing it. For complex scenes, this question alone eliminates 80-95% of unnecessary work.

---

## Code Example (Applied in Engine)
```csharp
// Unity C# — Manual frustum culling and plane extraction
using UnityEngine;

public class FrustumCullingDemo : MonoBehaviour
{
    public Camera cam;
    public Transform[] testObjects;

    // The 6 frustum planes
    private Plane[] frustumPlanes = new Plane[6];

    void Update()
    {
        // === METHOD 1: Unity's built-in plane extraction ===
        // Extracts the 6 planes from the camera's VP matrix
        GeometryUtility.CalculateFrustumPlanes(cam, frustumPlanes);

        // Test each object's bounding volume against the frustum
        foreach (var obj in testObjects)
        {
            Renderer rend = obj.GetComponent<Renderer>();
            if (rend == null) continue;

            // Test the renderer's axis-aligned bounding box
            bool isVisible = GeometryUtility.TestPlanesAABB(
                frustumPlanes, rend.bounds);

            // Enable/disable rendering based on visibility
            rend.enabled = isVisible;

            if (isVisible)
                Debug.DrawLine(cam.transform.position,
                    obj.position, Color.green);
            else
                Debug.DrawLine(cam.transform.position,
                    obj.position, Color.red);
        }

        // === METHOD 2: Manual sphere-vs-frustum test ===
        Vector3 sphereCenter = new Vector3(10f, 2f, 5f);
        float sphereRadius = 3f;

        bool sphereVisible = IsSphereInFrustum(
            sphereCenter, sphereRadius, frustumPlanes);
        Debug.Log($"Sphere visible: {sphereVisible}");
    }

    /// <summary>
    /// Test if a bounding sphere is at least partially inside the frustum.
    /// </summary>
    bool IsSphereInFrustum(Vector3 center, float radius, Plane[] planes)
    {
        for (int i = 0; i < 6; i++)
        {
            // Plane.GetDistanceToPoint returns SIGNED distance
            // Positive = same side as normal, Negative = opposite side
            float distance = planes[i].GetDistanceToPoint(center);

            // If the sphere is entirely on the negative side of any plane,
            // it's completely outside the frustum
            if (distance < -radius)
                return false;
        }
        return true; // Passed all 6 planes
    }

    /// <summary>
    /// Manual frustum plane extraction from VP matrix (educational).
    /// </summary>
    Plane[] ExtractFrustumPlanes(Matrix4x4 vp)
    {
        Plane[] planes = new Plane[6];

        // Left:   row3 + row0
        planes[0] = new Plane(
            new Vector3(vp.m30 + vp.m00, vp.m31 + vp.m01, vp.m32 + vp.m02),
            vp.m33 + vp.m03);

        // Right:  row3 - row0
        planes[1] = new Plane(
            new Vector3(vp.m30 - vp.m00, vp.m31 - vp.m01, vp.m32 - vp.m02),
            vp.m33 - vp.m03);

        // Bottom: row3 + row1
        planes[2] = new Plane(
            new Vector3(vp.m30 + vp.m10, vp.m31 + vp.m11, vp.m32 + vp.m12),
            vp.m33 + vp.m13);

        // Top:    row3 - row1
        planes[3] = new Plane(
            new Vector3(vp.m30 - vp.m10, vp.m31 - vp.m11, vp.m32 - vp.m12),
            vp.m33 - vp.m13);

        // Near:   row3 + row2
        planes[4] = new Plane(
            new Vector3(vp.m30 + vp.m20, vp.m31 + vp.m21, vp.m32 + vp.m22),
            vp.m33 + vp.m23);

        // Far:    row3 - row2
        planes[5] = new Plane(
            new Vector3(vp.m30 - vp.m20, vp.m31 - vp.m21, vp.m32 - vp.m22),
            vp.m33 - vp.m23);

        // Normalize all planes
        for (int i = 0; i < 6; i++)
        {
            float mag = planes[i].normal.magnitude;
            planes[i] = new Plane(planes[i].normal / mag,
                                   planes[i].distance / mag);
        }

        return planes;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What shape is a perspective view frustum? :: A **truncated pyramid** (also called a frustum) — narrow at the near plane, wide at the far plane. It has 6 bounding planes: near, far, left, right, top, bottom.
- How do you extract the 6 frustum planes from the VP matrix? :: Combine the rows of the View-Projection matrix: Left = $\mathbf{r}_3 + \mathbf{r}_0$, Right = $\mathbf{r}_3 - \mathbf{r}_0$, Bottom = $\mathbf{r}_3 + \mathbf{r}_1$, Top = $\mathbf{r}_3 - \mathbf{r}_1$, Near = $\mathbf{r}_3 + \mathbf{r}_2$, Far = $\mathbf{r}_3 - \mathbf{r}_2$. Then normalize each plane.
- How do you test if a sphere is outside the frustum? :: Compute the signed distance from the sphere center to each of the 6 planes. If any plane gives a distance $< -r$ (where $r$ is the radius), the entire sphere is on the outside of that plane → **cull it**.
- What is the N-vertex/P-vertex technique for AABB frustum testing? :: For each frustum plane, find the AABB corner **most aligned** with the plane normal (P-vertex) and **least aligned** (N-vertex). If the P-vertex is outside → AABB is fully outside. If N-vertex is inside → AABB is fully inside that plane's half-space.
- Why is frustum culling important for performance? :: It prevents the GPU from processing objects that aren't visible. In a typical scene, **80-95%** of objects may be off-screen or behind the camera. Culling them before GPU submission saves massive amounts of vertex processing, rasterization, and shading work.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Implementing frustum culling but forgetting to normalize the extracted planes, resulting in incorrect distance calculations and objects popping in/out at wrong distances.
> - **The Fix:** After extracting each plane $(a, b, c, d)$, divide all four components by $\sqrt{a^2 + b^2 + c^2}$. This ensures `GetDistanceToPoint` returns the actual Euclidean distance.
> - **Why:** The Gribb-Hartmann extraction gives un-normalized planes. The signed distance formula only returns true distances when the normal is a unit vector.

> [!danger] **Watch Out!**
> - **The Mistake:** Assuming that passing all 6 plane tests guarantees an AABB is truly visible. There are **false positives** where the AABB passes all plane tests but is actually outside a corner of the frustum.
> - **The Fix:** For most games, accept the rare false positive (object gets drawn unnecessarily — minor cost). For extreme precision, add corner/edge tests, but this is rarely worth the CPU cost.
> - **Why:** The 6-plane test is a necessary but not sufficient condition. An AABB can straddle all planes yet miss the actual frustum volume at the corners.

---

## Related Topics
- [[Math/05_Coordinate_Spaces/perspective_projection|Perspective Projection]]
- [[Math/07_Geometric_Primitives/bounding_volumes|Bounding Volumes]]
- [[Math/07_Geometric_Primitives/planes_implicit|Planes (Implicit Form)]]
