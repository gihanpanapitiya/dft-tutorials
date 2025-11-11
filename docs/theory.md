# Theoretical Foundations of DFT

This guide provides the theoretical background for understanding density functional theory calculations, including Kohn-Sham orbitals, pseudopotentials, and related concepts.

## 📚 Table of Contents

1. [Many-Body Problem](#many-body-problem)
2. [Density Functional Theory](#density-functional-theory)
3. [Kohn-Sham Equations](#kohn-sham-equations)
4. [Exchange-Correlation Functionals](#exchange-correlation-functionals)
5. [Pseudopotential Theory](#pseudopotential-theory)
6. [Basis Sets](#basis-sets)
7. [Practical Considerations](#practical-considerations)

---

## 🎯 Many-Body Problem

### The Electronic Schrödinger Equation

For a system of N electrons and M nuclei, the full many-body Schrödinger equation is:

$$\hat{H}\Psi(\mathbf{r}_1, \mathbf{r}_2, ..., \mathbf{r}_N) = E\Psi(\mathbf{r}_1, \mathbf{r}_2, ..., \mathbf{r}_N)$$

The Hamiltonian contains:

$$\hat{H} = \hat{T}_e + \hat{V}_{ext} + \hat{V}_{ee}$$

Where:
- $\hat{T}_e = -\frac{\hbar^2}{2m}\sum_{i=1}^N \nabla_i^2$ - Kinetic energy of electrons
- $\hat{V}_{ext} = \sum_{i=1}^N v_{ext}(\mathbf{r}_i)$ - External potential (nuclei)
- $\hat{V}_{ee} = \frac{1}{2}\sum_{i\neq j}^N \frac{e^2}{|\mathbf{r}_i - \mathbf{r}_j|}$ - Electron-electron repulsion

### The Challenge

- Wavefunction depends on **3N coordinates** (for N electrons)
- For N=100: 300-dimensional function!
- Impossible to solve exactly for N > 2
- Need approximations...

---

## 🔬 Density Functional Theory

### Hohenberg-Kohn Theorems (1964)

#### Theorem 1: Existence
*The external potential $v_{ext}(\mathbf{r})$ is (to within a constant) a unique functional of the electron density $\rho(\mathbf{r})$.*

**Consequence**: Since $v_{ext}$ determines the Hamiltonian, and the Hamiltonian determines all properties, the ground state electron density uniquely determines all ground state properties.

$$E[\rho] = T[\rho] + V_{ee}[\rho] + \int v_{ext}(\mathbf{r})\rho(\mathbf{r})d\mathbf{r}$$

#### Theorem 2: Variational Principle
*The ground state energy can be obtained variationally: the density that minimizes the total energy is the exact ground state density.*

$$E_0 = \min_{\rho} E[\rho]$$

subject to: $\int \rho(\mathbf{r})d\mathbf{r} = N$

### Key Insight

Instead of a **3N-dimensional wavefunction**, we work with the **3-dimensional density**:

$$\rho(\mathbf{r}) = N\int d\mathbf{r}_2...d\mathbf{r}_N |\Psi(\mathbf{r}, \mathbf{r}_2, ..., \mathbf{r}_N)|^2$$

This is an enormous simplification!

### The Problem

We don't know the exact form of $T[\rho]$ and $V_{ee}[\rho]$...

---

## ⚛️ Kohn-Sham Equations

### Kohn-Sham Ansatz (1965)

**Brilliant idea**: Map the interacting system onto a fictitious non-interacting system with the **same density**.

### Non-Interacting System

For non-interacting electrons in an effective potential $v_s(\mathbf{r})$:

$$\left[-\frac{\hbar^2}{2m}\nabla^2 + v_s(\mathbf{r})\right]\psi_i(\mathbf{r}) = \varepsilon_i\psi_i(\mathbf{r})$$

These are the **Kohn-Sham equations** for the **Kohn-Sham orbitals** $\psi_i(\mathbf{r})$.

### Density from Orbitals

The density is constructed from occupied orbitals:

$$\rho(\mathbf{r}) = \sum_{i=1}^{N} f_i|\psi_i(\mathbf{r})|^2$$

where $f_i$ are occupation numbers (0, 1, or fractional for metals).

### Effective Potential

$$v_s(\mathbf{r}) = v_{ext}(\mathbf{r}) + v_H(\mathbf{r}) + v_{xc}(\mathbf{r})$$

Components:

1. **External potential** (nuclei):
   $$v_{ext}(\mathbf{r}) = -\sum_I \frac{Z_I}{|\mathbf{r} - \mathbf{R}_I|}$$

2. **Hartree potential** (classical electron repulsion):
   $$v_H(\mathbf{r}) = \int \frac{\rho(\mathbf{r}')}{|\mathbf{r} - \mathbf{r}'|}d\mathbf{r}'$$

3. **Exchange-correlation potential**:
   $$v_{xc}(\mathbf{r}) = \frac{\delta E_{xc}[\rho]}{\delta \rho(\mathbf{r})}$$

### Energy Expression

$$E[\rho] = T_s[\rho] + \int v_{ext}(\mathbf{r})\rho(\mathbf{r})d\mathbf{r} + E_H[\rho] + E_{xc}[\rho]$$

Where:
- $T_s[\rho]$ - Kinetic energy of non-interacting system (exact!)
- $E_H[\rho] = \frac{1}{2}\int\int \frac{\rho(\mathbf{r})\rho(\mathbf{r}')}{|\mathbf{r}-\mathbf{r}'|}d\mathbf{r}d\mathbf{r}'$ - Hartree energy
- $E_{xc}[\rho]$ - Exchange-correlation energy (unknown!)

### Self-Consistent Solution

1. Start with initial guess for $\rho(\mathbf{r})$
2. Compute $v_s(\mathbf{r})$ from $\rho(\mathbf{r})$
3. Solve Kohn-Sham equations → get $\psi_i(\mathbf{r})$
4. Compute new density: $\rho'(\mathbf{r}) = \sum_i f_i|\psi_i(\mathbf{r})|^2$
5. Mix: $\rho_{new} = (1-\alpha)\rho + \alpha\rho'$
6. Check convergence: $|\rho_{new} - \rho| < \epsilon$?
7. If not converged, return to step 2

This is the **Self-Consistent Field (SCF)** procedure!

### What are Kohn-Sham Orbitals?

**Important**: Kohn-Sham orbitals $\psi_i(\mathbf{r})$ are:
- **Mathematical constructs** that give the correct density
- **Not** the true many-body wavefunctions
- **But**: Often have physical meaning (HOMO, LUMO, band structure)

**Eigenvalues** $\varepsilon_i$:
- **Not** exact excitation energies
- **But**: Good approximations for valence electrons
- **Exception**: HOMO energy ≈ ionization energy (exactly, if exact $E_{xc}$)

---

## 🎨 Exchange-Correlation Functionals

The only unknown in Kohn-Sham DFT is $E_{xc}[\rho]$. This contains:
- **Exchange**: Pauli exclusion (same-spin electrons avoid each other)
- **Correlation**: Coulombic correlation (opposite-spin too)
- **Correction** to kinetic energy: $T - T_s$

### Local Density Approximation (LDA)

**Idea**: Use exchange-correlation energy density of homogeneous electron gas:

$$E_{xc}^{LDA}[\rho] = \int \rho(\mathbf{r})\varepsilon_{xc}^{hom}(\rho(\mathbf{r}))d\mathbf{r}$$

where $\varepsilon_{xc}^{hom}(\rho)$ is known from quantum Monte Carlo.

**Exchange part** (exact for uniform gas):
$$\varepsilon_x(\rho) = -\frac{3}{4}\left(\frac{3\rho}{\pi}\right)^{1/3}$$

**Correlation part**: Parametrized (e.g., Perdew-Zunger 1981)

**Good for**: Metals, high-density regions
**Bad for**: Molecules, inhomogeneous systems

### Generalized Gradient Approximation (GGA)

**Idea**: Include gradient of density:

$$E_{xc}^{GGA}[\rho] = \int f(\rho(\mathbf{r}), \nabla\rho(\mathbf{r}))d\mathbf{r}$$

**Popular functionals**:

#### PBE (Perdew-Burke-Ernzerhof, 1996)
- Most widely used
- Good for solids and molecules
- No empirical parameters
- Default choice for most calculations

$$E_{xc}^{PBE} = \int \rho \varepsilon_x^{unif}F_x^{PBE}(s) + \rho \varepsilon_c^{PBE}(\rho, \nabla\rho) d\mathbf{r}$$

where $s = |\nabla\rho|/(2k_F\rho)$ is reduced density gradient.

#### PBEsol (for solids, 2008)
- Modified PBE for better lattice constants
- Slightly overbinds (like LDA)

#### BLYP (Becke-Lee-Yang-Parr)
- Popular in quantum chemistry
- Good for molecules

**Better than LDA for**: Bond energies, structures, surfaces

### Meta-GGA

**Idea**: Include kinetic energy density:

$$\tau(\mathbf{r}) = \frac{1}{2}\sum_i^{occ}|\nabla\psi_i(\mathbf{r})|^2$$

$$E_{xc}^{meta-GGA}[\rho] = \int f(\rho, \nabla\rho, \nabla^2\rho, \tau)d\mathbf{r}$$

**Examples**:
- **SCAN** (Strongly Constrained and Appropriately Normed)
- **TPSS** (Tao-Perdew-Staroverov-Scuseria)

**Pros**: Better accuracy than GGA
**Cons**: More expensive, sometimes convergence issues

### Hybrid Functionals

**Idea**: Mix exact exchange with DFT:

$$E_{xc}^{hybrid} = aE_x^{exact} + (1-a)E_x^{DFT} + E_c^{DFT}$$

**Examples**:
- **PBE0**: 25% exact exchange
  $$E_{xc}^{PBE0} = 0.25E_x^{HF} + 0.75E_x^{PBE} + E_c^{PBE}$$
  
- **HSE06**: Screened hybrid (range-separated)
  - Short-range: 25% exact exchange
  - Long-range: 100% PBE
  - Better for extended systems

- **B3LYP**: Popular in chemistry
  $$E_{xc}^{B3LYP} = 0.20E_x^{HF} + 0.72E_x^{B88} + 0.08E_x^{LDA} + 0.81E_c^{LYP} + 0.19E_c^{VWN}$$

**Pros**: 
- Better band gaps (closer to experiment)
- Better reaction barriers
- Improved description of localized states

**Cons**:
- Much more expensive (10-100× slower)
- Need exact exchange evaluation

### Jacob's Ladder of Functionals

1. **LDA** - Local density
2. **GGA** - Gradient of density
3. **Meta-GGA** - Kinetic energy density
4. **Hybrid** - Exact exchange
5. **Double hybrid** - Exact correlation (very expensive)

Higher rungs = more accurate, but more expensive!

### Choosing a Functional

**Rule of thumb**:
- **Start with PBE** (GGA) - best balance of accuracy/cost
- **Need band gaps?** → HSE06 or PBE0
- **High accuracy?** → SCAN (meta-GGA)
- **Quick calculation?** → LDA
- **Benchmarked system?** → Use what literature uses

**Important**: Match functional in pseudopotential to calculation!

---

## 🛡️ Pseudopotential Theory

### The Core Electron Problem

All-electron calculations are expensive because:
1. **Core electrons oscillate rapidly** near nucleus
2. **Need huge basis sets** to describe core wavefunctions
3. **Core electrons don't participate in bonding**

Example for Silicon (Si):
- Full configuration: 1s² 2s² 2p⁶ 3s² 3p² (14 electrons)
- Only 3s² 3p² participate in chemistry
- Why calculate all 14?

### Pseudopotential Approximation

**Idea**: Replace strong nucleus + core electrons with a weaker **effective potential** acting only on valence electrons.

### Requirements (Norm-Conserving)

A good pseudopotential should satisfy:

1. **Valence eigenvalues match**: 
   $$\varepsilon_l^{PS} = \varepsilon_l^{AE}$$

2. **Valence wavefunctions match outside $r_c$**:
   $$\psi_l^{PS}(r) = \psi_l^{AE}(r) \quad \text{for } r > r_c$$
   
   where $r_c$ is the **core radius**.

3. **Norm conservation** (charge inside $r_c$ matches):
   $$\int_0^{r_c}|\psi_l^{PS}(r)|^2 r^2 dr = \int_0^{r_c}|\psi_l^{AE}(r)|^2 r^2 dr$$

4. **First derivative continuous** at $r_c$:
   $$\frac{d\psi_l^{PS}}{dr}\Big|_{r_c} = \frac{d\psi_l^{AE}}{dr}\Big|_{r_c}$$

### Generating Pseudopotentials

#### 1. All-Electron Calculation
Solve all-electron Schrödinger equation for isolated atom:
$$\left[-\frac{\nabla^2}{2} + V_{AE}(r)\right]\psi_l^{AE}(r) = \varepsilon_l^{AE}\psi_l^{AE}(r)$$

#### 2. Choose Core Radius $r_c$
- Smaller $r_c$ → harder PP → higher cutoff
- Larger $r_c$ → softer PP → lower cutoff, less transferable

#### 3. Construct Pseudo-Wavefunction
Inside $r_c$, create smooth function (e.g., polynomial):
$$\psi_l^{PS}(r) = r^{l+1}\sum_{i=0}^n c_i r^{2i} \quad r < r_c$$

Must satisfy continuity conditions at $r_c$.

#### 4. Invert to Get Pseudopotential
$$V_l^{PS}(r) = \varepsilon_l - \frac{1}{2\psi_l^{PS}}\frac{d^2\psi_l^{PS}}{dr^2}$$

#### 5. Unscreen the Potential
Remove valence electron density to get bare pseudopotential:
$$V_l^{PP}(r) = V_l^{PS}(r) - V_H^{val}(r) - V_{xc}^{val}(r)$$

### Types of Pseudopotentials

#### Norm-Conserving (NC)

**Properties**:
- Satisfy all conditions above
- Most accurate and transferable
- **Hardness**: Need high cutoffs

**Common types**:
- **Troullier-Martins** (1991): Analytical smooth form
- **HGH/GTH** (Goedecker-Teter-Hutter): Separable, Gaussian form
- **Optimized NC** (Hamann 2013): Modern, efficient

**Cutoff**: Typically 50-100 Ry for plane waves

#### Ultrasoft Pseudopotentials (USPP)

**Idea** (Vanderbilt 1990): Relax norm-conservation → softer potentials

**Method**:
- Introduce **augmentation charges** $Q_{ij}$
- Generalized orthonormality:
  $$\langle\psi_i|\psi_j\rangle + \sum_{nm}Q_{ij,nm}\langle\psi_i|\beta_n\rangle\langle\beta_m|\psi_j\rangle = \delta_{ij}$$

**Advantages**:
- Lower cutoffs (50-75% of NC)
- Very efficient for 1st row and transition metals

**Disadvantages**:
- More complex formalism
- Need to store/compute augmentation

**Cutoff**: Typically 30-60 Ry

#### Projector Augmented Wave (PAW)

**Idea** (Blöchl 1994): Reconstruct all-electron wavefunctions

**Transformation**:
$$|\Psi_i\rangle = |\tilde{\Psi}_i\rangle + \sum_a\sum_{lm}(|\phi_a^{lm}\rangle - |\tilde{\phi}_a^{lm}\rangle)\langle\tilde{p}_a^{lm}|\tilde{\Psi}_i\rangle$$

where:
- $|\tilde{\Psi}_i\rangle$ - pseudo (smooth) wavefunction
- $|\phi_a^{lm}\rangle$ - all-electron partial waves
- $|\tilde{\phi}_a^{lm}\rangle$ - pseudo partial waves
- $|\tilde{p}_a^{lm}\rangle$ - projector functions

**Features**:
- **Frozen-core approximation** but all-electron accuracy
- Can access core properties (hyperfine, NMR)
- Excellent for magnetic properties

**Advantages**:
- Most accurate pseudopotential method
- Access to core observables
- Good efficiency

**Disadvantages**:
- More expensive than USPP
- More complex implementation

**Cutoff**: Typically 30-50 Ry

### Pseudopotential Form

General form (Kleinman-Bylander):

$$V^{PS} = V_{local}(r) + \sum_{l=0}^{l_{max}}\sum_{m=-l}^l |Y_{lm}\rangle\Delta V_l(r)\langle Y_{lm}|$$

where:
- $V_{local}(r)$ - local part (usually highest $l$)
- $\Delta V_l(r) = V_l(r) - V_{local}(r)$ - non-local corrections
- $Y_{lm}$ - spherical harmonics

### Transferability

**Definition**: Pseudopotential works well in different chemical environments.

**Test environments**:
1. Isolated atom
2. Dimer
3. Bulk solid
4. Surface
5. Different oxidation states

**Poor transferability** if:
- Core radius too large
- Wrong reference configuration
- Extreme environments (high pressure, unusual bonding)

### Semicore States

**Problem**: Some "core" electrons participate in bonding!

**Examples**:
- Transition metals: (n-1)d electrons
- Alkali/alkaline earth under pressure: (n-1)p electrons

**Solution**: Include semicore in valence

Si example:
- Standard: 3s² 3p² (4 valence)
- Semicore: 2s² 2p⁶ 3s² 3p² (12 valence)

Use semicore for:
- Transition metals (include 3d)
- High-pressure calculations
- When in doubt, test both!

### Relativistic Effects

For heavy elements (Z > 30), relativistic effects important:

**Scalar relativistic**:
- Include mass-velocity and Darwin terms
- No spin-orbit coupling
- Good for most applications

**Fully relativistic**:
- Include spin-orbit coupling
- Needed for heavy elements
- Doubles computational cost (spinor wavefunctions)

---

## 📐 Basis Sets

### Plane Waves

**Expansion**:
$$\psi_i(\mathbf{r}) = \sum_{\mathbf{G}}c_{i,\mathbf{G}}e^{i(\mathbf{k}+\mathbf{G})\cdot\mathbf{r}}$$

where $\mathbf{G}$ are reciprocal lattice vectors.

**Cutoff**: Include all $\mathbf{G}$ with:
$$\frac{|\mathbf{k}+\mathbf{G}|^2}{2} < E_{cut}$$

**Advantages**:
- Systematic convergence (increase $E_{cut}$)
- No basis set superposition error
- Easy force/stress calculations
- FFT acceleration

**Disadvantages**:
- Need pseudopotentials (oscillations near nuclei)
- Inefficient for localized states
- Large vacuum regions wasteful

**Used by**: Quantum ESPRESSO, ABINIT, VASP

### Atomic Orbitals (LCAO)

**Expansion**:
$$\psi_i(\mathbf{r}) = \sum_{a,\mu}c_{i,a\mu}\phi_{a\mu}(\mathbf{r}-\mathbf{R}_a)$$

where $\phi_{a\mu}$ are atomic orbitals centered on atom $a$.

**Basis quality**:
- **SZ** (Single-ζ): Minimal basis
- **DZ** (Double-ζ): Two functions per orbital
- **DZP** (Double-ζ + Polarization): Add higher $l$
- **TZP**: Triple-ζ + polarization

**Advantages**:
- Sparse matrices (localized orbitals)
- O(N) algorithms possible
- Natural for molecules
- Smaller basis for same accuracy (than plane waves)

**Disadvantages**:
- Basis set superposition error
- Non-systematic convergence
- Must choose basis carefully

**Confinement**:
Orbitals forced to zero at radius $r_c$:
$$\phi(r) = \begin{cases} \tilde{\phi}(r) & r < r_c \\ 0 & r \geq r_c \end{cases}$$

Controlled by **PAO.EnergyShift** in SIESTA.

**Used by**: SIESTA, OpenMX

### Gaussian and Plane Waves (GPW)

**Idea** (CP2K): Combine both approaches!

- **Orbitals**: Expanded in Gaussians (localized)
- **Density**: Represented on plane-wave grid (extended)

**Method**:
$$\rho(\mathbf{r}) = \sum_{ij}\langle\psi_i|\psi_j\rangle\psi_i^*(\mathbf{r})\psi_j(\mathbf{r})$$

Gaussians for orbitals, FFT for Hartree/XC potentials.

**Two cutoffs**:
- **CUTOFF**: Plane-wave cutoff for density grid
- **REL_CUTOFF**: Controls Gaussian→grid mapping

**Advantages**:
- Best of both worlds
- Efficient for molecules and solids
- Sparse matrix operations

**Used by**: CP2K

---

## 🔧 Practical Considerations

### Convergence Parameters

Must converge:
1. **Basis size** (plane-wave cutoff or LCAO basis)
2. **k-point sampling** (Brillouin zone integration)
3. **SCF tolerance** (self-consistency)
4. **Cell size** (for finite systems)

### Computational Cost

Typical scaling:
- **Diagonalization**: O(N³) for N electrons
- **FFT**: O(M log M) for M grid points
- **Total**: O(N³) + O(M log M)

**Advanced methods**:
- **Orbital transformation (OT)**: O(N) for gap systems
- **Linear scaling**: True O(N) for large systems

### Band Gap Problem

**Issue**: DFT (LDA/GGA) severely underestimates band gaps!

**Why?**: 
- Approximate $E_{xc}$ has wrong derivative discontinuity
- Missing static correlation

**Example** (Si):
- Experiment: 1.17 eV
- LDA: 0.5 eV
- PBE: 0.6 eV
- HSE06: 1.1 eV
- GW: 1.2 eV

**Solutions**:
- Hybrid functionals (HSE, PBE0)
- GW approximation (many-body)
- Empirical corrections (scissor operator)

### Spin Polarization

For magnetic systems:
$$\rho(\mathbf{r}) = \rho_\uparrow(\mathbf{r}) + \rho_\downarrow(\mathbf{r})$$

Solve separate equations for each spin:
$$\hat{H}_\sigma\psi_{i\sigma} = \varepsilon_{i\sigma}\psi_{i\sigma}$$

**When needed**:
- Open-shell atoms/molecules
- Magnetic materials
- Radicals

---

## 📚 Summary

### Key Concepts

1. **DFT**: Use density instead of wavefunction
2. **Kohn-Sham**: Map to non-interacting system
3. **Exchange-Correlation**: Only approximation needed
4. **Pseudopotentials**: Remove core electrons efficiently
5. **Basis Sets**: Expand Kohn-Sham orbitals

### Hierarchy of Approximations

```
Schrödinger Equation (Exact)
         ↓
Born-Oppenheimer (nuclei frozen)
         ↓
Density Functional Theory (use density)
         ↓
Kohn-Sham Equations (non-interacting reference)
         ↓
Exchange-Correlation Functional (LDA/GGA/Hybrid)
         ↓
Pseudopotential (remove core electrons)
         ↓
Basis Set (finite representation)
         ↓
Numerical Implementation (FFT, convergence)
```

Each step introduces approximations - must test carefully!

---

## 📖 Further Reading

### Books
- **Parr & Yang** - "Density-Functional Theory of Atoms and Molecules"
- **Martin** - "Electronic Structure: Basic Theory and Practical Methods"
- **Sholl & Steckel** - "Density Functional Theory: A Practical Introduction"
- **Koch & Holthausen** - "A Chemist's Guide to Density Functional Theory"

### Reviews
- **Kohn et al.** - "Electronic Structure of Matter" (Nobel lecture)
- **Payne et al.** - "Iterative minimization techniques" Rev. Mod. Phys. 64, 1045 (1992)
- **Hamann et al.** - "Norm-conserving pseudopotentials" Phys. Rev. Lett. 43, 1494 (1979)

### Original Papers
- **Hohenberg & Kohn** (1964) - HK theorems
- **Kohn & Sham** (1965) - KS equations
- **Perdew et al.** (1996) - PBE functional
- **Blöchl** (1994) - PAW method

---

**Remember**: DFT is an approximation! Always:
- Test convergence
- Compare with experiment when possible
- Understand limitations of your functional
- Document your choices!
