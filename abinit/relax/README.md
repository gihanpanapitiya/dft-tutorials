# Structure Optimization with ABINIT

Structure optimization in ABINIT finds equilibrium atomic positions and/or lattice parameters by minimizing total energy with respect to atomic forces and stresses.

## 📖 Theory

Optimization algorithms:
- **BFGS** (Broyden-Fletcher-Goldfarb-Shanno) - Quasi-Newton method
- **Conjugate gradient** - Steepest descent variant
- **Molecular dynamics** - Damped or simulated annealing

Types:
- `ionmov` - Controls atomic position optimization
- `optcell` - Controls cell shape/size optimization

## 🎯 What You'll Learn

- Optimize atomic positions with ABINIT
- Perform variable-cell relaxation
- Set force and stress tolerances
- Monitor optimization progress

## 📝 Example 1: Atomic Position Relaxation

### Input File (si_relax.abi)

```
# Silicon structure optimization
# Atomic relaxation only (fixed cell)

#Optimization parameters
ionmov 2              # BFGS ionic relaxation
ntime 50              # Maximum number of optimization steps
tolmxf 5.0d-6         # Tolerance on max force (Hartree/Bohr)

#Definition of the unit cell
acell 3*10.26         # Lattice constant in Bohr
rprim 0.0 0.5 0.5     # FCC primitive vectors
      0.5 0.0 0.5
      0.5 0.5 0.0

#Definition of the atom types and atoms
ntypat 1
znucl 14
natom 2
typat 1 1
xred                  # Slightly displaced from equilibrium
   0.0  0.0  0.0
   0.26 0.26 0.26

#Plane wave basis
ecut 30.0

#K-point grid
ngkpt 8 8 8
nshiftk 1
shiftk 0.0 0.0 0.0

#SCF parameters
nstep 50
toldfe 1.0d-10
diemac 12.0

#Output
prtwf 1
prtden 1

#Pseudopotential
pp_dirpath "$ABINIT_PP_PATH"
pseudos "Si.psp8"
```

### Key Parameters for Relaxation

#### Ionic motion:
- `ionmov` - Ion dynamics algorithm
  - `2` - BFGS (recommended)
  - `3` - Conjugate gradient (old method)
  - `7` - LBFGS (memory-efficient)
  - `22` - BFGS with lattice parameters (similar to optcell)
- `ntime` - Maximum optimization steps
- `tolmxf` - Max force tolerance (Ha/Bohr)
  - Typical: `5.0d-6` to `1.0d-5`

## 📝 Example 2: Variable Cell Relaxation

### Input File (si_vc-relax.abi)

```
# Silicon variable-cell optimization
# Both atoms and cell are relaxed

#Optimization parameters
ionmov 2              # BFGS ionic relaxation
optcell 2             # Full cell optimization
ntime 50
tolmxf 5.0d-6         # Force tolerance
strten1 1.0d-8        # Stress tolerance (not used for optcell)
ecutsm 0.5            # Energy cutoff smearing (for optcell)

#Definition of the unit cell (approximate)
acell 3*10.20         # Slightly off from equilibrium
rprim 0.0 0.5 0.5
      0.5 0.0 0.5
      0.5 0.5 0.0

#Definition of the atom types and atoms
ntypat 1
znucl 14
natom 2
typat 1 1
xred
   0.0  0.0  0.0
   0.25 0.25 0.25

#Plane wave basis
ecut 30.0

#K-point grid
ngkpt 8 8 8
nshiftk 1
shiftk 0.0 0.0 0.0

#SCF parameters
nstep 50
toldfe 1.0d-10
diemac 12.0

#Output
prtwf 1
prtden 1

#Pseudopotential
pp_dirpath "$ABINIT_PP_PATH"
pseudos "Si.psp8"
```

### Cell Optimization Parameters

