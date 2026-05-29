---
title: "Rigid Body Dynamics: Mass, Center of Mass, and Inertia Tensors"
tags:
  - math
  - level/Lv4
  - category/physics_math
---

# Rigid Body Dynamics: Mass, Center of Mass, and Inertia Tensors

> [!abstract] **The Concept in a Nutshell**
> While particle physics treats objects as infinitely small points with no volume, real game objects are **rigid bodies** — shapes with dimensions, distributed mass, and rotation. Rigid body dynamics introduces three critical properties: the **Center of Mass** (the balance point of the shape), **Torque** (forces that cause rotation), and the **Moment of Inertia Tensor** (a 3x3 matrix representing an object's resistance to rotating along different axes).

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: A Falling Wooden Crate Bouncing Off a Ledge**
> Imagine a crate falling in your game. It clips the edge of a platform.
>
> If you only simulate linear physics, the crate will slide off the platform without rotating, looking like a sliding box on ice. 
> To look realistic, the collision force must hit the crate offset from its center of mass. This offset force creates a rotational force called **torque**.
>
> How fast the crate spins depends on its **inertia tensor**. A long, thin metal beam resists spinning much more than a compact wooden box of the same weight. By calculating this rotational resistance using an inertia tensor, we get realistic tumbling, tipping, and spinning.

---

## The Blueprint (Formula & Structure)

### 1. Center of Mass
The average position of all the mass in a rigid body. For a system of discrete particles:
$$\vec{C}_{\text{com}} = \frac{\sum m_i \vec{r}_i}{\sum m_i}$$
For standard convex colliders (cubes, spheres, cylinders), the center of mass is typically assumed to be the geometric center of the shape.

### 2. Torque ($\vec{\tau}$)
Torque is the rotational equivalent of linear force. It is generated when a force $\vec{F}$ is applied at an offset vector $\vec{r}$ from the center of mass:
$$\vec{\tau} = \vec{r} \times \vec{F}$$
- **Units:** Newton-meters ($\text{N}\cdot\text{m}$).
- Winding direction is determined by the right-hand rule.

### 3. Moment of Inertia Tensor ($I$)
The inertia tensor is a 3x3 matrix representing an object's resistance to angular acceleration.
$$I = \begin{bmatrix}
I_{xx} & I_{xy} & I_{xz} \\
I_{yx} & I_{yy} & I_{yz} \\
I_{zx} & I_{zy} & I_{zz}
\end{bmatrix}$$
For symmetric shapes aligned with their local axes, the diagonal elements ($I_{xx}, I_{yy}, I_{zz}$) represent the principal moments of inertia, and the off-diagonal elements are $0$.

#### Principal Inertia Tensors for Common Colliders (Mass $m$)
- **Sphere (Radius $R$):**
  $$I_{xx} = I_{yy} = I_{zz} = \frac{2}{5} m R^2$$
- **Cuboid (Box with width $w$, height $h$, depth $d$):**
  $$I_{xx} = \frac{1}{12} m (h^2 + d^2), \quad I_{yy} = \frac{1}{12} m (w^2 + d^2), \quad I_{zz} = \frac{1}{12} m (w^2 + h^2)$$

### 4. Angular Momentum ($\vec{L}$)
The rotational equivalent of linear momentum:
$$\vec{L} = I \vec{\omega}$$
Where $\vec{\omega}$ is the angular velocity vector. In the absence of external torque, angular momentum is conserved.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Ice Skater**
> Think of the inertia tensor as the ice skater's arm span.
> - When the skater pulls their arms in close to their body, their mass is clustered near the rotation axis. Rotational resistance decreases (lower moment of inertia), causing them to spin incredibly fast.
> - When they stretch their arms out, their mass moves far from the center. Rotational resistance increases, slowing their spin down.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Simulating custom Center of Mass and applying Torque
using UnityEngine;

public class CustomRigidBody : MonoBehaviour
{
    public Vector3 centerOfMassOffset = Vector3.zero;
    public Vector3 inertiaTensorDiagonal = new Vector3(1f, 1f, 1f);

    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb != null)
        {
            // Shift the center of mass (e.g. for a racing car, to keep it grounded)
            rb.centerOfMass = centerOfMassOffset;
            
            // Set the inertia tensor diagonal values manually
            // Useful to make objects resist tipping over along specific axes
            rb.inertiaTensor = inertiaTensorDiagonal;
        }
    }

    // Apply force at an offset position relative to center of mass
    public void ApplyForceAtOffset(Vector3 force, Vector3 worldApplyPoint)
    {
        Vector3 comPosition = transform.TransformPoint(rb.centerOfMass);
        Vector3 r = worldApplyPoint - comPosition; // Offset vector

        // Calculate Torque: tau = r x F
        Vector3 torque = Vector3.Cross(r, force);

        // Apply to Unity Rigidbody
        rb.AddForce(force);
        rb.AddTorque(torque);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is torque? :: The rotational equivalent of force, computed as the cross product of the offset vector and the force vector: $\vec{\tau} = \vec{r} \times \vec{F}$.
- What is the Moment of Inertia Tensor? :: A 3x3 matrix representing an object's resistance to angular acceleration along different rotation axes.
- How does shifting the center of mass downward affect a vehicle's stability in a racing game? :: It lowers the center of mass, reducing the torque generated during sharp turns and preventing the vehicle from flipping over.
- What is the formula for the principal moments of inertia of a sphere of mass $m$ and radius $R$? :: $I = \frac{2}{5}mR^2$.
- Write the relationship between torque, inertia tensor, and angular acceleration. :: $\vec{\tau} = I \vec{\alpha}$ (where $\vec{\alpha}$ is the angular acceleration vector).

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Using local-space inertia tensors directly in world-space calculations.
> - **The Fix:** Transform the inverse inertia tensor to world space each frame using the object's rotation matrix $R$: $I_{\text{world}}^{-1} = R I_{\text{local}}^{-1} R^T$.
> - **Why:** The resistance to rotation is relative to the object's orientation. If you don't rotate the inertia tensor alongside the object, spinning physics will act as if the object never rotated, leading to unnatural wobble and jerky physics.

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/10_Physics_Math/newtonian_dynamics|Newtonian Dynamics]]
- [[Math/10_Physics_Math/rotational_dynamics|Rotational Dynamics]]
