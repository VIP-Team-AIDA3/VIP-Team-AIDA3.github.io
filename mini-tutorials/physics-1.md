<!-- Mini-Tutorial 1 — Newton's Laws and Point-Mass Models

Question: How do physical systems move?

Topics:

Why mathematical models?
Scalars and vectors (brief)
Position, velocity, acceleration
Newton's three laws
Point-mass assumption
Free-body diagrams
Free mass
Mass–damper
Mass–spring–damper
Deriving equations of motion

Notebook:

Simulate the three systems.
Vary m, c, and k. -->

# Physics refresher 1: Newton's Laws and Point-Mass Motion
This tutorial provides the foundations for physics-informed machine learning to solve problems related to autonomous systems such as motion-planing, control, system identification, as well as digital twinning. For some of the students, such a refresher on the basics of physics might be helpful to understand the forces that that constitute the dynamics of a drone, an aicraft, or other rigid bodies. We will not cover aerodyanmics or other fields of physics but start with the basics. The basic goal of this tutorial is to answer the following questions:

> How do physical systems move? 
> How can Newton's Law of Motion help us derive equations of motions, based in simplified assumptions of a "point-mass? 

By end of this tutorial you should be able to:

1. Refresh your memory on Newton's three laws of motion. 
2. Be able to distinguish between the study of kinematics from the study of kinetics, and dynamics (e.g. aerodynamics)
3. Clarify the assumption of a point-mass and (its limitations)
4. Derive the equations of motions for a free mass, and family of forced and unforced mass-damper systems
5. Simulate how different physical parameters effect the stability of a system. 

## Vectors versus scalars

In order to understand equations of motions, it is important that you can distinguish between scalars and vectors, and how we represent them. Vectors are essential in Newton mechanics to derive the equations of motions. The key mathematical distinction is:

$$\text{scalar} \in \mathbb{R} \quad \text{versus} \quad \text{vector} \in \mathbb{R}^n$$

#### 1 A scalar 
A scalar is a single numerical value. Formally, a real-valued physical quantity x, is expressed as

$$ x \in \mathcal{R}$$

For example, the mass of an object is a scalar:

$$m \in \mathbb{R}_{>0}$$

If $m = 2 \text{ kg}$, then $m$ specifies a magnitude but has no spatial direction. 

Other scalar quantities include time, temperature, speed, distance, and energy:

$$t \in \mathbb{R}_{\ge0}, \quad T \in \mathbb{R}, \quad v_{\text{speed}} \in \mathbb{R}_{\ge0}$$

Strictly speaking, physical quantities carry units, so $m = 2 \text{ kg}$ is not literally just the real number $2$. In an introductory treatment, however, it is standard to represent the numerical value of a scalar physical quantity as an element of $\mathbb{R}$, with its unit stated separately.

#### 2. Vectors
A vector is an element of a vector space. For mechanics, the relevant vector space is usually Euclidean space:

$$\mathbf{v} \in \mathbb{R}^n$$

For three-dimensional motion:

$$\mathbf{v} \in \mathbb{R}^3$$

It can be represented in Cartesian coordinates as:

$$\mathbf{v} = \begin{bmatrix} v_x \\ v_y \\ v_z \end{bmatrix}$$

Each component is itself a scalar:

$$v_x, v_y, v_z \in \mathbb{R}$$

A vector has both a magnitude and a direction. Its Euclidean magnitude is:

$$\|\mathbf{v}\|_2 = \sqrt{v_x^2 + v_y^2 + v_z^2}$$

> Important: The $L_2$ norm $\|\mathbf{v}\|_2$ is a **scalar** rather than a vector and describes the lengths of the vector. 

For example, below you see the following vector: 

```python
import numpy as np
import matplotlib.pyplot as plt

startpoint = np.array([0.0, 0.0, 0.0])
endpoint = np.array([0.0, 3.0, 5.0])
v = endpoint - startpoint
```
And plot it with matplotlib, you can see what the vector looks like. 

In order to distinguish vectors from scalars, we can adopt different notations. We use **bold**, lowercase notations like "$\mathbf{v}$" rather than "$\vec{v}$" when referring to vectors, and would use simple v for a scalar.  

![Alt text](/mini-tutorials/figures/vector-physics.png)

So keep in mind that some physical quantities are vectors, and thus, have both direction and quantity. 

## Kinematics

Kinematics describes motion without asking what causes it. 
Please note that kinematics only describes the geometry of motion. It is distinct from dynamics (often split into kinetics and kinematics), laws and theories that seek to relate motion directly to the forces, torques, and masses that cause it. We turn to Newton's law of motions to relate motion to forces after explaining three kinematics concepts of relevance for us: 1. position, 2. velocity, and 3. acceleration. 

