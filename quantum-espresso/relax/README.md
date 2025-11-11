# Structure Optimization with Quantum ESPRESSO

Structure optimization (also called geometry relaxation or energy minimization) finds the atomic positions and/or cell parameters that minimize the total energy, giving you the equilibrium structure.

## 📖 Theory

During optimization:
1. Calculate forces on atoms and stress on cell
2. Move atoms/cell in direction that reduces energy
3. Recalculate forces/stress
4. Repeat until forces < threshold

Types of optimization:
- **Atomic relaxation** (`relax`) - Optimize atomic positions, fixed cell
- **Variable cell** (`vc-relax`) - Optimize both atoms and cell parameters

## 🎯 What You'll Learn

- Perform atomic position relaxation
- Perform full variable-cell relaxation
- Set force and stress convergence criteria
- Monitor optimization progress

## 📝 Example 1: Atomic Relaxation (relax)

### Input File (si_relax.in)

```fortran
&CONTROL
  calculation = 'relax'
  prefix = 'silicon'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
  forc_conv_thr = 1.0d-4
  tprnfor = .true.
  tstress = .true.
/

&SYSTEM
  ibrav = 2
  celldm(1) = 10.26
  nat = 2
  ntyp = 1
  ecutwfc = 30.0
  ecutrho = 240.0
/

&ELECTRONS
  conv_thr = 1.0d-8
  mixing_beta = 0.7
/

&IONS
  ion_dynamics = 'bfgs'
/

ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.26 0.26 0.26

K_POINTS (automatic)
  8 8 8 0 0 0
```

### Key Parameters for Relaxation

#### &CONTROL:
- `calculation = 'relax'` - Atomic position optimization
- `forc_conv_thr = 1.0d-4` - Force convergence threshold (Ry/bohr)

#### &IONS namelist (new):
- `ion_dynamics = 'bfgs'` - Quasi-Newton optimization algorithm
  - Other options: `'damp'` (damped dynamics), `'cg'` (conjugate gradient)
- `upscale = 100` - (optional) Max step size scaling
- `bfgs_ndim = 1` - (optional) Number of old forces used

## 📝 Example 2: Variable Cell Relaxation (vc-relax)

### Input File (si_vc-relax.in)

```fortran
&CONTROL
  calculation = 'vc-relax'
  prefix = 'silicon'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
  forc_conv_thr = 1.0d-4
  tprnfor = .true.
  tstress = .true.
/

&SYSTEM
  ibrav = 2
  celldm(1) = 10.20
  nat = 2
  ntyp = 1
  ecutwfc = 30.0
  ecutrho = 240.0
/

&ELECTRONS
  conv_thr = 1.0d-8
  mixing_beta = 0.7
/

&IONS
  ion_dynamics = 'bfgs'
/

&CELL
  cell_dynamics = 'bfgs'
  press_conv_thr = 0.5
/

ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25

K_POINTS (automatic)
  8 8 8 0 0 0
```

### Key Parameters for Variable Cell

#### &CELL namelist (new):
- `cell_dynamics = 'bfgs'` - Cell optimization algorithm
  - Other options: `'damp-pr'`, `'damp-w'`
- `press_conv_thr = 0.5` - Pressure convergence threshold (kbar)
- `press = 0.0` - (optional) Target pressure (kbar)
- `cell_dofree = 'all'` - (optional) Degrees of freedom
  - `'ibrav'` - Only volume changes
  - `'x'`, `'y'`, `'z'` - Relax specific directions
  - `'shape'` - Fix volume, relax shape

## 🚀 Running the Calculation

```bash
# Atomic relaxation
pw.x < si_relax.in > si_relax.out

# Variable cell relaxation
pw.x < si_vc-relax.in > si_vc-relax.out

# Parallel
mpirun -np 4 pw.x < si_vc-relax.in > si_vc-relax.out
```

## 📊 Reading the Output

### Optimization Progress

For each ionic step, you'll see:

```
     iteration #  1     ecut=    30.00 Ry     beta= 0.70
     ...
!    total energy              =     -15.84523410 Ry

     Forces acting on atoms (cartesian axes, Ry/au):
     atom    1 type  1   force =     0.00124567    0.00124567    0.00124567
     atom    2 type  1   force =    -0.00124567   -0.00124567   -0.00124567

     Total force =     0.003045     Total SCF correction =     0.000012
```

### Convergence Message

```
     A final scf calculation at the relaxed structure.
     The G-vectors are recalculated for the final unit cell
     Results may differ from those at the preceding step.

     End of BFGS Geometry Optimization

     Final energy   =     -15.8452341000 Ry
```

### Optimized Structure

