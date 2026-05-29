---
title: "Rotational Dynamics: Angular Motion, Precession, and Torque Integration"
tags:
  - math
  - level/Lv4
  - category/physics_math
---

# Rotational Dynamics: Angular Motion, Precession, and Torque Integration

> [!abstract] **The Concept in a Nutshell**
> Rotational dynamics describes how torque changes an object's rotation over time. Unlike linear motion (where velocity changes along simple axes), rotational motion is non-linear and much more complex because rotating objects constantly redirect their own axes of rotation. Mastering this mathematics is key to simulating realistic spinning tops, gyroscopic stabilizers, tumbling debris, and aircraft maneuvers.

---

## Game Dev Context (Why This Matters)
> [!example] **Scene: Simulating a Spinning Top (Precession)**
> In an adventure game, a player spins a metallic top on a table. As it slows down, it doesn't just stop instantly; it starts to wobble, trace a circular path with its tip, and slowly tip over.
>
> If you just rotate the top around the Y-axis, it will look like a rotating billboard. 
> Real wobbling is caused by **precession** — a physical phenomenon where gravity applies torque to the spinning axis, pushing it sideways instead of directly tipping it over.
>
> To capture this effect, the physics engine must integrate angular velocity using the 3D rigid-body Euler equations:
> $$\vec{\tau} = I\vec{\alpha} + \vec{\omega} \times (I\vec{\omega})$$
> The term $\vec{\omega} \times (I\vec{\omega})$ represents gyroscopic forces. Without it, wobbling, tumbling, and gyroscopic stabilization cannot be simulated.

---

## The Blueprint (Formula & Structure)

### 1. Rotational Kinematics Variables
- **Orientation:** Represented by a rotation matrix $R$ or a Quaternion $q$.
- **Angular Velocity ($\vec{\omega}$):** A vector whose direction is the axis of rotation and magnitude is the speed of rotation (in radians/sec).
- **Angular Acceleration ($\vec{\alpha}$):** Rate of change of angular velocity:
  $$\vec{\alpha} = \frac{d\vec{\omega}}{dt}$$

### 2. Rotational Euler Equations of Motion
Newton's Second Law for rotation in the body-local frame is:
$$\vec{\tau} = I \vec{\alpha} + \vec{\omega} \times (I \vec{\omega})$$

To find the angular acceleration vector $\vec{\alpha}$ to integrate:
$$\vec{\alpha} = I^{-1} \left( \vec{\tau} - \vec{\omega} \times (I \vec{\omega}) \right)$$

Where:
- $\vec{\tau}$: Net torque vector.
- $I$: 3x3 Inertia tensor.
- $\vec{\omega} \times (I \vec{\omega})$: **Coriolas/Gyroscopic term** (representing torque created by internal rotation axes crossing).

### 3. Integrating Rotation (Quaternions)
Once $\vec{\omega}$ is updated using $\vec{\alpha}$, we update the object's orientation quaternion $q$:
$$\frac{dq}{dt} = \frac{1}{2} \vec{\omega}_q \otimes q$$
Where $\vec{\omega}_q = (0, \omega_x, \omega_y, \omega_z)$ is the angular velocity formatted as a pure quaternion, and $\otimes$ represents quaternion multiplication.

---

## Building Intuition (Visual Understanding)
> [!tip] **Mental Model: The Right-Hand Rule thumb**
> Think of angular velocity as a rotating axle. 
> - If you curl the fingers of your right hand in the direction of the rotation, your thumb points along the **angular velocity vector** ($\vec{\omega}$).
> - If you try to tilt a fast-spinning wheel, it resists you and deflects at a right angle to your push. This is gyroscopic deflection, caused by the cross product of the rotation vector and the torque.

---

## Code Example (Applied in Engine)

