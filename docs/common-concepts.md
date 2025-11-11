# Common Concepts in DFT Calculations

This guide explains fundamental concepts that apply across all DFT codes (Quantum ESPRESSO, ABINIT, SIESTA, CP2K).

## 🧮 Density Functional Theory (DFT) Basics

### What is DFT?

DFT is a quantum mechanical method to calculate the electronic structure of many-body systems. Instead of dealing with the complex many-electron wavefunction, DFT works with the much simpler electron density.

**Key theorem**: The ground state properties of a many-electron system are uniquely determined by the electron density ρ(r).

### Exchange-Correlation Functionals

The accuracy of DFT depends on the exchange-correlation (XC) functional:

#### Local Density Approximation (LDA)
- **Pros**: Computationally cheap, often accurate for lattice constants
- **Cons**: Overbinds (lattice constants too small), poor for molecules
- **Use when**: Quick calculations, metals, preliminary studies
- **Common**: `PZ81`, `PW92`

#### Generalized Gradient Approximation (GGA)
- **Pros**: Better than LDA for most systems, good energies
- **Cons**: Still underestimates band gaps
- **Use when**: Most production calculations (default choice)
- **Common**: `PBE` (most popular), `PBEsol` (solids), `BLYP`

#### Hybrid Functionals
- **Pros**: Better band gaps, reaction barriers
- **Cons**: 10-100× more expensive
- **Use when**: Band gaps important, high accuracy needed
- **Common**: `PBE0`, `HSE06`, `B3LYP`

#### Meta-GGA
- **Pros**: Between GGA and hybrid in accuracy/cost
- **Cons**: More complex, not always better
- **Common**: `SCAN`, `TPSS`

**Recommendation**: Start with **PBE** for most systems.

## ⚡ Basis Sets

Different codes use different basis sets:

### Plane Waves (QE, ABINIT)

Pros:
- Systematic convergence (increase cutoff)
- No basis set superposition error
- Periodic systems natural

Cons:
- Require pseudopotentials
- Less efficient for isolated molecules
- Need large cutoffs for heavy elements

**Convergence parameter**: Energy cutoff (ecutwfc, ecut)

### Atomic Orbitals (SIESTA)

Pros:
- Efficient for large systems
- Natural for molecules
- O(N) scaling possible

Cons:
- Basis set superposition error
- Less systematic convergence
- Must test basis size

**Convergence parameters**: Basis size (SZ, DZ, DZP, TZP), PAO.EnergyShift

### Gaussians + Plane Waves (CP2K)

Pros:
- Combines benefits of both
- Efficient for molecules and solids
- Good for mixed systems

Cons:
- Two cutoffs to converge
- More complex

**Convergence parameters**: CUTOFF, REL_CUTOFF

## 🔢 K-Point Sampling

K-points sample the Brillouin zone (reciprocal space) for periodic systems.

### Why K-Points?

Bloch's theorem states that wavefunctions in periodic systems have the form:
$$\psi_{n\mathbf{k}}(\mathbf{r}) = e^{i\mathbf{k}\cdot\mathbf{r}} u_{n\mathbf{k}}(\mathbf{r})$$

We must integrate over all **k** vectors, approximated by sampling.

### Monkhorst-Pack Grids

Most common: uniform grid of k-points
- Specified as: nx × ny × nz
- Denser grid = more accurate but slower

### How Many K-Points?

**Rule of thumb**: k-spacing ≈ 0.03-0.05 Å⁻¹

For a cell with lattice constant *a*:
- If *a* = 5 Å → ~6×6×6 grid
- If *a* = 10 Å → ~3×3×3 grid
- If *a* = 20 Å → 1×1×1 (Gamma point)

### System-Specific Guidelines:

| System Type | K-Points | Notes |
|-------------|----------|-------|
| Molecules (large box) | 1×1×1 (Gamma) | No periodicity needed |
| 2D materials | 12×12×1 | Dense in plane, none perpendicular |
| Bulk metals | 12×12×12+ | Need dense sampling |
| Bulk semiconductors | 6×6×6 to 8×8×8 | Moderate sampling |
| Large supercells | 2×2×2 | Already large cell |

### Convergence Testing

Always test k-point convergence:

```python
# Example convergence test
kpoints = [2, 4, 6, 8, 10, 12, 14]
energies = []

for k in kpoints:
    # Run calculation with k×k×k grid
    # Extract energy
    energies.append(energy)

# Plot and find where energy changes < 0.001 eV
```

## 🎯 SCF Convergence

Self-Consistent Field iterations:

### Convergence Criteria

Different codes use different criteria:

- **Total energy** (`toldfe` in ABINIT): Change in total energy
- **Density** (`conv_thr` in QE): Change in charge density
- **Forces** (`toldff`): Max force component
- **Wavefunction** (`tolwfr`): Wavefunction residual

**Typical values**:
- Standard: 10⁻⁶ to 10⁻⁸ Ry (energy)
- Tight: 10⁻⁸ to 10⁻¹⁰ Ry (for forces)
- Relaxation: 10⁻⁸ Ry minimum

### Mixing Schemes

SCF mixes old and new densities:

$$\rho_{n+1} = (1-\alpha)\rho_n + \alpha\rho_{out}$$

