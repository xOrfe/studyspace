---
title: "Polar Coordinate Systems: Circular Space, Orbiting Cameras, and Radars"
tags:
  - math
  - level/Lv2
  - category/geometry_trigonometry
---

# Polar Coordinate Systems: Circular Space, Orbiting Cameras, and Radars

> [!abstract] **The Concept in a Nutshell**
> While **Cartesian coordinates** describe positions using a grid of axes ($x, y$), **Polar coordinates** describe positions using a distance (radius $r$) and an angle ($\theta$). In 3D space, this concept extends to **Cylindrical coordinates** ($r, \theta, z$) and **Spherical coordinates** ($r, \theta, \phi$). Polar coordinate systems are mathematically cleaner and far more intuitive for implementing anything based on circles or spheres: orbital cameras, radial menus, circular radars, and procedural circular patterns.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Implementing an Orbital Camera**
> You're writing a third-person camera that orbits around the player character. The player can look up, down, left, and right.
>
> If you try to calculate the camera's position using Cartesian coordinates ($x, y, z$), you have to manage complex offset offsets and vector rotations.
> However, in **Spherical coordinates**, the camera's position is simply:
> 1. Distance from the player (radius $r$).
> 2. Horizontal rotation angle (yaw $\theta$).
> 3. Vertical rotation angle (pitch $\phi$).
>
> When the player moves the mouse, you simply add mouse inputs to $\theta$ and $\phi$, and then convert the spherical coordinates back to Cartesian ($x, y, z$) to set the camera's position.

---

## The Blueprint (Formula & Structure)

### 2D Polar Coordinates $(r, \theta)$
A point is represented by:
- $r$: Radial distance from the origin (where $r \ge 0$).
- $\theta$: Angular direction from the positive X-axis.

```
          P(r, θ)
         /|
     r  / | 
       /  | y
      /___|
     O  x
```

- **Polar to Cartesian conversion:**
  $$x = r \cos\theta$$
  $$y = r \sin\theta$$

- **Cartesian to Polar conversion:**
  $$r = \sqrt{x^2 + y^2}$$
  $$\theta = \text{atan2}(y, x)$$

### 3D Cylindrical Coordinates $(r, \theta, z)$
Extends Polar coordinates by adding a vertical Cartesian height axis ($z$):
- $x = r \cos\theta$
- $y = r \sin\theta$
- $z = z$

### 3D Spherical Coordinates $(r, \theta, \phi)$
Defines a point on a sphere using a radius and two angles:
- $r$: Radial distance.
- $\theta$: Azimuth angle (rotation around the vertical Y-axis).
- $\phi$: Polar angle / elevation (angle from the vertical axis).

- **Conversion to Cartesian (using Y-up convention):**
  $$x = r \sin\phi \cos\theta$$
  $$y = r \cos\phi$$
  $$z = r \sin\phi \sin\theta$$

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Radar Sweep**
> Imagine a marine radar screen: the sweeper rotators display contacts by their heading (angle $\theta$) and distance (radius $r$) from the ship. 
> 
> Trying to build a radar screen in Cartesian coordinates would require updating $X$ and $Y$ via complex vector mathematics. In Polar coordinates, you simply rotate the angle $\theta$ and plot points directly using their natural range and heading.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Spherical coordinates for a third-person orbital camera
using UnityEngine;

public class OrbitalCamera : MonoBehaviour
{
    public Transform target; // The player to orbit
    public float radius = 5f; // Distance from target
    
    // Angles in radians
    private float theta = 0f; // Horizontal orbit angle (yaw)
    private float phi = Mathf.PI / 3f; // Vertical orbit angle (pitch)

    public float mouseSpeed = 2f;
    public float minPitch = 0.1f;
    public float maxPitch = Mathf.PI / 2.1f; // Avoid look-straight-down singularity

    void Update()
    {
        // Handle input
        theta += Input.GetAxis("Mouse X") * mouseSpeed * Time.deltaTime;
        phi -= Input.GetAxis("Mouse Y") * mouseSpeed * Time.deltaTime;

        // Clamp pitch to avoid screen flipping
        phi = Mathf.Clamp(phi, minPitch, maxPitch);

        // Convert Spherical to Cartesian (Y-up convention)
        float x = radius * Mathf.Sin(phi) * Mathf.Cos(theta);
        float y = radius * Mathf.Cos(phi);
        float z = radius * Mathf.Sin(phi) * Mathf.Sin(theta);

        Vector3 relativePos = new Vector3(x, y, z);

        // Position camera relative to target
        transform.position = target.position + relativePos;
        
        // Point camera at player
        transform.LookAt(target.position);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What parameters define 2D Polar coordinates? :: Radius $r$ (distance from origin) and angle $\theta$ (rotation).
- How do you convert 2D Polar coordinates $(r, \theta)$ to Cartesian coordinates $(x, y)$? :: $x = r \cos\theta$ and $y = r \sin\theta$.
- What coordinate system extends Polar coordinates into 3D by adding a standard vertical Z height axis? :: The **Cylindrical coordinate system** $(r, \theta, z)$.
- How do you calculate radius $r$ from Cartesian coordinates $(x, y)$? :: $r = \sqrt{x^2 + y^2}$ (which is the Pythagorean theorem / Euclidean distance).
- Why do orbital cameras use spherical coordinates? :: Spherical coordinates separate camera distance (radius) from camera orientation (yaw and pitch angles), making camera control code clean and robust.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Forgetting to clamp the vertical angle ($\phi$) in spherical coordinate cameras.
> - **The Fix:** Clamp the vertical angle between a value slightly above $0$ and slightly below $\pi$ (or $180^\circ$).
> - **Why:** If the camera goes exactly over the top (pole), the horizontal angle ($\theta$) becomes undefined (a singularity), causing the camera view to spin rapidly and glitch.

---

## Related Topics
- [[Math/01_Foundations/coordinate_systems_2d|2D Coordinate Systems]]
- [[Math/01_Foundations/coordinate_systems_3d|3D Coordinate Systems]]
- [[Math/02_Geometry_Trigonometry/trig_applications|Trigonometry in Game Dev]]
