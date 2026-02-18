# IsoCompiler Solver Overview

This document describes the physics and numerical methods underlying the IsoCompiler electromagnetic solver. It is intended for reviewers who want to understand the approach without access to proprietary source code.

---

## 1. Governing Equations

IsoCompiler solves Maxwell's curl equations in the time domain:

```
curl(H) = eps * dE/dt + sigma * E
curl(E) = -mu * dH/dt
```

where:
- E = electric field vector (V/m)
- H = magnetic field vector (A/m)
- eps = permittivity (F/m), possibly frequency-dependent
- mu = permeability (H/m), assumed mu_0 for non-magnetic substrates
- sigma = electric conductivity (S/m), used for metallic structures

In the 2D Transverse Magnetic (TMz) polarization, the field components reduce to Ez, Hx, Hy. The update equations are:

```
Hx^{n+1/2} = Hx^{n-1/2} - (dt/mu) * dEz/dy
Hy^{n+1/2} = Hy^{n-1/2} + (dt/mu) * dEz/dx
Ez^{n+1}   = c_a * Ez^n + c_b * (dHy/dx - dHx/dy)
```

where c_a and c_b incorporate the permittivity and conductivity.

In the full 3D solver, all six field components (Ex, Ey, Ez, Hx, Hy, Hz) are updated following the standard Yee algorithm.

---

## 2. Spatial Discretization: Yee Grid

The solver uses the Yee staggered grid (Yee, 1966), where E-field and H-field components are evaluated at spatially offset half-cell positions. This staggering provides:

- **Second-order spatial accuracy** without requiring higher-order stencils.
- **Automatic enforcement** of the divergence-free condition (div(B) = 0, div(D) = 0 in source-free regions).
- **Natural coupling** between E and H through the leapfrog time-stepping scheme.

### Grid Resolution

The grid spacing dx must satisfy the Nyquist-like criterion:

```
dx <= lambda_min / N_min
```

where lambda_min is the shortest wavelength of interest (in the substrate material) and N_min >= 20 is the minimum cells-per-wavelength requirement. The effective wavelength in a substrate with relative permittivity eps_r is:

```
lambda_sub = c_0 / (f * sqrt(eps_r))
```

IsoCompiler automatically computes dx for each frequency band to ensure >= 20 cells per wavelength.

---

## 3. Time Stepping and Stability

The time step dt is determined by the Courant-Friedrichs-Lewy (CFL) condition:

```
2D:  dt < dx / (c_0 * sqrt(2))     -->  Courant number < 1/sqrt(2) ~ 0.707
3D:  dt < dx / (c_0 * sqrt(3))     -->  Courant number < 1/sqrt(3) ~ 0.577
```

IsoCompiler automatically computes a Courant-stable timestep for each simulation. The leapfrog scheme (E at integer timesteps, H at half-integer timesteps) provides second-order temporal accuracy.

---

## 4. Absorbing Boundaries: PML and CPML

### 2D Solver: Polynomial PML

The 2D solver uses a 20-cell Perfectly Matched Layer (PML) with polynomial-graded conductivity:

```
sigma_pml(d) = sigma_max * (d / d_pml)^m
```

where d is the distance into the PML, d_pml is the PML thickness, and m=3 (cubic grading). This provides less than -60 dB reflection from domain boundaries.

### 3D Solver: Convolutional PML (CPML)

The 3D solver uses the Convolutional PML formulation of Roden and Gedney (2000), which uses the Complex Frequency-Shifted (CFS) stretching function:

```
s_i = kappa_i + sigma_i / (alpha_i + j*omega*eps_0)
```

In the time domain, this becomes a recursive convolution with auxiliary variables (psi arrays):

```
psi^{n+1} = b * psi^n + a * (spatial_derivative)
```

CPML provides improved absorption at low frequencies and for evanescent waves compared to standard PML, and requires 12 auxiliary arrays for the 3D case (2 per field component for the 2 curl derivative directions).

---

## 5. Source Excitation

The solver uses a modulated Gaussian source:

```
J(t) = sin(2*pi*f_center * (t - t_0)) * exp(-0.5 * ((t - t_0) / sigma_t)^2)
```

where:
- f_center is the carrier frequency
- t_0 is the center time (delayed to avoid truncation)
- sigma_t controls the temporal width (bandwidth)

Source parameters are automatically computed to maximize spectral energy at the target analysis frequency. The source is injected as a soft current source (added to the E-field, not replacing it) along a line (2D) or plane (3D) at the designated source position.

---

## 6. S21 Extraction

The transmission coefficient S21 is extracted from time-domain monitor signals using the FFT ratio method:

```
S21(f) = FFT(E_monitor(t)) / FFT(E_source(t))
```

### Hanning Windowing

A Hanning window is applied to both time series before FFT to reduce spectral leakage. This provides approximately 30 dB improvement in sidelobe suppression compared to a rectangular window, preventing noise-floor artifacts from contaminating S21 measurements.

### Spectral Quality Validation

Each S21 extraction is accompanied by a spectral quality metric that checks whether the source has sufficient energy at the extraction frequency. Measurements are categorized as:

- **GENUINE:** High source energy at target frequency, low uncertainty.
- **MARGINAL:** Moderate spectral quality, results should be interpreted with caution.
- **ARTIFACT:** Insufficient source energy; the S21 value is a noise-floor measurement, not a real coupling coefficient.

