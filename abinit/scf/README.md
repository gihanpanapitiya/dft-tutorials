# SCF Calculations with ABINIT

Self-Consistent Field (SCF) calculations in ABINIT find the ground state electron density and total energy through iterative solution of the Kohn-Sham equations.

## 📖 Theory

The SCF procedure:
1. Initialize with guess density
2. Construct Kohn-Sham Hamiltonian
3. Solve for wavefunctions
4. Compute new density
5. Mix old and new densities
6. Check convergence
7. Repeat until converged

## 🎯 What You'll Learn

- Write ABINIT input files for SCF
- Understand key ABINIT variables
- Interpret output files
- Extract energies and properties

## 📝 Example: Silicon Crystal

### Input File (si_scf.abi)

```
# Silicon SCF calculation
# Ground state calculation using ABINIT

#Definition of the unit cell
#**************************
acell 3*10.26         # Lattice constant in Bohr
rprim 0.0 0.5 0.5     # FCC primitive vectors
      0.5 0.0 0.5
      0.5 0.5 0.0

#Definition of the atom types and atoms
#***************************************
ntypat 1              # Number of types of atoms
znucl 14              # Atomic number of species
natom 2               # Number of atoms
typat 1 1             # Type of each atom
xred                  # Reduced coordinates
   0.0  0.0  0.0
   0.25 0.25 0.25

#Definition of the plane wave basis set
#***************************************
ecut 30.0             # Plane wave cutoff energy in Hartree

#Definition of the k-point grid
#*******************************
ngkpt 8 8 8           # Grid of k-points
nshiftk 1             # Number of shifts
shiftk 0.0 0.0 0.0    # Shift vector

#Definition of the SCF procedure
#********************************
nstep 50              # Maximum number of SCF iterations
toldfe 1.0d-10        # Tolerance on total energy (Hartree)
diemac 12.0           # Dielectric constant for preconditioning

#Dataset and printing options
#*****************************
prtwf 1               # Print wavefunctions
prtden 1              # Print density
prteig 1              # Print eigenvalues

#Pseudopotential
#***************
pp_dirpath "$ABINIT_PP_PATH"
pseudos "Si.psp8"
```

### Parameter Explanation

#### Structural parameters:
- `acell` - Lattice parameters (Bohr). Use `3*value` for cubic
- `rprim` - Primitive vectors (dimensionless, scaled by acell)
- `ntypat` - Number of atom types
- `znucl` - Nuclear charge (atomic number) for each type
- `natom` - Total number of atoms
- `typat` - Type assignment for each atom
- `xred` - Reduced (fractional) coordinates

#### Basis set:
- `ecut` - Kinetic energy cutoff in Hartree
  - Typical values: 20-50 Ha depending on pseudopotential
  - Test convergence!

#### K-points:
- `ngkpt` - Monkhorst-Pack grid dimensions
- `nshiftk` - Number of grid shifts
- `shiftk` - Shift vectors (for better sampling)

#### SCF parameters:
- `nstep` - Max SCF iterations (default 30)
- `toldfe` - Convergence on total energy difference (Hartree)
  - Alternative: `toldff` (forces), `tolwfr` (wavefunction residual)
- `diemac` - Model dielectric constant (helps convergence)
  - Metals: use `diemac 1e6` or large value

#### Output control:
- `prtwf 1` - Write wavefunction file
- `prtden 1` - Write density file
- `prteig 1` - Print eigenvalues

## 🚀 Running the Calculation

```bash
# Create a files file (si.files)
cat > si.files << EOF
si_scf.abi
si_scf.log
si_scfi
si_scfo
si_scf
/path/to/pseudopotentials/
EOF

# Run ABINIT
abinit < si.files > si_scf.log
# or simply
abinit si_scf.abi > si_scf.log
```

Alternative modern approach:
```bash
# ABINIT can auto-generate file names
abinit si_scf.abi
```

## 📊 Reading the Output

### Header Information

```
.Version 9.x.x of ABINIT
  
 === Build Information ===
...
```

### Input Echo

```
-outvars: echo values of preprocessed input variables --------
 
         acell      1.0260000000E+01  1.0260000000E+01  1.0260000000E+01 Bohr
          ecut      3.00000000E+01 Hartree
```

### SCF Cycle

```
 ITER STEP NUMBER     1
 vtorho : nnsclo_now=  2, note that nnsclo,dbl_nnsclo,istep=  0 0  1
 Total charge density [el/Bohr^3]
      Maximum=    8.2914E-02  at reduced coord.    0.0000    0.0000    0.0000
      Minimum=    3.1416E-03  at reduced coord.    0.5000    0.5000    0.5000
   Integrated=    8.0000E+00

ETOT  1  -8.7140531876323    -8.714E+00 1.234E-02 5.678E+01
```

Each iteration shows:
- Iteration number
- Total energy (ETOT)
- Energy change
- Density residual
- Potential residual

### Convergence

```
 At SCF step   12, etot is converged : 
  for the second time, diff in etot=  1.234E-11 < toldfe=  1.000E-10
  
 Cartesian components of stress tensor (hartree/bohr^3)
  sigma(1 1)=  1.23456789E-05  sigma(3 2)=  0.00000000E+00
  sigma(2 2)=  1.23456789E-05  sigma(3 1)=  0.00000000E+00
  sigma(3 3)=  1.23456789E-05  sigma(2 1)=  0.00000000E+00
```

### Final Results

