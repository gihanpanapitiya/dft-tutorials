Sure. **ABINIT** is an *ab initio* electronic-structure code, and its theory is rooted mainly in **Density Functional Theory (DFT)** and related many-body extensions. I’ll explain the theory step-by-step, from fundamentals to what ABINIT actually solves in practice.

---

## 1. Many-electron problem (starting point)

For a system of nuclei and electrons, the exact non-relativistic Hamiltonian is

$$
\left[
\hat H =
-\sum_i \frac{\hbar^2}{2m_e}\nabla_i^2
-\sum_I \frac{\hbar^2}{2M_I}\nabla_I^2
-\sum_{i,I} \frac{Z_I e^2}{\lvert \mathbf r_i - \mathbf R_I \rvert}
+\sum_{i<j} \frac{e^2}{\lvert \mathbf r_i - \mathbf r_j \rvert}
+\sum_{I<J} \frac{Z_I Z_J e^2}{\lvert \mathbf R_I - \mathbf R_J \rvert}
\right]
$$
This problem is impossible to solve exactly for real materials.

---

## 2. Born–Oppenheimer approximation

ABINIT assumes **Born–Oppenheimer separation**:

* Nuclei are treated as fixed classical particles
* Only the electronic Schrödinger equation is solved

The nuclear–nuclear interaction becomes a constant energy term.

---

## 3. Density Functional Theory (DFT)

### 3.1 Hohenberg–Kohn theorems

DFT is based on two key theorems:

1. **Ground-state electron density** (n(\mathbf r)) uniquely determines:

   * External potential
   * Total energy
   * All ground-state properties

2. There exists a **universal energy functional**:
   $
   E[n] = T[n] + E_{\text{ext}}[n] + E_{\text{Hartree}}[n] + E_{\text{xc}}[n]
   $

The challenge is the **exchange–correlation functional** ($E_{\text{xc}}[n]$).

---

## 4. Kohn–Sham formalism (what ABINIT actually solves)

Instead of interacting electrons, Kohn–Sham DFT introduces **non-interacting electrons** with the same density.

### 4.1 Kohn–Sham equations

ABINIT solves:
$$
\left[
-\frac{\hbar^2}{2m_e}\nabla^2
+ V_{\text{ext}}(\mathbf r)
+ V_H(\mathbf r)
+ V_{xc}(\mathbf r)
\right]
\psi_{n\mathbf k}(\mathbf r)
=
\epsilon_{n\mathbf k}\,
\psi_{n\mathbf k}(\mathbf r)
$$

where:

- $(V_H)$: Hartree (classical Coulomb)
- $(V_{xc} = \delta E_{xc}/\delta n)$

The electron density is:
$
n(\mathbf r) = \sum_{n\mathbf k} f_{n\mathbf k} |\psi_{n\mathbf k}(\mathbf r)|^2
$

This is solved **self-consistently (SCF)**.

---

## 5. Exchange–correlation approximations in ABINIT

ABINIT supports many (E_{xc}) approximations:

### Local & semi-local

* **LDA**
* **GGA** (PBE, PBEsol, etc.)
* **meta-GGA**

### Non-local & advanced

* **Hybrid functionals** (HSE, PBE0)
* **DFT+U** (strongly correlated systems)
* **van der Waals** (DFT-D, vdW-DF)

---

## 6. Basis set: Plane waves

ABINIT uses a **plane-wave basis**:

$
\psi_{n\mathbf k}(\mathbf r) =
\sum_{\mathbf G} c_{n\mathbf k}(\mathbf G)
e^{i(\mathbf k + \mathbf G)\cdot \mathbf r}
$

Advantages:

* Systematic convergence via cutoff energy
* Naturally suited for periodic solids

This requires **pseudopotentials**.

---

## 7. Pseudopotentials

ABINIT replaces core electrons with **pseudopotentials**:

* Norm-conserving
* Ultrasoft (limited support)
* PAW (Projector Augmented Wave)

This reduces computational cost while retaining accuracy for valence electrons.

---

## 8. Periodic boundary conditions & k-points

For crystals:

* Bloch’s theorem is used
* Brillouin zone integrals are approximated with **k-point sampling**
  (Monkhorst–Pack grids)

---

## 9. Total energy and forces

Once self-consistency is reached, ABINIT computes:

### Total energy

$$
E_{\text{tot}}
=
\sum_{n\mathbf k} f_{n\mathbf k}\,\epsilon_{n\mathbf k}
- E_H
+ E_{xc}
- \int V_{xc}(\mathbf r)\, n(\mathbf r)\, d\mathbf r
+ E_{\text{ion--ion}}
$$

### Forces (Hellmann–Feynman)

Used for:

* Geometry optimization
* Molecular dynamics

---

## 10. Beyond ground-state DFT in ABINIT

ABINIT includes advanced many-body and response theories:

### Density-functional perturbation theory (DFPT)

* Phonons
* Dielectric constants
* Elastic constants
* Electron–phonon coupling

### GW approximation

* Quasiparticle band structures

### Bethe–Salpeter equation (BSE)

* Optical excitations
* Excitons

### Time-dependent DFT (TDDFT)

* Excited states
* Optical spectra

---

## 11. Summary (in one sentence)

**ABINIT implements Kohn–Sham DFT using plane-wave basis sets and pseudopotentials, solving the electronic ground state self-consistently under periodic boundary conditions, with extensions to phonons, excited states, and many-body perturbation theory.**

---

If you want, I can:

