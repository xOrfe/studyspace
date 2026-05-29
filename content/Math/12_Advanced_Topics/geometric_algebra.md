---
title: "Geometric Algebra: Rotors, Bivectors, and Next-Gen Engine Mathematics"
tags:
  - math
  - level/Lv4
  - category/advanced_topics
---

# Geometric Algebra: Rotors, Bivectors, and Next-Gen Engine Mathematics

> [!abstract] **The Concept in a Nutshell**
> Geometric Algebra (GA), or Clifford Algebra, is an emerging mathematical framework that unifies vectors, complex numbers, quaternions, and matrix transformations into a single coordinate-free system. Instead of treating vectors (1D lines) and quaternions (4D rotation complexes) as completely separate systems, GA introduces **multivectors** like **bivectors** (oriented 2D planes) and **rotors** (which generalize rotations to any dimension). This is the next-generation math framework used in cutting-edge physics and rendering engines.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Replacing Complex Vector-Quarternion-Matrix Math**
> In traditional 3D graphics, you use:
> - Vectors for positions and velocities.
> - Quaternions for 3D rotations.
> - Plücker coordinates for lines.
> - 4x4 Matrices for projection and affine transformations.
>
> Managing conversions between these four formats introduces mathematical complexity and bugs. 
> In **Geometric Algebra**, all of these primitives are represented as different parts of a single algebraic object: the **multivector**.
> - Rotating a vector $\vec{v}$ no longer requires converting to a quaternion, applying sandwich multiplication, and converting back.
> - You define a **rotor** $R$ and rotate any geometric entity (a point, a line, or a plane) using the exact same formula:
>   $$\vec{v}' = R \vec{v} R^{\dagger}$$
> This unifies physics engine math, making code cleaner and more efficient.

---

## The Blueprint (Formula & Structure)

### 1. The Wedge Product ($\wedge$) and Bivectors
In standard vector algebra, we use the cross product ($\vec{a} \times \vec{b}$) to find a perpendicular vector in 3D. However, the cross product does not exist in 2D or dimensions higher than 3D.
GA introduces the **wedge product** ($\vec{a} \wedge \vec{b}$). The result is a **bivector** — a directed segment of a 2D plane:
- Magnitude represents the area of the parallelogram spanned by $\vec{a}$ and $\vec{b}$.
- Winding direction represents the orientation (clockwise or counter-clockwise).

```
         ^  a
         |    
         |----> b
        ( bivector plane segment )
```

### 2. The Geometric Product
The core operation of Clifford algebra, defined as the sum of the inner (dot) product and outer (wedge) product:
$$\vec{a}\vec{b} = \vec{a} \cdot \vec{b} + \vec{a} \wedge \vec{b}$$
- $\vec{a} \cdot \vec{b}$: A scalar (real number).
- $\vec{a} \wedge \vec{b}$: A bivector (directed plane segment).

### 3. Rotors ($R$)
A **rotor** is the geometric algebra generalization of quaternions. It rotates objects in the plane defined by a bivector $B$ by an angle $\theta$:
$$R = e^{\frac{\theta}{2} B} = \cos\left(\frac{\theta}{2}\right) - \sin\left(\frac{\theta}{2}\right) B$$
Where $B$ is a unit bivector ($B^2 = -1$).
- Rotors perform rotations in *any* dimension (2D, 3D, 4D) without modifying the formula.
- In 3D space, rotors are isomorphic (mathematically identical) to quaternions, but they offer a clear geometric meaning: instead of rotating around an abstract axis, you rotate *within a physical plane* ($B$).

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: Dimensions of Space**
> Geometric Algebra organizes geometric entities by their grade (dimensionality):
> - **Grade 0 (Scalars):** Points with magnitude, no direction (1D number line).
> - **Grade 1 (Vectors):** Directed line segments (arrows).
> - **Grade 2 (Bivectors):** Directed plane segments (sheets of paper).
> - **Grade 3 (Trivectors):** Directed volume segments (boxes).
> 
> A multivector is a composite object that can hold a scalar, a vector, and a bivector simultaneously, representing a complete coordinate frame.

---

## Code Example (Applied in Engine)

```csharp
// Outline representation of a 2D/3D Multivector in Geometric Algebra
using UnityEngine;

public struct Multivector
{
    // Grade 0
    public float scalar;
    
    // Grade 1 (Vector)
    public Vector3 vector;
    
    // Grade 2 (Bivector components in 3D: xy, yz, zx)
    public float b_xy;
    public float b_yz;
    public float b_zx;
    
    // Grade 3 (Trivector / Pseudoscalar)
    public float pseudoscalar;

    // Geometric Product of two vectors: ab = a.b + a^b
    public static Multivector GeometricProduct(Vector3 a, Vector3 b)
    {
        Multivector mv = new Multivector();
        
        // Scalar part (dot product)
        mv.scalar = Vector3.Dot(a, b);
        
        // Bivector part (wedge product components)
        mv.b_xy = a.x * b.y - a.y * b.x;
        mv.b_yz = a.y * b.z - a.z * b.y;
        mv.b_zx = a.z * b.x - a.x * b.z;
        
        return mv;
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- What is the geometric product of two vectors? :: The sum of their dot product and wedge product: $\vec{a}\vec{b} = \vec{a} \cdot \vec{b} + \vec{a} \wedge \vec{b}$.
- What is a bivector? :: An oriented 2D plane segment representing an area and direction of rotation, generated by the wedge product of two vectors.
- What is a rotor in Geometric Algebra? :: The generalization of quaternions to any dimension, representing rotation within a plane defined by a bivector.
- Why is Geometric Algebra considered unified next-gen math for games? :: It integrates vectors, quaternions, and matrices into a single coordinate-free algebraic object (the multivector), simplifying physics and kinematics.
- What does the wedge product $\vec{a} \wedge \vec{b}$ represent? :: The oriented plane area spanned by vectors $\vec{a}$ and $\vec{b}$.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Treating bivectors as simple 3D vector cross products.
> - **The Fix:** Remember that bivectors are planes, not normal vectors.
> - **Why:** In 3D, a plane can be represented by its normal vector (duality). However, this duality breaks in 2D and 4D space. Using bivectors directly ensures your rotation and physics code scales to 2D screen space or 4D shader spaces without modifications.

---

## Related Topics
- [[Math/03_Vectors/cross_product|Cross Product]]
- [[Math/06_Rotations_Orientation/quaternion_fundamentals|Quaternion Fundamentals]]
- [[Math/06_Rotations_Orientation/quaternion_rotations|Quaternion Rotations]]
