# Convergence Testing Guide

Convergence testing ensures your DFT results are reliable and independent of numerical parameters. This guide explains how to systematically test and achieve convergence.

## 🎯 Why Convergence Testing?

DFT calculations involve approximations:
- Finite basis sets (cutoffs, basis size)
- Discrete k-point sampling
- Finite SCF iterations
- Finite cell size

**Results must be independent of these choices!**

## 📊 What to Converge

### 1. Energy Cutoff (Plane-Wave Codes)

**QE/ABINIT**: Test `ecutwfc`/`ecut`
**SIESTA**: Test `MeshCutoff`
**CP2K**: Test `CUTOFF` and `REL_CUTOFF`

### 2. K-Point Sampling

All codes: Test k-point mesh density

### 3. Basis Set Size (SIESTA)

Test: SZ → DZ → DZP → TZP

### 4. SCF Convergence

Ensure SCF actually converged (check output!)

### 5. Cell Size (Molecules, Surfaces, Defects)

Test vacuum spacing, supercell size

## 🔬 Convergence Methodology

### General Procedure

For each parameter:
1. **Fix all other parameters** at reasonable values
2. **Vary the parameter** systematically
3. **Calculate property** of interest
4. **Plot** results
5. **Determine convergence** threshold
6. **Choose** converged value

### Convergence Criteria

Different properties need different thresholds:

| Property | Threshold | Use |
|----------|-----------|-----|
| Total energy | < 1 meV/atom | Energetics |
| Energy differences | < 10 meV | Reaction energies |
| Forces | < 0.01 eV/Å | Structure optimization |
| Lattice constant | < 0.01 Å | Crystal structure |
| Band gap | < 0.01 eV | Electronic properties |
| Stress | < 0.1 GPa | Elastic properties |

**Choose based on what you need!**

## ⚡ Energy Cutoff Convergence

### Quantum ESPRESSO Example

```bash
#!/bin/bash
# Test ecutwfc convergence

for ecut in 20 25 30 35 40 50 60 70 80; do
    cat > pw_${ecut}.in << EOF
&CONTROL
  calculation = 'scf'
  prefix = 'test'
  pseudo_dir = './pseudo/'
  outdir = './tmp/'
/
&SYSTEM
  ibrav = 2
  celldm(1) = 10.26
  nat = 2
  ntyp = 1
  ecutwfc = ${ecut}
  ecutrho = $((ecut * 8))
/
&ELECTRONS
  conv_thr = 1.0d-8
/
ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF
ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25
K_POINTS (automatic)
  8 8 8 0 0 0
EOF

    mpirun -np 4 pw.x < pw_${ecut}.in > pw_${ecut}.out
    
    # Extract energy
    grep "!    total energy" pw_${ecut}.out | awk '{print $5}' >> energies.dat
done
```

### Python Analysis

```python
import numpy as np
import matplotlib.pyplot as plt

# Read data
cutoffs = np.array([20, 25, 30, 35, 40, 50, 60, 70, 80])
energies = np.loadtxt('energies.dat')

# Convert to meV/atom relative to highest cutoff
natoms = 2
ref_energy = energies[-1]
rel_energies = (energies - ref_energy) * 1000 / natoms  # meV/atom

# Plot
plt.figure(figsize=(8, 6))
plt.plot(cutoffs, rel_energies, 'o-', markersize=8)
plt.axhline(y=1, color='r', linestyle='--', label='1 meV/atom threshold')
plt.axhline(y=-1, color='r', linestyle='--')
plt.xlabel('Energy Cutoff (Ry)')
plt.ylabel('Energy Difference (meV/atom)')
plt.title('Energy Cutoff Convergence')
plt.grid(True, alpha=0.3)
plt.legend()
plt.savefig('cutoff_convergence.png', dpi=150)

# Find converged cutoff
threshold = 1.0  # meV/atom
for i, (ecut, delta_e) in enumerate(zip(cutoffs, rel_energies)):
    if abs(delta_e) < threshold and all(abs(rel_energies[i:]) < threshold):
        print(f"Converged at {ecut} Ry")
        break
```

### Typical Cutoff Values

| Element Type | USPP (Ry) | NC (Ry) | PAW (Ry) |
|--------------|-----------|---------|----------|
| Light (H, C, N, O) | 30-50 | 50-80 | 40-60 |
| Medium (Si, Al) | 25-40 | 40-60 | 30-50 |
| Transition metals | 40-60 | 60-100 | 40-70 |
| Heavy elements | 50-80 | 80-120 | 50-80 |

## 🔷 K-Point Convergence

### Test Script (Quantum ESPRESSO)