#### 1. Position

Formally, a **position** is a point in Euclidean space, whereas **displacement** is a vector. Once we choose an origin and a coordinate frame, we represent the position of a point $P$ by its position vector:

$$\mathbf{p}^W = \begin{bmatrix} p_x \\ p_y \\ p_z \end{bmatrix} \in \mathbb{R}^3$$

The superscript $W$ indicates that its coordinates are expressed relative to the world frame $W$. The world frame is related to a fixed point on the earth. We will turn other frames such as body frame or wind frame in a following tutorial. For now, we use the superscript $\mathbb{W}$ to refer to the world frame reference. 

Given two positions, $\mathbf{p}_1^W, \mathbf{p}_2^W \in \mathbb{R}^3$, their displacement is:

$$\Delta\mathbf{p}^W = \mathbf{p}_2^W - \mathbf{p}_1^W$$

#### 2. Velocity
Velocity is the time rate of change of position. It represents both the speed and the instantaneous direction of motion. Formally, it is the first time derivative of the position vector:

$$\mathbf{v}^W = \frac{d\mathbf{p}^W}{dt} = \begin{bmatrix} \dot{p}_x \\ \dot{p}_y \\ \dot{p}_z \end{bmatrix} = \begin{bmatrix} v_x \\ v_y \\ v_z \end{bmatrix} \in \mathbb{R}^3$$

The overdot ($\dot{p}$) is standard notation representing a derivative with respect to time ($t$).

#### 3. Acceleration
Acceleration is the time rate of change of velocity, describing how quickly an object's speed or direction changes. It is the first time derivative of velocity and the second time derivative of position:

$$\mathbf{a}^W = \frac{d\mathbf{v}^W}{dt} = \frac{d^2\mathbf{p}^W}{dt^2} = \begin{bmatrix} \dot{v}_x \\ \dot{v}_y \\ \dot{v}_z \end{bmatrix} = \begin{bmatrix} \ddot{p}_x \\ \ddot{p}_y \\ \ddot{p}_z \end{bmatrix} = \begin{bmatrix} a_x \\ a_y \\ a_z \end{bmatrix} \in \mathbb{R}^3$$


## Newton's three laws

#### Newton's First Law
A body remains at rest or moves at a constant velocity unless acted on by a nonzero net external force. Mathematically, this implies that when the vector sum of forces is zero, the acceleration vector is also zero:

$$\sum_{i} \mathbf{F}_i = \mathbf{0} \implies \mathbf{a} = \mathbf{0}$$

#### Newton's Second Law
The net external force equals the rate of change of linear momentum:

$$\sum_{i} \mathbf{F}_i = \frac{d}{dt}(m\mathbf{v})$$

For a fixed mass ($m$), this simplifies to Newton's second law:

$$\sum_{i} \mathbf{F}_i = m\mathbf{a}$$

Here, $m \in \mathbb{R}_{>0}$ represents the mass, while $\mathbf{F}_i \in \mathbb{R}^3$ and $\mathbf{a} \in \mathbb{R}^3$ represent the force and acceleration vectors. Multiplying the vector $\mathbf{a}$ by the scalar $m$ yields:

$$m\mathbf{a} = m \begin{bmatrix} a_x \\ a_y \\ a_z \end{bmatrix} = \begin{bmatrix} ma_x \\ ma_y \\ ma_z \end{bmatrix}$$

This element-wise scaling highlights why Newton's second law is fundamentally a vector equation. Consequently, it serves as the foundational principle for deriving equations of motion in classical mechanics.

#### Newton's Third Law
When body $A$ exerts a force on body $B$, body $B$ exerts an equal and opposite force on body $A$. If $\mathbf{F}_{BA}$ represents the force exerted on body $B$ by body $A$, and $\mathbf{F}_{AB}$ represents the force exerted on body $A$ by body $B$, then:

$$\mathbf{F}_{BA} = -\mathbf{F}_{AB}$$

This balance ensures that internal forces within a closed system cancel out, conserving total linear momentum.

In summary, Newton's Three Laws are fundamentally based on Newton Mechanics key theoretical foundations: **Forces** and **vectors** describe the state of a system. Lagrangian and Hamiltonian mechanics adopt a different perspective. We will touch upon this in some research projects performed by our team. 

## 📐  Key External Forces

There are a couple of forces that are essential for studying autonomous systems. 

### 1. Gravity ($\mathbf{F}_g$)
Gravity is a constant field force acting downward along the negative vertical axis (typically the $z$-axis in aviation coordinate systems; we will get  back to this in another tutorial).
$$\mathbf{F}_g^W = \begin{bmatrix} 0 \\ 0 \\ -mg \end{bmatrix} \in \mathbb{R}^3$$
*   **$m \in \mathbb{R}_{>0}$:** The scalar mass of the vehicle ($\text{kg}$).
*   **$g \approx 9.81\text{ m/s}^2$:** The acceleration due to gravity.


