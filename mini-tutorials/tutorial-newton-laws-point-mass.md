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

# Tutorial 1 - Foundatinos: Newton's Law of Motion and Point-Mass Equation
This tutorial provides the foundations for physics-informed machine learning to solve problems related to autonomous systems such as motion-planing, control, system identification, as well as digital twinning. For some of the students, such a refresher on the basics of physics might be helpful to understand the forces that that constitute the dynamics of a drone, an aicraft, or other rigid bodies. We will not cover aerodyanmics or other fields of physics but start with the basics. The basic goal of this tutorial is to answer the following questions:

> How do physical systems move? 
> How can Newton's Law of Motion help us derive equations of motions, based in simplified assumptions of a "point-mass? 

By end of this tutorial you should be able to:

1. Refresh your memory on Newton's three laws of motion. 
2. Be able to distinguish between the study of kinematics from the study of kinetics, and dynamics (e.g. aerodynamics)
3. Clarify the assumption of a point-mass and (its limitations)
4. Derive the equations of motions for a free mass, and family of forced and unforced mass-damper systems
5. Simulate how different physical parameters effect the stability of a system. 

## Refresher: Scalar versus vectors
In order to understand equations of motions, it is important that you can distinguish between scalars and vectors, and how we represent them. The key mathematical distinction is:

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

```python
import numpy as np
import matplotlib.pyplot as plt

startpoint = np.array([0.0, 0.0, 0.0])
endpoint = np.array([0.0, 3.0, 5.0])
v = endpoint - startpoint
```

## Postion, velocity, and acceleration

Kinematics describes motion without asking what causes it.
Dynamics relates motion to forces and mass.

## Newton's three laws

### Newton's first law

A body remains at rest or moves at constant velocity unless acted on by a nonzero net external force.

### Newton's second law

The net external force equals the rate of change of linear momentum:

$$ \sum F = \frac{d}{dt}(m\vec{v}) $$

For constant mass $m$,

$$
\sum F = m\vec{a}
$$

This is the main law used to derive equations of motion.

### Newton's third law

When body $A$ exerts a force on body $B$, body $B$ exerts an equal and opposite force on body $A$.

## Point mass assumptions

## Mass-damper

## Mass-spring-damper

## Deriving the equations of motions

## References