- `optcell` - Cell degree of freedom
  - `0` - No cell optimization (only atoms)
  - `1` - Only volume (constant shape)
  - `2` - Full optimization (volume and shape)
  - `3` - Constant volume (only shape)
  - `4`, `5`, `6` - Specific axis constraints
- `ecutsm` - Smearing of `ecut` when cell changes
  - Helps avoid egg-box effect
  - Typical value: `0.5` Hartree
- `dilatmx` - Max cell size change (default 1.15)
  - Prevents too large steps

## 🚀 Running the Calculation

```bash
# Atomic relaxation
abinit si_relax.abi > si_relax.log

# Variable cell relaxation
abinit si_vc-relax.abi > si_vc-relax.log

# Parallel
mpirun -np 4 abinit si_vc-relax.abi > si_vc-relax.log
```

## 📊 Reading the Output

### Optimization Progress

Each ionic step shows:

```
--- Iteration (  1) for the ion step number   1
 
 ITER STEP NUMBER     1
 vtorho : nnsclo_now=  2
 Total charge density [el/Bohr^3]
...

 ETOT  1  -8.7123456789012

 At SCF step    X, etot is converged :
  for the second time, diff in etot=  X.XXX < toldfe=  1.000E-10
  
 Cartesian components of stress tensor (hartree/bohr^3)
  sigma(1 1)=  X.XXXE-XX
  ...
  
 cartesian forces (hartree/bohr) at end:
    1      0.00012345     0.00012345     0.00012345
    2     -0.00012345    -0.00012345    -0.00012345
 frms,max,avg= 2.137E-04 2.137E-04   0.000E+00  0.000E+00  0.000E+00 h/b
```

### Convergence Message

```
 At the end of the iteration:
 
 BFGS update: ---- Geometry Optimization Achieved ----
  
 Effective mass at the end:
      rhog =  1.234567890123456E-01
      
 Forces are below tolerance.
 
 Geometry optimization is CONVERGED.
```

### Final Structure

```
 Final (x,y,z) coordinates of atoms:
     1   0.000000000000    0.000000000000    0.000000000000
     2   0.250000000000    0.250000000000    0.250000000000
```

For variable cell:

```
 acell(angstrom)      5.431E+00  5.431E+00  5.431E+00
 
 rprim:
  0.0000000000E+00  5.0000000000E-01  5.0000000000E-01
  5.0000000000E-01  0.0000000000E+00  5.0000000000E-01
  5.0000000000E-01  5.0000000000E-01  0.0000000000E+00
```

## 🔍 Monitoring Optimization

### Extract energies per step:
```bash
grep "ETOT" si_relax.log | grep "Iteration"
```

### Extract forces:
```bash
grep "frms,max,avg" si_relax.log
```

### Check convergence:
```bash
grep "Geometry optimization is CONVERGED" si_relax.log
```

### Get final positions:
```bash
grep -A 10 "Final (x,y,z)" si_relax.log
```

## 🎨 Post-Processing

### Python script to monitor optimization:

```python
import re
import numpy as np
import matplotlib.pyplot as plt

def parse_relax_output(filename):
    """Extract energies and forces from ABINIT relaxation"""
    energies = []
    max_forces = []
    
    with open(filename, 'r') as f:
        content = f.read()
    
    # Find ionic step sections
    ionic_pattern = r'--- Iteration.*?ion step number\s+(\d+)'
    energy_pattern = r'>>>>>>>>> Etotal=\s+([-\d.E+]+)'
    force_pattern = r'frms,max,avg=\s+([\d.E+-]+)\s+([\d.E+-]+)'
    
    # Extract energies
    for match in re.finditer(energy_pattern, content):
        energies.append(float(match.group(1)))
    
    # Extract max forces
    for match in re.finditer(force_pattern, content):
        max_forces.append(float(match.group(2)))
    
    return energies, max_forces

# Parse output
energies, forces = parse_relax_output('si_relax.log')

# Plot
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

# Energy convergence
ax1.plot(energies, 'o-')
ax1.set_xlabel('Ionic Step')
ax1.set_ylabel('Total Energy (Ha)')
ax1.set_title('Energy Convergence')
ax1.grid(True)

# Force convergence
ax2.plot(forces, 'o-')
ax2.axhline(y=5e-6, color='r', linestyle='--', label='Threshold')
ax2.set_xlabel('Ionic Step')
ax2.set_ylabel('Max Force (Ha/Bohr)')
ax2.set_title('Force Convergence')
ax2.set_yscale('log')
ax2.legend()
ax2.grid(True)

plt.tight_layout()
plt.savefig('relax_convergence.png', dpi=150)

print(f"Final energy: {energies[-1]:.10f} Ha")
print(f"Converged in {len(energies)} steps")
```