```csharp
// Unity C# — Custom Rotational Physics Integration
using UnityEngine;

public class RotationalIntegrator : MonoBehaviour
{
    public Vector3 inertiaTensor = new Vector3(1f, 1f, 1f); // Local diagonal inertia
    public Vector3 angularVelocity; // in radians/sec
    public Vector3 torque;

    private Quaternion orientation;

    void Start()
    {
        orientation = transform.rotation;
    }

    void FixedUpdate()
    {
        // 1. Calculate inverse inertia tensor in local space
        Vector3 invInertia = new Vector3(1f / inertiaTensor.x, 1f / inertiaTensor.y, 1f / inertiaTensor.z);

        // 2. Compute gyroscopic torque: w x (I * w)
        Vector3 I_w = new Vector3(
            angularVelocity.x * inertiaTensor.x,
            angularVelocity.y * inertiaTensor.y,
            angularVelocity.z * inertiaTensor.z
        );
        Vector3 gyroscopicTorque = Vector3.Cross(angularVelocity, I_w);

        // 3. Compute angular acceleration (local frame)
        // alpha = I^-1 * (tau - w x Iw)
        Vector3 netLocalTorque = torque - gyroscopicTorque;
        Vector3 angularAcceleration = new Vector3(
            netLocalTorque.x * invInertia.x,
            netLocalTorque.y * invInertia.y,
            netLocalTorque.z * invInertia.z
        );

        // 4. Update angular velocity: w = w + alpha * dt
        angularVelocity += angularAcceleration * Time.fixedDeltaTime;

        // 5. Update orientation using Quaternion derivative: dq/dt = 0.5 * w * q
        Quaternion wq = new Quaternion(angularVelocity.x, angularVelocity.y, angularVelocity.z, 0f);
        Quaternion dq = MultiplyQuaternions(wq, orientation);
        
        // Integrate orientation: q = q + dq * dt
        orientation.x += 0.5f * dq.x * Time.fixedDeltaTime;
        orientation.y += 0.5f * dq.y * Time.fixedDeltaTime;
        orientation.z += 0.5f * dq.z * Time.fixedDeltaTime;
        orientation.w += 0.5f * dq.w * Time.fixedDeltaTime;

        orientation = NormalizeQuaternion(orientation);
        transform.rotation = orientation;

        // Clear torque
        torque = Vector3.zero;
    }

    // Helper to multiply pure quaternion by orientation
    private Quaternion MultiplyQuaternions(Quaternion a, Quaternion b)
    {
        return new Quaternion(
            a.w * b.x + a.x * b.w + a.y * b.z - a.z * b.y,
            a.w * b.y - a.x * b.z + a.y * b.w + a.z * b.x,
            a.w * b.z + a.x * b.y - a.y * b.x + a.z * b.w,
            a.w * b.w - a.x * b.x - a.y * b.y - a.z * b.z
        );
    }

    private Quaternion NormalizeQuaternion(Quaternion q)
    {
        float len = Mathf.Sqrt(q.x * q.x + q.y * q.y + q.z * q.z + q.w * q.w);
        return new Quaternion(q.x / len, q.y / len, q.z / len, q.w / len);
    }
}
```

---

## Interactive Flashcards (Spaced Repetition)
- Write the rigid-body Euler equation of rotation. :: $\vec{\tau} = I \vec{\alpha} + \vec{\omega} \times (I \vec{\omega})$.
- What does the term $\vec{\omega} \times (I\vec{\omega})$ represent? :: Gyroscopic torque, which is the internal torque produced when an object rotates simultaneously on multiple axes.
- What direction does the angular velocity vector point for a wheel spinning clockwise in the XY plane? :: Into the screen (along the negative Z-axis, according to the right-hand rule).
- How do you update an orientation quaternion $q$ using angular velocity $\vec{\omega}$? :: Calculate the derivative $\frac{dq}{dt} = \frac{1}{2}\vec{\omega}_q \otimes q$ and add it to the current quaternion.
- What is precession? :: The slow, circular wobble of the rotation axis of a spinning body when torque (like gravity) acts on it.

---

## Common Pitfalls (The Trap)
> [!danger] **Watch Out!**
> - **The Mistake:** Storing orientation in Euler angles (pitch, yaw, roll) and integrating angular velocity by adding values to them directly.
> - **The Fix:** Always integrate rotation using Quaternions or Rotation Matrices, converting to Euler angles only when displaying values in the editor.
> - **Why:** Integrating Euler angles directly creates gimbal lock (where you lose a degree of freedom) and generates highly distorted, unnatural rotational acceleration.

---

## Related Topics
- [[Math/06_Rotations_Orientation/quaternion_rotations|Quaternion Rotations]]
- [[Math/10_Physics_Math/rigid_body_dynamics|Rigid Body Dynamics]]
- [[Math/09_Calculus_for_Games/numerical_integration|Numerical Integration Methods]]