```
 Components of total free energy (in Hartree) :
 
    Kinetic energy  =  3.12345678901234E+00
    Hartree energy  =  5.67890123456789E-01
    XC energy       = -3.45678901234567E+00
    Ewald energy    = -8.76543210987654E+00
    PspCore energy  =  1.23456789012345E-01
    Loc. psp. energy= -2.34567890123456E+00
    NL   psp  energy=  1.11111111111111E+00
    >>>>>>>>> Etotal= -7.98765432109876E+00
```

### Forces and Stresses

```
 cartesian forces (hartree/bohr) at end:
    1      0.00000000000000     0.00000000000000     0.00000000000000
    2      0.00000000000000     0.00000000000000     0.00000000000000
 frms,max,avg= 0.0000000E+00 0.0000000E+00   0.000E+00  0.000E+00  0.000E+00 h/b
```

## 🔍 Important Information to Extract

### Total energy:
```bash
grep "Etotal" si_scf.log | tail -1
```

### Eigenvalues:
```bash
grep -A 10 "Eigenvalues" si_scf.log
```

### Forces:
```bash
grep -A 5 "cartesian forces" si_scf.log
```

### Stress:
```bash
grep -A 5 "stress tensor" si_scf.log
```

## 🎨 Post-Processing

### Python script to extract energies:

```python
import re
import matplotlib.pyplot as plt

def parse_abinit_scf(filename):
    """Parse ABINIT output file"""
    energies = []
    
    with open(filename, 'r') as f:
        for line in f:
            if line.startswith(' ETOT'):
                parts = line.split()
                if len(parts) >= 3:
                    energy = float(parts[2])
                    energies.append(energy)
    
    return energies

# Extract energies
energies = parse_abinit_scf('si_scf.log')

# Plot convergence
plt.figure(figsize=(8, 5))
plt.plot(energies, 'o-')
plt.xlabel('SCF Iteration')
plt.ylabel('Total Energy (Ha)')
plt.title('SCF Convergence')
plt.grid(True)
plt.savefig('scf_convergence.png', dpi=150)

print(f"Final energy: {energies[-1]:.10f} Ha")
print(f"Converged in {len(energies)} iterations")
```

## ⚙️ Convergence Testing

### K-point convergence test:

```
# File: kpt_convergence.abi

ndtset 5

# Vary k-point grid
ngkpt1 4 4 4
ngkpt2 6 6 6
ngkpt3 8 8 8
ngkpt4 10 10 10
ngkpt5 12 12 12

# Common parameters
acell 3*10.26
rprim 0.0 0.5 0.5
      0.5 0.0 0.5
      0.5 0.5 0.0
ntypat 1
znucl 14
natom 2
typat 1 1
xred 0.0 0.0 0.0
     0.25 0.25 0.25
ecut 30.0
nshiftk 1
shiftk 0.0 0.0 0.0
toldfe 1.0d-10
```

### Energy cutoff convergence:

```
ndtset 5

# Vary energy cutoff
ecut1 20.0
ecut2 25.0
ecut3 30.0
ecut4 35.0
ecut5 40.0

# Common parameters (same as above)
...
```

## 🐛 Common Issues

### Issue: SCF not converging
**Solutions**:
- Increase `diemac` (try 12 for semiconductors, 1e6 for metals)
- Reduce mixing: add `npulayit 7` (Pulay iterations)
- For metals: use `occopt 3` and `tsmear 0.01`
- Increase `nstep`

### Issue: Negative eigenvalues
**Solution**: Increase `ecut`

### Issue: Memory errors
**Solutions**:
- Reduce `ecut` for testing
- Use coarser k-point grid
- Enable memory distribution: `paral_kgb 1`

### Issue: Wrong pseudopotential format
**Solution**: Check pseudopotential file extension matches format
- `.psp8` for modern format
- Use `pp_dirpath` and `pseudos` correctly

## 📚 Additional Examples

### Aluminum (metal with smearing):

```
# al_scf.abi
acell 3*7.65
rprim 0.0 0.5 0.5
      0.5 0.0 0.5
      0.5 0.5 0.0
ntypat 1
znucl 13
natom 1
typat 1
xred 0.0 0.0 0.0

ecut 35.0
ngkpt 12 12 12
nshiftk 1
shiftk 0.0 0.0 0.0

# Metal-specific
occopt 3              # Fermi-Dirac smearing
tsmear 0.01           # Smearing width in Hartree

nstep 50
toldfe 1.0d-10
diemac 1.0e6          # Large for metals

pp_dirpath "$ABINIT_PP_PATH"
pseudos "Al.psp8"
```

### Water molecule:

```
# h2o_scf.abi
acell 3*20.0          # Large box for molecule
rprim 1.0 0.0 0.0
      0.0 1.0 0.0
      0.0 0.0 1.0

ntypat 2
znucl 8 1
natom 3
typat 1 2 2
xcart                 # Cartesian coordinates in Bohr
  0.0  0.0  0.0
  1.8  1.4  0.0
 -1.8  1.4  0.0

ecut 40.0
ngkpt 1 1 1           # Gamma point only for molecule
nshiftk 1
shiftk 0.0 0.0 0.0

nstep 50
toldfe 1.0d-10

pp_dirpath "$ABINIT_PP_PATH"
pseudos "O.psp8, H.psp8"
```

## 🔗 Next Steps

- [Structure Optimization Tutorial](../relax/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- ABINIT uses keyword-based input (more flexible than namelists)
- Test `ecut` and `ngkpt` convergence
- Use appropriate tolerance (`toldfe` for energy)
- Metals need smearing (`occopt`, `tsmear`)
- Check forces and stress in output