```bash
#!/bin/bash
# K-point convergence test

for k in 2 4 6 8 10 12 16; do
    cat > kpt_${k}.in << EOF
&CONTROL
  calculation = 'scf'
  prefix = 'ktest'
/
&SYSTEM
  ibrav = 2
  celldm(1) = 10.26
  nat = 2
  ntyp = 1
  ecutwfc = 40
/
&ELECTRONS
  conv_thr = 1.0d-8
/
ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF
ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25
K_POINTS (automatic)
  ${k} ${k} ${k} 0 0 0
EOF

    mpirun -np 4 pw.x < kpt_${k}.in > kpt_${k}.out
done
```

### K-Point Density Rule

**Rule of thumb**: k-spacing ≈ 0.03-0.05 Å⁻¹

For lattice constant *a* (in Å):
$$n_k \approx \frac{2\pi}{a \times k_{spacing}}$$

Examples:
- Si (a = 5.43 Å): 8×8×8 (0.042 Å⁻¹ spacing)
- Al (a = 4.05 Å): 12×12×12 (0.039 Å⁻¹ spacing)

### System-Specific Guidelines

**Molecules** (large vacuum):
- Usually 1×1×1 (Gamma point only)
- No periodicity needed

**Semiconductors**:
- Standard: 6×6×6 to 8×8×8
- For DOS/bands: 12×12×12+

**Metals**:
- Need dense sampling: 12×12×12 minimum
- Use smearing!
- Test more carefully

**2D Materials**:
- Dense in-plane: 12×12×1
- Single point perpendicular

**Large supercells** (> 20 Å):
- 2×2×2 or even 1×1×1
- Cell size provides convergence

## 📐 Basis Set Convergence (SIESTA)

### Test Script

```bash
#!/bin/bash
# SIESTA basis set convergence

for basis in SZ DZ DZP TZP; do
    cat > basis_${basis}.fdf << EOF
SystemLabel test_${basis}
PAO.BasisSize ${basis}
PAO.EnergyShift 0.01 Ry
MeshCutoff 300.0 Ry
# ... rest of input ...
EOF

    siesta < basis_${basis}.fdf > basis_${basis}.out
done
```

### Expected Behavior

- SZ → DZ: Large change (few tenths eV)
- DZ → DZP: Moderate change (~0.1 eV)
- DZP → TZP: Small change (< 0.05 eV)

**Recommendation**: DZP usually sufficient, TZP for high accuracy

## 🎨 Comprehensive Convergence Study

### Complete Python Script

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.gridspec import GridSpec

class ConvergenceStudy:
    def __init__(self, name):
        self.name = name
        self.results = {}
    
    def test_cutoff(self, cutoffs, energies):
        """Analyze cutoff convergence"""
        self.results['cutoff'] = {
            'values': cutoffs,
            'energies': energies
        }
        
        # Relative to highest cutoff
        ref = energies[-1]
        delta = (energies - ref) * 1000 / self.natoms
        
        # Find converged value (within 1 meV/atom)
        threshold = 1.0
        for i, (cut, de) in enumerate(zip(cutoffs, delta)):
            if all(np.abs(delta[i:]) < threshold):
                self.results['cutoff_converged'] = cut
                break
        
        return delta
    
    def test_kpoints(self, kgrids, energies):
        """Analyze k-point convergence"""
        self.results['kpoints'] = {
            'values': kgrids,
            'energies': energies
        }
        
        ref = energies[-1]
        delta = (energies - ref) * 1000 / self.natoms
        
        # Find converged grid
        threshold = 1.0
        for i, (k, de) in enumerate(zip(kgrids, delta)):
            if all(np.abs(delta[i:]) < threshold):
                self.results['kpoint_converged'] = k
                break
        
        return delta
    
    def plot_all(self):
        """Create comprehensive convergence plot"""
        fig = plt.figure(figsize=(12, 5))
        gs = GridSpec(1, 2, figure=fig)
        
        # Cutoff convergence
        ax1 = fig.add_subplot(gs[0, 0])
        if 'cutoff' in self.results:
            cutoffs = self.results['cutoff']['values']
            energies = self.results['cutoff']['energies']
            ref = energies[-1]
            delta = (energies - ref) * 1000 / self.natoms
            
            ax1.plot(cutoffs, delta, 'o-', markersize=8)
            ax1.axhline(y=1, color='r', linestyle='--', alpha=0.5)
            ax1.axhline(y=-1, color='r', linestyle='--', alpha=0.5)
            ax1.set_xlabel('Energy Cutoff (Ry)')
            ax1.set_ylabel('ΔE (meV/atom)')
            ax1.set_title('Cutoff Convergence')
            ax1.grid(True, alpha=0.3)
            
            if 'cutoff_converged' in self.results:
                ax1.axvline(x=self.results['cutoff_converged'], 
                           color='g', linestyle=':', alpha=0.5,
                           label=f"Converged: {self.results['cutoff_converged']} Ry")
                ax1.legend()
        
        # K-point convergence
        ax2 = fig.add_subplot(gs[0, 1])
        if 'kpoints' in self.results:
            kgrids = self.results['kpoints']['values']
            energies = self.results['kpoints']['energies']
            ref = energies[-1]
            delta = (energies - ref) * 1000 / self.natoms
            
            ax2.plot(kgrids, delta, 's-', markersize=8)
            ax2.axhline(y=1, color='r', linestyle='--', alpha=0.5)
            ax2.axhline(y=-1, color='r', linestyle='--', alpha=0.5)
            ax2.set_xlabel('K-grid (N×N×N)')
            ax2.set_ylabel('ΔE (meV/atom)')
            ax2.set_title('K-point Convergence')
            ax2.grid(True, alpha=0.3)
            
            if 'kpoint_converged' in self.results:
                ax2.axvline(x=self.results['kpoint_converged'],
                           color='g', linestyle=':', alpha=0.5,
                           label=f"Converged: {self.results['kpoint_converged']}³")
                ax2.legend()
        
        plt.tight_layout()
        plt.savefig(f'{self.name}_convergence.png', dpi=150)
        return fig