- Small α (0.1-0.3): More stable, slower
- Large α (0.5-0.7): Faster, may not converge

**Advanced**: Pulay mixing, Broyden mixing (use code defaults)

### Troubleshooting Non-Convergence

1. **Reduce mixing** parameter
2. **Use better initial guess**
3. **Increase cutoff** (often helps)
4. **For metals**: Use smearing
5. **Check input** for errors
6. **Try different mixing scheme**

## 🌡️ Temperature and Smearing (Metals)

Metals have partially filled bands → need electronic temperature.

### Why Smearing?

At T=0, the Fermi surface is infinitely sharp → very dense k-points needed.

Smearing broadens occupation: smooth transitions between occupied/empty.

### Smearing Methods

1. **Gaussian**: Simple, smooth
2. **Fermi-Dirac**: Physical (thermal distribution)
3. **Methfessel-Paxton**: Minimizes error in total energy
4. **Marzari-Vanderbilt (mv)**: Cold smearing, good for metals

**QE**: `occupations='smearing'`, `smearing='mv'`, `degauss=0.02 Ry`
**ABINIT**: `occopt=3`, `tsmear=0.01 Ha`
**SIESTA**: `ElectronicTemperature 300 K`
**CP2K**: `&SMEAR ... METHOD FERMI_DIRAC`

**Typical smearing width**: 
- 0.01-0.03 Ha (300-1000 K)
- Should be smaller than smallest energy differences of interest
- Test that results don't depend strongly on smearing width

## 📊 Total Energy vs. Free Energy

For T > 0:
- **Total energy E**: Electronic energy
- **Free energy F = E - TS**: Thermodynamic potential

For geometry optimization with smearing, minimize free energy!

## 🔬 Pseudopotentials

Replace core electrons with effective potential.

### Types

1. **Norm-conserving (NC)**
   - Pros: Accurate, transferable
   - Cons: Higher cutoffs needed

2. **Ultrasoft (US)**
   - Pros: Lower cutoffs
   - Cons: More complex

3. **PAW (Projector Augmented Wave)**
   - Pros: Most accurate, all-electron like
   - Cons: More expensive than US

### Choosing Pseudopotentials

**Quality libraries**:
- **SSSP**: [Materials Cloud](https://www.materialscloud.org/discover/sssp/)
- **PseudoDojo**: [pseudo-dojo.org](http://www.pseudo-dojo.org/)
- **Quantum ESPRESSO**: Official library
- **ABINIT**: JTH PAW table
- **CP2K**: GTH potentials

**Check**:
- Number of valence electrons
- Functional match (PBE pseudopotential for PBE calculation)
- Cutoff recommendations
- Transferability tests

## 🎯 Convergence Philosophy

**Golden rule**: Always test convergence!

### What to Converge

1. **Energy cutoff** (plane-wave codes)
2. **K-point density**
3. **Basis set size** (SIESTA)
4. **Cell size** (molecules, surfaces)
5. **Vacuum thickness** (2D materials, surfaces)

### Convergence Procedure

For each parameter:
1. Fix all other parameters
2. Vary the parameter systematically
3. Calculate property of interest
4. Plot results
5. Choose value where property changes < threshold

**Thresholds**:
- Energy: < 1 meV/atom
- Forces: < 0.01 eV/Å
- Lattice constant: < 0.01 Å
- Band gap: < 0.01 eV

### Practical Workflow

```
1. Quick test with loose parameters
2. Converge cutoff (keep k-points moderate)
3. Converge k-points (use converged cutoff)
4. Production calculations with converged parameters
```

## 📏 Units Across Codes

| Quantity | QE | ABINIT | SIESTA | CP2K |
|----------|----|---------| -------|------|
| Energy | Ry | Ha | eV | Ha |
| Length | Bohr | Bohr | Ang/Bohr | Ang/Bohr |
| Force | Ry/Bohr | Ha/Bohr | eV/Ang | Ha/Bohr |
| Cutoff | Ry | Ha | Ry | Ry |

**Conversions**:
- 1 Ry = 0.5 Ha = 13.606 eV
- 1 Ha = 2 Ry = 27.211 eV
- 1 Bohr = 0.529177 Å
- 1 eV/Å = 0.0194469 Ha/Bohr

## 🔍 Common Pitfalls

1. **Not testing convergence**: Always verify!
2. **Mismatched functional and pseudopotential**: Use PBE pseudo with PBE functional
3. **Too loose convergence**: Especially for forces (needs tight SCF)
4. **Forgetting smearing for metals**: Will not converge!
5. **Wrong units**: Check your input carefully
6. **Insufficient k-points for metals**: Need dense sampling
7. **Comparing absolute energies**: Only energy differences meaningful
8. **Cell too small**: Especially for molecules and defects

## 📚 Further Reading

- **Martin's book**: "Electronic Structure: Basic Theory and Practical Methods"
- **Density Functional Theory**: Parr & Yang
- **Plane Waves Review**: Payne et al., Rev. Mod. Phys. 64, 1045 (1992)

---

**Key Takeaways**:
- PBE is the standard functional
- Always test convergence (cutoff + k-points)
- Metals need smearing
- Only energy differences are meaningful
- Check pseudopotential matches functional