---

## 7. Adjoint-Based Inverse Design

### The Optimization Problem

Given a design region D within the simulation domain, find the binary material distribution rho(x) in {0, 1} that minimizes S21 at the target frequency:

```
minimize  |S21(f_target; rho)|^2
subject to  rho(x) in {0, 1}  for all x in D
            DRC constraints (min feature size, min spacing)
```

### Adjoint Gradient Computation

The gradient dS21/d(rho) is computed using two FDTD simulations:

1. **Forward simulation:** Excite the source port, record fields everywhere in D.
2. **Adjoint simulation:** Place a source at the monitor port with amplitude proportional to the conjugate of the measured field. Run FDTD.
3. **Gradient:** At each point x in D, the overlap integral of forward and adjoint fields gives the sensitivity of S21 to a material perturbation at x.

The key insight is that this requires exactly 2 simulations regardless of the number of design variables. A 1,800-pixel design region (60 x 30) requires the same 2 simulations as a 180,000-pixel region.

### SIMP Penalization

The Solid Isotropic Material with Penalization (SIMP) method maps continuous density rho in [0, 1] to permittivity:

```
eps(rho) = eps_min + rho^p * (eps_max - eps_min)
```

With penalization exponent p=3, an intermediate density rho=0.5 yields only (0.5)^3 = 12.5% of the permittivity contrast. This makes intermediate densities energetically unfavorable, driving the optimizer toward binary solutions.

### Heaviside Projection

A smooth step function projects the density field toward binary values:

```
rho_projected = tanh(beta * eta) + tanh(beta * (rho - eta))
                ----------------------------------------
                tanh(beta * eta) + tanh(beta * (1 - eta))
```

where beta is a steepness parameter ramped from 1 to 256 over the course of optimization, and eta = 0.5 is the projection threshold. At high beta, this approaches a true step function.

### Density Filter

A spatial convolution filter with radius r_min enforces minimum feature size:

```
rho_filtered(x) = sum_y w(x-y) * rho(y) / sum_y w(x-y)
```

where w is a cone filter function with support radius r_min. This prevents isolated single-pixel features that would violate DRC constraints.

---

## 8. Material Models

### Drude Model (Metals)

```
eps(omega) = 1 - omega_p^2 / (omega^2 + j*omega*gamma)
```

Used for copper, aluminum, gold, silver, tin, nickel.

### Debye Model (Dielectrics)

```
eps(omega) = eps_inf + (eps_s - eps_inf) / (1 + j*omega*tau)
```

Used for FR4, Rogers 4350B, Megtron 6/7, polyimide, and other PCB substrates.

### Lorentz Model (Resonant Materials)

```
eps(omega) = eps_inf + (eps_s - eps_inf) * omega_0^2 / (omega_0^2 - omega^2 - j*omega*gamma)
```

Used for materials with resonant dielectric response.

---

## 9. Synthesis Families

IsoCompiler supports five isolation structure synthesis families:

### Via Fence
Periodic vertical via walls connecting top and bottom ground planes. Effective when via spacing < lambda/10. Isolation governed by via pitch relative to wavelength.

### Mushroom EBG (Sievenpiper)
Metal patches connected to ground by central vias. Bandgap center frequency:

```
f_bandgap = 1 / (2*pi*sqrt(L*C))
L = mu_0 * h        (via inductance)
C = eps_0*eps_r*w^2/h  (patch capacitance)
```

### Fractal EBG (Hilbert Curve)
Space-filling Hilbert curve creating multiple resonances at different frequencies. The multi-scale geometry provides broadband isolation.

### Metasurface / DGS
Slots in ground plane creating resonant stopband:

```
f_resonance = c_0 / (2 * L_slot * sqrt(eps_eff))
```

### Inverse Design
Freeform binary patterns optimized by adjoint topology optimization (see Section 7).

---

## 10. References

- Yee, K.S., "Numerical solution of initial boundary value problems involving Maxwell's equations in isotropic media," IEEE Trans. AP, 14(3), 302-307, 1966.
- Taflove, A. and Hagness, S.C., "Computational Electrodynamics: The FDTD Method," 3rd ed., Artech House, 2005.
- Roden, J.A. and Gedney, S.D., "CPML: An Efficient FDTD Implementation of the CFS-PML," IEEE MWCL, 10(11), 467-469, 2000.
- Berenger, J.P., "A Perfectly Matched Layer for the Absorption of Electromagnetic Waves," J. Comp. Phys., 114, 185-200, 1994.
- Molesky, S. et al., "Inverse design in nanophotonics," Nature Photonics, 12, 659-670, 2018.
- Bendsoe, M.P. and Sigmund, O., "Topology Optimization: Theory, Methods, and Applications," Springer, 2003.
- Guest, J.K. et al., "Achieving minimum length scale in topology optimization," Int. J. Numer. Methods Eng., 61, 238-254, 2004.
- Sievenpiper, D. et al., "High-impedance electromagnetic surfaces with a forbidden frequency band," IEEE Trans. MTT, 47(11), 2059-2074, 1999.
- Pozar, D.M., "Microwave Engineering," 4th ed., Wiley, 2012.

---

*This document describes the mathematical and physical foundations of the IsoCompiler solver without disclosing proprietary implementation details or source code.*