# Usage example
study = ConvergenceStudy('silicon')
study.natoms = 2

# Test cutoff
cutoffs = np.array([20, 25, 30, 35, 40, 50, 60])
cutoff_energies = np.array([-15.82, -15.84, -15.845, -15.847, -15.8472, -15.8473, -15.8473])
study.test_cutoff(cutoffs, cutoff_energies)

# Test k-points
kgrids = np.array([4, 6, 8, 10, 12, 16])
kpt_energies = np.array([-15.82, -15.843, -15.847, -15.8472, -15.8472, -15.8472])
study.test_kpoints(kgrids, kpt_energies)

# Plot
study.plot_all()

# Report
print("="*50)
print("CONVERGENCE STUDY RESULTS")
print("="*50)
print(f"Converged cutoff: {study.results.get('cutoff_converged', 'Not converged')} Ry")
print(f"Converged k-grid: {study.results.get('kpoint_converged', 'Not converged')}³")
print("="*50)
```

## 💡 Practical Tips

### 1. Convergence Order
1. **First**: Cutoff (with moderate k-points)
2. **Second**: K-points (with converged cutoff)
3. **Then**: Production calculations

### 2. Starting Values

**Quick initial test**:
- Cutoff: Use recommended value from pseudopotential
- K-points: Moderate grid (6×6×6 for bulk)
- SCF: Default tolerances

**Then refine** based on convergence tests.

### 3. Computational Efficiency

- Use coarser parameters for testing/debugging
- Use converged parameters for production
- Save converged wavefunctions/density for restarts

### 4. System-Dependent

Convergence requirements vary:
- **Metals**: Need denser k-points
- **Large gaps**: Converge faster
- **Magnetic systems**: May need tighter tolerances
- **Forces/phonons**: Need very tight SCF

### 5. Document Everything!

Keep notes on:
- Which parameters tested
- Convergence thresholds used
- Final chosen values
- Why you made those choices

## ⚠️ Common Mistakes

1. **Not testing at all**: "Default should be fine" - NO!
2. **Testing one parameter only**: Must test all!
3. **Wrong reference**: Always use highest value as reference
4. **Too loose threshold**: 10 meV/atom often too loose
5. **Assuming transferability**: Different systems need different parameters
6. **Not re-checking**: Changed functional? Test again!

## 📊 Example Convergence Table (for Paper)

```
Table: Convergence parameters for bulk silicon

Parameter          Tested Values      Converged Value   Criterion
-------------------------------------------------------------------
Energy cutoff      20-80 Ry          40 Ry             < 1 meV/atom
K-point grid       4³-16³            8×8×8             < 1 meV/atom
SCF tolerance      10⁻⁶-10⁻¹⁰ Ry     10⁻⁸ Ry           Default
Force tolerance    -                 10⁻⁴ Ry/bohr      For relaxation
-------------------------------------------------------------------
```

## 🔗 Further Reading

- "Convergence and Validation", QE tutorial
- SSSP paper (Prandini et al., 2018)
- Your code's manual!

---

**Key Takeaways**:
- Always test convergence systematically
- Plot your results
- Use appropriate thresholds for your needs
- Document your choices
- Different properties need different convergence