### 3. Drag / Viscous Damping ($\mathbf{F}_d$)
Damping represents fluid resistance (e.g. aerodynamic drag) acting against the instantaneous velocity vector of the object. Under low-speed linear assumptions, it scales directly with velocity:
$$\mathbf{F}_d^W = -c \mathbf{v}^W = -c \begin{bmatrix} v_x \\ v_y \\ v_z \end{bmatrix} \in \mathbb{R}^3$$
*   **$c \in \mathbb{R}_{>0}$:** The damping coefficient ($\text{N}\cdot\text{s/m}$).
*   **$\mathbf{v}^W$:** The velocity vector. The negative sign ensures the force is always purely dissipative, directly opposing the direction of travel.

### 4. Restoring Spring Force ($\mathbf{F}_s$)
Spring forces represent physical connections (e.g., landing gear struts) to attract a moving object/vehicle to a desired reference position $\mathbf{p}_{\text{ref}}^W$:
$$
\mathbf{F}_s^W = -k (\mathbf{p}^W - \mathbf{p}_{\text{ref}}^W) = -k \Delta\mathbf{p}^W \in \mathbb{R}^3
$$ 
*   **$k \in \mathbb{R}_{>0}$:** The spring stiffness constant ($\text{N/m}$).
*   **$\mathbf{p}^W$:** The current position vector of the center of mass, and $\Delta\mathbf{p}^W$  describing the displacement of the point mass/vehicle. 

## Point mass assumptions
In many areas of research for autonomous aviation (such as high-level trajectory generation and global motion-planning), it is often sufficient to represent a complex vehicle like a quadrotor or fixed-wing aircraft as a simplified **point-mass** (or particle) to derive its translational equations of motion. 

For a point-mass model, the following core assumptions hold:

#### 1. Concentrated Mass at a Single Point
The point mass is a model representation of a body where the physical properties like volume, and geometry of the body are neglected. The entire finite mass of the body is assumed to be concentrated at a single, infinitely small spatial point corresponding to the body's Center of Mass (CoM). It is important that this just an approximation, as in reality no body is just a point mass. 

#### 2. Neglected Rotational Dynamics
Because a mathematical point has no spatial distribution, it possesses no rotational inertia. Consequently, the vehicle's orientation and angular velocity are completely omitted. The body is treated as having **three translational degrees of freedom (3-DoF)** rather than the full six degrees of freedom (6-DoF) of a true rigid body:

$$
\mathbf{x} = \begin{bmatrix} \mathbf{p}^W \\ \mathbf{v}^W \end{bmatrix} \in \mathbb{R}^6
$$

#### 3. Coincident Forces
All external forces acting on the vehicle—such as gravity or thrust—are assumed to act simultaneously and directly through this single point. Because the forces have no moment arm relative to the Center of Mass, they produce zero torque ($\boldsymbol{\tau} = \mathbf{0}$). 

%$$\boldsymbol{\tau} = \mathbf{r} \times \mathbf{F} = \mathbf{0} \times \mathbf{F} = \mathbf{0}$$

For sure, using such a simplified model has limitations. For example, it ignores the affects of the rigid bodies geometries on the forces and moments. Further, it ignores the affect of how the shape of the aircraft's wings create forces through fluid flows. 


# Deriving equations of motions for point mass sytems 

## Free Mass (Unforced, inertial motion)

When a point-mass moves in a vacuum with no external forces acting on it Newton's First Law holds. Inertia defines the motion. 

$$
\sum \mathbf{F} = 0 \implies m\,\mathbf{a} = 0 
$$

Since mass cannot be zero, the magnitude of the acceleration vector must be zero. 

We find the velocity vector by integrating the zero acceleration vector over time:

$$ \frac{d}{dt} \mathbf{v}(t) = \mathbf{0} $$
$$ \mathbf{v}(t) = \mathbf{v}_0 = \begin{bmatrix} v_{0x} \\ v_{0y} \\ v_{0z} \end{bmatrix} $$

The velocity vector remains completely constant over time. The mass experiences no change in its speed or its direction of motion.

We find the position vector by substituting the constant velocity vector into the position derivative equation and integrating:

$$ \frac{d}{dt} \mathbf{p}(t) = \mathbf{v}_0 $$
$$ \int_{0}^{t} \frac{d}{d\tau} \mathbf{p}(\tau) \, d\tau = \int_{0}^{t} \mathbf{v}_0 \, d\tau $$