* Explain **DFPT or GW in ABINIT**
* Connect theory to **input variables**
* Compare ABINIT with **VASP / Quantum ESPRESSO**
* Go through **one equation → one input keyword mapping**

Just tell me your level (student / PhD / postdoc) and goal 🙂




---


This is an **excellent and very fundamental question**—once this clicks, *plane-wave DFT* really makes sense.

I’ll answer it in layers, from **physics → mathematics → numerical reasons**, and I’ll explicitly connect it to **periodic solids and ABINIT**.

---

## 1. The key physical reason: **periodicity of crystals**

In a crystal, the external potential from the ions is **periodic**:

$
V(\mathbf r + \mathbf R) = V(\mathbf r)
\quad\text{for any lattice vector }\mathbf R
$

Because of this, the electronic eigenstates obey **Bloch’s theorem**:

$
\psi_{n\mathbf k}(\mathbf r)
= e^{i\mathbf k\cdot \mathbf r}
u_{n\mathbf k}(\mathbf r),
\quad
u_{n\mathbf k}(\mathbf r+\mathbf R)=u_{n\mathbf k}(\mathbf r)
$

So the *nontrivial part* of the wavefunction, (u_{n\mathbf k}(\mathbf r)), is itself **periodic**.

---

## 2. Mathematical consequence: periodic functions → Fourier series

Any periodic function can be expanded exactly in a **Fourier series**:

$
u_{n\mathbf k}(\mathbf r)
=\sum_{\mathbf G}
c_{n\mathbf k}(\mathbf G)
e^{i\mathbf G\cdot \mathbf r}
$

where (\mathbf G) are **reciprocal lattice vectors**.

Substituting into Bloch’s form:

$
\boxed{
\psi_{n\mathbf k}(\mathbf r)
=\sum_{\mathbf G}
c_{n\mathbf k}(\mathbf G)
e^{i(\mathbf k+\mathbf G)\cdot \mathbf r}
}
$

So the reciprocal-space expansion is **not an arbitrary choice** — it is the *natural basis* for periodic eigenstates.

---

## 3. Why plane waves are so attractive numerically

### (a) They are **unbiased**

Plane waves:

* do **not depend on atomic positions**
* do **not favor any bonding geometry**

This makes them:

* ideal for **geometry optimization**
* robust for **high-pressure phases**
* transferable between systems

Atomic orbitals *do not* have this property.

---

### (b) Systematic convergence with one parameter

The basis is controlled by a **single number**:

$
\frac{\hbar^2}{2m}|\mathbf k+\mathbf G|^2 \le E_\text{cut}
$

→ include more plane waves by increasing `ecut`.

This gives:

* predictable convergence
* no basis-set superposition error (BSSE)

This is a *huge* practical advantage.

---

### (c) Kinetic energy operator becomes diagonal

In real space:
$
-\nabla^2 \psi(\mathbf r) \quad \text{(hard)}
$

In reciprocal space:

$
-\nabla^2 e^{i(\mathbf k+\mathbf G)\cdot\mathbf r}
=|\mathbf k+\mathbf G|^2 e^{i(\mathbf k+\mathbf G)\cdot\mathbf r}
$

So the kinetic energy is **exactly diagonal** in the plane-wave basis.

This is why FFT-based methods are so fast.

---

## 4. Why not expand in real-space atomic orbitals?

You *can* (Gaussian, numerical orbitals, etc.), but:

| Atomic orbitals      | Plane waves     |
| -------------------- | --------------- |
| System-dependent     | Universal       |
| Need re-optimization | No              |
| BSSE present         | BSSE-free       |
| Hard for metals      | Very natural    |
| Good for molecules   | Best for solids |

ABINIT is designed primarily for **extended periodic systems**, so plane waves are the natural choice.

---

## 5. Why reciprocal space is especially natural in DFT

Several key quantities simplify:

### Hartree potential

$
\nabla^2 V_H(\mathbf r) = -4\pi n(\mathbf r)
\quad\Rightarrow\quad
V_H(\mathbf G) = \frac{4\pi}{G^2} n(\mathbf G)
$

### Poisson equation

Solved **algebraically** in reciprocal space.

### FFT trick

* Wavefunctions stored in reciprocal space
* Potentials in real space
* FFT back and forth efficiently

This is the backbone of ABINIT, VASP, QE.

---

## 6. Physical intuition (important!)

You might be thinking:

> “But electrons are localized near atoms — why plane waves?”

Answer:

* **Localization comes from the coefficients** ($c_{n\mathbf k}(\mathbf G)$), not from the basis functions themselves.
* A localized function is just a superposition of many plane waves.

Analogy:

> A square pulse looks nothing like a sine wave, yet is *exactly* a Fourier series.

---

## 7. Why pseudopotentials are essential here

Core electrons → rapidly oscillating wavefunctions → need huge ($E_\text{cut}$)

Pseudopotentials:

* remove core states
* smooth the valence wavefunctions
* make plane-wave expansion efficient

Without pseudopotentials, plane-wave DFT would be impractical.

---

## 8. One-sentence answer

> **We expand (\psi_{n\mathbf k}) in reciprocal space because crystal wavefunctions are Bloch-periodic, plane waves form the natural basis for periodic functions, they diagonalize the kinetic energy, and they allow systematic, unbiased convergence with FFT efficiency.**

---

If you want, next we can:

* derive the **Kohn–Sham matrix equation in G-space**
* show **why cutoff energy corresponds to spatial resolution**
* compare plane waves vs **real-space grids**
* explain why **localized Wannier functions** come *after* plane waves

Just say where you want to go next.
