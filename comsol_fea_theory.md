[📄 View Full COMSOL Simulation Documentation (PDF)](COMSOL%20Simulation%20Documentation.pdf)

# Roll Cage COMSOL Simulation: Finite Element Method Behind the Interface

This document explains **how COMSOL Multiphysics internally models a solid mechanics simulation** like the roll cage deformation study described, using the finite element method (FEM).

---

## 1. Governing Equations

In linear elasticity (prior to plasticity or hyperelasticity), COMSOL solves:

$$
\nabla \cdot \boldsymbol{\sigma} + \mathbf{f} = \rho \ddot{\mathbf{u}}
$$

In the quasistatic case (stationary study with no inertial effects):

$$
\nabla \cdot \boldsymbol{\sigma} + \mathbf{f} = 0
$$

Where:
- $\boldsymbol{\sigma}$ is the Cauchy stress tensor
- $\mathbf{u}$ is the displacement vector field
- $\mathbf{f}$ is the body force (e.g., gravity)
- $\rho$ is the density

### Constitutive Law (Linear Elastic Material)
COMSOL uses Hooke’s law for isotropic materials:

$$
\boldsymbol{\sigma} = \lambda (\nabla \cdot \mathbf{u}) \mathbf{I} + 2\mu \boldsymbol{\varepsilon}
$$

where $\lambda$ and $\mu$ are the Lamé parameters:

$$
\lambda = \frac{E\nu}{(1+\nu)(1-2\nu)}, \quad \mu = \frac{E}{2(1+\nu)}
$$

and the strain-displacement relation is:

$$
\boldsymbol{\varepsilon} = \frac{1}{2}(\nabla \mathbf{u} + \nabla \mathbf{u}^T)
$$

---

## 2. Weak Formulation

To apply FEM, COMSOL rewrites the strong form as a weak (variational) form.

Multiply by a test function $\mathbf{v}$ and integrate over the domain $\Omega$:

$$
\int_{\Omega} \boldsymbol{\sigma} : \nabla \mathbf{v} \, d\Omega = \int_{\Omega} \mathbf{f} \cdot \mathbf{v} \, d\Omega + \int_{\Gamma_t} \bar{\mathbf{t}} \cdot \mathbf{v} \, d\Gamma
$$

Boundary conditions:
- Essential (Dirichlet): $\mathbf{u} = \mathbf{u}_0$ on $\Gamma_u$
- Natural (Neumann): $\boldsymbol{\sigma} \cdot \mathbf{n} = \bar{\mathbf{t}}$ on $\Gamma_t$

---

## 3. Discretization with Shape Functions

COMSOL interpolates displacement with nodal shape functions:

$$
\mathbf{u}(x) \approx \sum_i N_i(x) \mathbf{u}_i
$$

- $N_i$ are the shape functions (e.g., linear or quadratic polynomials)
- $\mathbf{u}_i$ are the nodal displacements

The weak form becomes a **system of algebraic equations**:

$$
\mathbf{K}\mathbf{U} = \mathbf{F}
$$

Where:
- $\mathbf{K}$ is the global stiffness matrix
- $\mathbf{F}$ is the global force vector
- $\mathbf{U}$ is the nodal displacement vector

---

## 4. Geometric Nonlinearity

In COMSOL, enabling *Include Geometric Nonlinearity* modifies strain definitions to include nonlinear terms (e.g., Green-Lagrange strain):

$$
\boldsymbol{E} = \frac{1}{2}(\nabla \mathbf{u} + \nabla \mathbf{u}^T + \nabla \mathbf{u}^T \nabla \mathbf{u})
$$

And updates stress from the second Piola-Kirchhoff stress $\mathbf{S}$ to account for nonlinear deformation gradients.

COMSOL uses **Total Lagrangian formulation** if chosen explicitly, otherwise the update method depends on the solver and selected formulation.

---

## 5. Mesh and Refinement

The geometry is discretized into tetrahedral elements (in 3D):

- **P-refinement** increases polynomial order of shape functions
- **H-refinement** increases mesh density (more elements)

Mesh convergence ensures accuracy of strain energy, stress distribution, and failure modes like buckling.

---

## 6. Solvers

COMSOL assembles and solves:

$$
\mathbf{K(U)} \mathbf{U} = \mathbf{F}
$$

If the system is **nonlinear**, Newton-Raphson iterations are used:
1. Compute residual $\mathbf{R(U)} = \mathbf{K(U)} \mathbf{U} - \mathbf{F}$
2. Linearize and solve: $\mathbf{K}_T \Delta \mathbf{U} = -\mathbf{R}$
3. Update: $\mathbf{U}^{(k+1)} = \mathbf{U}^{(k)} + \Delta \mathbf{U}$

This repeats until the residual is below a tolerance.

---

## 7. Crash Load Application

Crash force is modeled via:

$$
F_{\text{Crash}} = \frac{m \Delta v}{\Delta t}
$$

This is applied as a **boundary force** on selected surfaces. COMSOL distributes the force over the selected boundary elements.

---

## 8. Buckling and Collapse

Because large displacements and geometric stiffness changes are included:

- COMSOL captures **elastic instability** (buckling)
- No post-buckling plasticity is modeled unless a plastic material model is used

To detect collapse onset, one can monitor:
- Divergence of nonlinear solver
- Sharp change in stress/deformation with small load increase

---

## Summary

This simulation uses:
- Linear elastic material
- Geometric nonlinearity (nonlinear strain)
- Stationary solver with nonlinear iterations
- Boundary loads derived from crash impact
- Finite element mesh with tetrahedral elements
- Weak form of the elasticity equations

COMSOL automates many FEM tasks but is fundamentally solving the governing PDE in weak form using numerical discretization and Newton-Raphson methods.

