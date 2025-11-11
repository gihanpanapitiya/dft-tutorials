# SCF Calculations with Quantum ESPRESSO

Self-Consistent Field (SCF) calculations are the foundation of DFT computations. They solve the Kohn-Sham equations iteratively to find the ground state electron density and total energy.

## 📖 Theory

SCF calculations:
1. Start with an initial guess for electron density
2. Calculate the effective potential from the density
3. Solve the Kohn-Sham equations to get new wavefunctions
4. Compute new density from wavefunctions
5. Repeat until convergence (density/energy change below threshold)

## 🎯 What You'll Learn

- Create basic QE input files for SCF calculations
- Set appropriate convergence parameters
- Understand the main output
- Extract total energy and other properties

## 📝 Example: Silicon Crystal

### Input File (si_scf.in)

```fortran
&CONTROL
  calculation = 'scf'
  prefix = 'silicon'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
  verbosity = 'high'
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

ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25

K_POINTS (automatic)
  8 8 8 0 0 0
```

### Parameter Explanation

#### &CONTROL namelist:
- `calculation = 'scf'` - Perform self-consistent calculation
- `prefix` - Prefix for output files
- `outdir` - Directory for temporary files
- `pseudo_dir` - Directory containing pseudopotentials
- `verbosity` - Output detail level ('low', 'high', etc.)

#### &SYSTEM namelist:
- `ibrav = 2` - FCC Bravais lattice
- `celldm(1) = 10.26` - Lattice parameter in Bohr
- `nat = 2` - Number of atoms
- `ntyp = 1` - Number of atomic types
- `ecutwfc = 30.0` - Wavefunction cutoff energy (Ry)
- `ecutrho = 240.0` - Charge density cutoff (Ry)

#### &ELECTRONS namelist:
- `conv_thr = 1.0d-8` - SCF convergence threshold (Ry)
- `mixing_beta = 0.7` - Mixing factor for self-consistency

#### Cards:
- `ATOMIC_SPECIES` - Element symbol, mass, pseudopotential file
- `ATOMIC_POSITIONS` - Atomic coordinates (in alat units)
- `K_POINTS` - 8×8×8 Monkhorst-Pack mesh with no shift

## 🚀 Running the Calculation

```bash
# Serial
pw.x < si_scf.in > si_scf.out

# Parallel (4 processors)
mpirun -np 4 pw.x < si_scf.in > si_scf.out
```

## 📊 Reading the Output

Key information in the output file:

### 1. Lattice Information
```
bravais-lattice index     =            2
lattice parameter (alat)  =      10.2600  a.u.
```

### 2. SCF Convergence
```
     iteration #  1     ecut=    30.00 Ry     beta= 0.70
     Davidson diagonalization with overlap
     ethr =  1.00E-02,  avg # of iterations =  2.0
     total cpu time spent up to now is        0.1 secs
     total energy              =     -15.82345678 Ry
```

### 3. Final Results
```
!    total energy              =     -15.84523410 Ry
     estimated scf accuracy    <       0.00000001 Ry

     The total energy is the sum of the following terms:
     one-electron contribution =       5.01234567 Ry
     hartree contribution      =       1.01234567 Ry
     xc contribution           =      -4.56789012 Ry
     ewald contribution        =     -17.30203532 Ry
```

### 4. Fermi Energy
```
     the Fermi energy is     6.3325 ev
```

### 5. Forces (if requested)
```
     Forces acting on atoms (cartesian axes, Ry/au):
     atom    1 type  1   force =     0.00000000    0.00000000    0.00000000
```

## 🔍 Important Checks

1. **Convergence**: Ensure "convergence has been achieved"
2. **Total Energy**: Note the final total energy value
3. **Accuracy**: Check `estimated scf accuracy` is below threshold
4. **Timing**: Monitor computation time

## 🎨 Post-Processing

### Extract total energy:
```bash
grep "!    total energy" si_scf.out
```

### Extract Fermi energy:
```bash
grep "Fermi energy" si_scf.out
```

### Check convergence:
```bash
grep "convergence" si_scf.out
```

## ⚙️ Convergence Testing

Always test convergence with respect to:

### 1. K-points
Create inputs with different k-point meshes (4×4×4, 6×6×6, 8×8×8, 10×10×10, 12×12×12) and plot total energy vs. k-points.

### 2. Energy cutoff
Test different `ecutwfc` values (20, 25, 30, 35, 40, 50 Ry) and plot total energy vs. cutoff.

Python script for convergence analysis:
```python
import numpy as np
import matplotlib.pyplot as plt

# Example: k-point convergence data
kpoints = [4, 6, 8, 10, 12]
energies = [-15.843, -15.845, -15.8452, -15.8452, -15.8452]

plt.plot(kpoints, energies, 'o-')
plt.xlabel('K-points (N×N×N)')
plt.ylabel('Total Energy (Ry)')
plt.title('K-point Convergence')
plt.grid(True)
plt.savefig('kpoint_convergence.png')
```

## 🐛 Common Issues

### Issue: SCF not converging
**Solution**: 
- Reduce `mixing_beta` (try 0.3-0.5)
- Increase `ecutwfc`
- Use better initial guess
- For metals, use smearing

### Issue: Negative eigenvalues
**Solution**: Increase `ecutwfc`

### Issue: Too slow
**Solution**: 
- Use coarser k-point mesh for testing
- Lower `ecutwfc` for initial tests
- Use parallelization

## 📚 Additional Examples

See the `examples/` directory for:
- `al_scf.in` - Aluminum (metal with smearing)
- `h2o_scf.in` - Water molecule
- `graphene_scf.in` - 2D graphene sheet

## 🔗 Next Steps

- [Structure Optimization Tutorial](../relax/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- SCF gives you the ground state energy
- Always test convergence parameters
- Check output for convergence achievement
- Forces and stress tell you about equilibrium