## ⚙️ Advanced Options

### Multiple datasets for SCF + relax:

```
ndtset 2

# Dataset 1: SCF with reasonable tolerance
ionmov1 0             # No relaxation
toldfe1 1.0d-8

# Dataset 2: Relaxation with tight SCF
ionmov2 2
ntime2 50
tolmxf2 5.0d-6
toldfe2 1.0d-10
getden2 1             # Use density from dataset 1

# Common parameters
acell 3*10.26
...
```

### Constrained relaxation:

Fix specific atoms using `iatfix`:

```
iatfix 1              # Fix atom 1 (all coordinates)

# Or fix specific directions
iatfixx 1             # Fix x-coordinate of atom 1
iatfixy 1             # Fix y-coordinate of atom 1
iatfixz 1             # Fix z-coordinate of atom 1
```

### Apply external pressure:

```
optcell 2
strtarget -1.0d-6 -1.0d-6 -1.0d-6 0.0 0.0 0.0  # Target stress (Ha/Bohr^3)
# Negative for compression, positive for tension
```

## 🐛 Common Issues

### Issue: Optimization not converging
**Solutions**:
- Check starting structure is reasonable
- Try different algorithm: `ionmov 3` (CG) instead of BFGS
- Tighten SCF: reduce `toldfe`
- Increase `ntime`
- Relax `tolmxf` slightly

### Issue: Forces oscillating
**Solutions**:
- Tighten `toldfe` (e.g., `1.0d-12`)
- Increase `ecut`
- Use denser k-point grid
- Check pseudopotentials

### Issue: Cell optimization unstable
**Solutions**:
- Start with better `acell` guess
- Use `ecutsm 0.5` to smooth energy surface
- Reduce `dilatmx` (max cell change)
- Use `optcell 1` (volume only) first

### Issue: Egg-box effect
**Solutions**:
- Add `ecutsm 0.5` or higher
- Increase `ecut`
- Check that structure makes sense

## 📚 Example: 2D Material (Graphene)

```
# Graphene optimization
# Relax in-plane, fix out-of-plane

#Optimization
ionmov 2
optcell 2             # Optimize cell
ntime 30
tolmxf 5.0d-6
ecutsm 0.5

#Unit cell (hexagonal)
acell 2*4.65 15.0     # a, a, c (vacuum in z)
rprim 0.866025404  0.5 0.0
     -0.866025404  0.5 0.0
      0.0           0.0 1.0

#Atoms
ntypat 1
znucl 6
natom 2
typat 1 1
xred
   1/3  1/3  0.5
   2/3  2/3  0.5

#Basis and k-points
ecut 40.0
ngkpt 12 12 1         # No k-points in z
nshiftk 1
shiftk 0.0 0.0 0.0

#SCF
nstep 50
toldfe 1.0d-10
diemac 2.0

#Pseudopotential
pp_dirpath "$ABINIT_PP_PATH"
pseudos "C.psp8"
```

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- `ionmov 2` for atomic relaxation (BFGS)
- `optcell 2` for full variable-cell optimization
- Monitor forces and energies
- Use `ecutsm` for cell optimization
- Check final structure is physically reasonable