```
ATOMIC_POSITIONS (alat)
Si       0.000000000   0.000000000   0.000000000
Si       0.250000000   0.250000000   0.250000000

     Writing output data file silicon.save/
```

For `vc-relax`, you'll also see:

```
CELL_PARAMETERS (alat= 10.26000000)
   -0.500000000   0.000000000   0.500000000
    0.000000000   0.500000000   0.500000000
   -0.500000000   0.500000000   0.000000000
```

## 🔍 Monitoring the Optimization

### Extract energies per step:
```bash
grep "!    total energy" si_relax.out
```

### Extract forces:
```bash
grep -A 5 "Forces acting" si_relax.out
```

### Check convergence:
```bash
grep "bfgs converged" si_relax.out
```

### Get final structure:
```bash
grep -A 10 "ATOMIC_POSITIONS" si_relax.out | tail -n 6
```

## 🎨 Post-Processing

### Python script to plot optimization:

```python
import re
import matplotlib.pyplot as plt

def extract_energies(filename):
    energies = []
    with open(filename, 'r') as f:
        for line in f:
            if '!    total energy' in line:
                energy = float(line.split()[-2])
                energies.append(energy)
    return energies

def extract_forces(filename):
    forces = []
    with open(filename, 'r') as f:
        for line in f:
            if 'Total force' in line:
                force = float(line.split()[3])
                forces.append(force)
    return forces

# Plot energy convergence
energies = extract_energies('si_relax.out')
plt.figure(figsize=(10, 5))

plt.subplot(1, 2, 1)
plt.plot(energies, 'o-')
plt.xlabel('Ionic Step')
plt.ylabel('Total Energy (Ry)')
plt.title('Energy Convergence')
plt.grid(True)

# Plot force convergence
forces = extract_forces('si_relax.out')
plt.subplot(1, 2, 2)
plt.plot(forces, 'o-')
plt.axhline(y=1e-4, color='r', linestyle='--', label='Threshold')
plt.xlabel('Ionic Step')
plt.ylabel('Total Force (Ry/bohr)')
plt.title('Force Convergence')
plt.yscale('log')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.savefig('optimization.png', dpi=150)
```

## ⚙️ Advanced Options

### Constrained relaxation:

Fix specific atoms by using `ATOMIC_POSITIONS {crystal}` with constraints:

```
ATOMIC_POSITIONS {crystal}
  Si 0.00 0.00 0.00  0 0 0
  Si 0.25 0.25 0.25  1 1 1
```

The `0 0 0` means atom is fixed, `1 1 1` means free to move.

### Optimize only cell volume:

```fortran
&CELL
  cell_dynamics = 'bfgs'
  cell_dofree = 'ibrav'
/
```

### Apply external pressure:

```fortran
&CELL
  cell_dynamics = 'bfgs'
  press = 10.0  ! Apply 10 kbar
/
```

## 🐛 Common Issues

### Issue: Optimization not converging
**Solutions**:
- Start from better initial geometry
- Reduce step size: add `upscale = 10` to &IONS
- Use damped dynamics: `ion_dynamics = 'damp'`
- Check if structure is unstable

### Issue: Forces oscillating
**Solutions**:
- Tighten SCF convergence: `conv_thr = 1.0d-9`
- Increase k-points
- Increase `ecutwfc`

### Issue: Cell optimization failing
**Solutions**:
- Start from reasonable cell parameters
- Use `cell_dynamics = 'damp-pr'` instead of BFGS
- Check pressure convergence threshold

### Issue: Atoms moving out of cell
**Solutions**:
- Use better starting geometry
- Add constraints to problematic atoms
- Check pseudopotentials are appropriate

## 📚 Example: Water Molecule Relaxation

```fortran
&CONTROL
  calculation = 'relax'
  prefix = 'water'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
  forc_conv_thr = 1.0d-4
/

&SYSTEM
  ibrav = 1
  celldm(1) = 20.0
  nat = 3
  ntyp = 2
  ecutwfc = 50.0
  ecutrho = 400.0
/

&ELECTRONS
  conv_thr = 1.0d-8
/

&IONS
  ion_dynamics = 'bfgs'
/

ATOMIC_SPECIES
  O 15.999 O.pbe-n-kjpaw_psl.1.0.0.UPF
  H  1.008 H.pbe-kjpaw_psl.1.0.0.UPF

ATOMIC_POSITIONS (angstrom)
  O  0.00  0.00  0.00
  H  0.96  0.00  0.00
  H -0.24  0.93  0.00

K_POINTS (gamma)
```

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- `relax` for atomic positions, `vc-relax` for atoms + cell
- Monitor forces and energy convergence
- Use BFGS for efficient optimization
- Always verify final structure makes physical sense