Because $\mathbf{v}_0$ is a constant vector, it pulls out of the integral operator:
$$ \mathbf{p}(t) - \mathbf{p}_0 = \mathbf{v}_0 \int_{0}^{t} d\tau $$
$$ \mathbf{p}(t) = \mathbf{p}_0 + \mathbf{v}_0 t $$


#### Free Mass (Forced, one-dimensional motion)
Next, we can imagine a mass being forced with a force $\mathbf{u}$ along the horizontal axis x, defined as
$$
\mathbf{F_u(t)} = \mathbf{u(t)} = \begin{bmatrix} u_x(t) \\ 0 \\ 0 \end{bmatrix} \in \mathbb{R}^3
$$ 
We will omit for simplicity moving forward. 

Then, Newton's second law gives

$$
m\,\ddot{\mathbf{p}}(t)=\mathbf{u(t)}.
$$

Therefore,

$$
\boxed{\ddot p(t)=\frac{1}{m}u(t).}
$$

![Free point mass](figures/free_mass.png)

## Mass-damper (Forced, damped, one-dimensional motion)

A viscous damper produces a force proportional to relative velocity and opposite to motion. 

Using the definition of a damping force (see above), and focusing on horizontal motion only, this leads to  

$$
\mathbf{F_d(t)} = -c\dot{\mathbf{p}}(t) = -c \mathbf{v}(t) = -c \begin{bmatrix} v_x(t) \\ 0 \\ 0 \end{bmatrix} \in \mathbb{R}^3
$$. 


where \(c>0\) is the damping coefficient.

Thus, based on Newton's second law the horizontal force balance is

$$\
\mathbf{u}(t)-c\dot{\mathbf{p}}(t)=m\ddot{\mathbf{p}}(t).
$$

Rearranging and dropping the vector notation due to the one-dimensional motion, this leads to

$$
\boxed{m\ddot p(t)+c\dot p(t)=u(t)}
$$

or

$$
\boxed{\ddot p(t)=-\frac{c}{m}\dot p(t)+\frac{1}{m}u(t)}
$$

![Mass–damper system](figures/mass_damper.png)

>Interpretation: The damper resists velocity, not position. If the applied force is removed, the mass slows down and eventually stops, but it is not pulled back to its starting point.

## Mass-spring-damper

A linear spring exerts the restoring force

$$
F_s(t)=-kp(t),
$$

where \(k>0\) is the spring stiffness and \(q=0\) is the equilibrium position.

The forces are

$$
F_u=u(t),\qquad
F_d=-c\dot p(t),\qquad
F_s=-kq(t).
$$

Applying Newton's second law,

$$
u(t)-c\dot p(t)-kp(t)=m\ddot q(t).
$$

Thus,
$$
\boxed{m\ddot p(t)+c\dot p(t)+kp(t)=u(t).}
$$

or

$$
\boxed{
\ddot p(t)
=
-\frac{k}{m}p(t)
-\frac{c}{m}\dot p(t)
+\frac{1}{m}u(t).
}
$$

Summary of the key terms and physical mechanisms

| Term | Physical role |
|---|---|
| $m\, \ddot p(t)$ | inertial response |
| $c\,\dot p(t)$ | damping; opposes velocity |
| $k\,p(t)$ | restoring effect; opposes displacement |
| $u\,(t)$ | externally applied force |

## Connection to autonomous systems

A simplified vehicle model also begins with

$
m\ddot p=F_{\text{net}}.
$

The important modeling questions are:

- Which forces matter for the task?
- Which effects can be neglected?
- Is a point-mass model sufficient?
- Is motion one-dimensional, planar, or three-dimensional?
- Is orientation required?

These decisions determine the equation of motion and the fidelity of the resulting simulator.


## Quiz

### 1. Question 1

What information is neglected when a rigid object is modeled as a point mass?

### 2. Question 2

Why is the damping force written as

$
F_d=-c\dot p?
$

### 3. Question 3

A mass–damper system is released with nonzero velocity and no applied force. Does it return to $p(T)=0$ at time t=T, where $v(T)=0$?


### 4. Question 4

Starting from the force balance

$
u-c\dot p-kp=m\ddot p,
$

rewrite as unforced mass-spring-damper with $u=0$. 


## References

Åström, K. J., and Murray, R. M. (2008). *Feedback Systems: An Introduction for Scientists and Engineers*. Princeton University Press.
Roth, S., and Stahl, A. 2025. *Kinematics of the Point Mass,* in Mechanics: Experimental Physics - Descriptively Explained, S. Roth and A. Stahl (eds.), Berlin, Heidelberg: Springer, pp. 47–79. (https://doi.org/10.1007/978-3-662-68079-7_5).




