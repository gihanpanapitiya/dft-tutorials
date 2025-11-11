# SCF Calculations with CP2K

CP2K uses the Gaussian and Plane Waves (GPW) method for efficient DFT calculations, combining Gaussian basis functions for orbitals with plane waves for the electron density.

## 📖 Theory

GPW Method:
- **Orbitals**: Expanded in Gaussian basis (localized)
- **Density**: Represented on plane-wave grid (extended)
- Combines efficiency of both approaches

## 🎯 What You'll Learn

- Write CP2K input files
- Set cutoffs and basis sets
- Run and interpret SCF calculations
- Understand GPW method parameters

## 📝 Example: Silicon Crystal

### Input File (si_scf.inp)

```
&GLOBAL
  PROJECT silicon_scf
  RUN_TYPE ENERGY
  PRINT_LEVEL MEDIUM
&END GLOBAL

&FORCE_EVAL
  METHOD Quickstep
  
  &DFT
    BASIS_SET_FILE_NAME  BASIS_MOLOPT
    POTENTIAL_FILE_NAME  GTH_POTENTIALS
    
    &QS
      EPS_DEFAULT 1.0E-10
      EXTRAPOLATION ASPC
    &END QS
    
    &MGRID
      CUTOFF 400
      REL_CUTOFF 50
      NGRIDS 4
    &END MGRID
    
    &XC
      &XC_FUNCTIONAL PBE
      &END XC_FUNCTIONAL
    &END XC
    
    &SCF
      MAX_SCF 50
      EPS_SCF 1.0E-6
      SCF_GUESS RESTART
      
      &OUTER_SCF
        EPS_SCF 1.0E-6
        MAX_SCF 10
      &END OUTER_SCF
      
      &OT
        MINIMIZER DIIS
        PRECONDITIONER FULL_SINGLE_INVERSE
      &END OT
      
      &PRINT
        &RESTART
          &EACH
            QS_SCF 10
          &END EACH
        &END RESTART
      &END PRINT
    &END SCF
    
    &KPOINTS
      SCHEME MONKHORST-PACK 8 8 8
    &END KPOINTS
    
  &END DFT
  
  &SUBSYS
    &CELL
      ABC 5.43 5.43 5.43
      ALPHA_BETA_GAMMA 60 60 60
    &END CELL
    
    &COORD
      SCALED
      Si  0.00  0.00  0.00
      Si  0.25  0.25  0.25
    &END COORD
    
    &KIND Si
      BASIS_SET DZVP-MOLOPT-SR-GTH
      POTENTIAL GTH-PBE-q4
    &END KIND
    
  &END SUBSYS
  
  &PRINT
    &FORCES ON
    &END FORCES
    &STRESS_TENSOR ON
    &END STRESS_TENSOR
  &END PRINT
  
&END FORCE_EVAL
```

### Parameter Explanation

#### &GLOBAL section:
- `PROJECT` - Base name for output files
- `RUN_TYPE` - Type of calculation
  - `ENERGY` - Single point energy
  - `GEO_OPT` - Geometry optimization
  - `CELL_OPT` - Cell optimization
  - `BAND` - Band structure
- `PRINT_LEVEL` - Output verbosity (`LOW`, `MEDIUM`, `HIGH`)

#### &DFT section:
- `BASIS_SET_FILE_NAME` - Basis set library file
- `POTENTIAL_FILE_NAME` - Pseudopotential library

#### &QS (Quickstep) section:
- `EPS_DEFAULT` - Default epsilon for numerics
- `EXTRAPOLATION` - Wavefunction extrapolation method

#### &MGRID section (crucial):
- `CUTOFF` - Plane-wave cutoff (Ry)
  - Typical: 300-600 Ry
  - Higher = more accurate
- `REL_CUTOFF` - Relative cutoff for Gaussians (Ry)
  - Typical: 40-60 Ry
  - Controls mapping of Gaussians to grid
- `NGRIDS` - Number of multi-grids (default 4)

#### &XC section:
- `XC_FUNCTIONAL` - Exchange-correlation functional
  - `PBE`, `BLYP`, `PADE` (LDA)

#### &SCF section:
- `MAX_SCF` - Maximum SCF iterations
- `EPS_SCF` - SCF convergence threshold
- `SCF_GUESS` - Initial guess (`ATOMIC`, `RESTART`, `RANDOM`)

#### &OT (Orbital Transformation):
- CP2K's default SCF method (more efficient than diagonalization)
- `MINIMIZER` - Minimization algorithm (`DIIS`, `CG`)
- `PRECONDITIONER` - Preconditioning method

#### &KPOINTS:
- `SCHEME MONKHORST-PACK nx ny nz` - K-point grid

#### &CELL:
- `ABC` - Lattice constants (Angstrom)
- `ALPHA_BETA_GAMMA` - Lattice angles (degrees)
- For FCC, use: a=b=c and α=β=γ=60°

#### &COORD:
- `SCALED` - Use fractional coordinates
- Or just list Cartesian coordinates in Angstrom

#### &KIND:
- Define basis set and potential for each element
- `BASIS_SET` - e.g., `DZVP-MOLOPT-SR-GTH`
- `POTENTIAL` - e.g., `GTH-PBE-q4` (4 valence electrons)

## 🚀 Running the Calculation

```bash
# Serial/OpenMP
cp2k.ssmp -i si_scf.inp -o si_scf.out

# MPI
mpirun -np 4 cp2k.popt -i si_scf.inp -o si_scf.out
```

## 📊 Reading the Output

### Header:
```
 CP2K| version string:                    CP2K version 9.x
 CP2K| source code revision number:       git:xxxxx
```

### Input echo and system info:
```
 CELL| Volume [angstrom^3]:                        40.044
 CELL| Vector a [angstrom]:       5.430       0.000       0.000
```

### SCF cycle:
```
  Step     Update method      Time    Convergence         Total energy    Change
  ------------------------------------------------------------------------------
     1 OT DIIS     0.15E+00    0.5     0.01234567      -15.12345678 -1.51E+01
     2 OT DIIS     0.15E+00    0.4     0.00567890      -15.23456789 -1.11E-01
     3 OT DIIS     0.15E+00    0.4     0.00234567      -15.34567890 -1.11E-01
     ...
    12 OT DIIS     0.15E+00    0.4     0.00000123      -15.84712345 -1.23E-07
  
  *** SCF run converged in    12 steps ***
```

### Final energy:
```
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:        -15.847123456789
```

### Forces:
```
 ATOMIC FORCES in [a.u.]
 
 # Atom   Kind   Element          X              Y              Z
      1      1      Si          0.00000000     0.00000000     0.00000000
      2      1      Si          0.00000000     0.00000000     0.00000000
 SUM OF ATOMIC FORCES           0.00000000     0.00000000     0.00000000
```

### Stress:
```
 STRESS| Analytical stress tensor [GPa]
 STRESS|                        x          y          z
 STRESS|           x        0.123456   0.000000   0.000000
 STRESS|           y        0.000000   0.123456   0.000000
 STRESS|           z        0.000000   0.000000   0.123456
```

## 🔍 Important Information

### Extract total energy:
```bash
grep "ENERGY|" si_scf.out | tail -1
```

### Check convergence:
```bash
grep "SCF run converged" si_scf.out
```

### Get forces:
```bash
grep -A 10 "ATOMIC FORCES" si_scf.out
```

## 🎨 Post-Processing

### Python script:

```python
import re
import matplotlib.pyplot as plt

def parse_cp2k_scf(filename):
    """Parse CP2K SCF output"""
    energies = []
    convergence = []
    
    with open(filename, 'r') as f:
        for line in f:
            # Match SCF step lines
            if re.match(r'\s+\d+\s+OT', line):
                parts = line.split()
                conv = float(parts[4])
                energy = float(parts[5])
                convergence.append(conv)
                energies.append(energy)
    
    return energies, convergence

# Parse and plot
energies, conv = parse_cp2k_scf('si_scf.out')

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

ax1.plot(energies, 'o-')
ax1.set_xlabel('SCF Step')
ax1.set_ylabel('Total Energy (Ha)')
ax1.set_title('Energy Convergence')
ax1.grid(True)

ax2.plot(conv, 'o-')
ax2.set_xlabel('SCF Step')
ax2.set_ylabel('Convergence')
ax2.set_yscale('log')
ax2.set_title('SCF Convergence')
ax2.grid(True)

plt.tight_layout()
plt.savefig('cp2k_scf_convergence.png', dpi=150)
```

## ⚙️ Convergence Testing

### Cutoff convergence:

Test `CUTOFF`: 300, 400, 500, 600 Ry

### Relative cutoff convergence:

Test `REL_CUTOFF`: 40, 50, 60, 80 Ry

### K-point convergence:

Test grids: 4×4×4, 6×6×6, 8×8×8, 10×10×10

## 🐛 Common Issues

### Issue: SCF not converging
**Solutions**:
- Switch from OT to diagonalization (remove &OT section)
- Reduce `ADDED_MOS` for better SCF
- Check if system is metallic - use smearing

### Issue: Memory errors
**Solutions**:
- Reduce `CUTOFF`
- Use fewer k-points initially
- Enable `LOW_MEMORY_OPTION .TRUE.` in &QS

### Issue: "Cholesky decompose failed"
**Solution**: 
- Basis set linear dependency - try different basis
- Increase `EPS_DEFAULT`

## 📚 Example: Aluminum (Metal)

```
&GLOBAL
  PROJECT aluminum_scf
  RUN_TYPE ENERGY
&END GLOBAL

&FORCE_EVAL
  METHOD Quickstep
  
  &DFT
    BASIS_SET_FILE_NAME  BASIS_MOLOPT
    POTENTIAL_FILE_NAME  GTH_POTENTIALS
    
    &MGRID
      CUTOFF 400
      REL_CUTOFF 50
    &END MGRID
    
    &XC
      &XC_FUNCTIONAL PBE
      &END XC_FUNCTIONAL
    &END XC
    
    &SCF
      MAX_SCF 50
      EPS_SCF 1.0E-6
      ADDED_MOS 20          # Add empty orbitals for metals
      
      &SMEAR ON             # Smearing for metals
        METHOD FERMI_DIRAC
        ELECTRONIC_TEMPERATURE [K] 300
      &END SMEAR
      
      &DIAGONALIZATION ON   # Use diagonalization for metals
        ALGORITHM STANDARD
      &END DIAGONALIZATION
      
      &MIXING
        METHOD BROYDEN_MIXING
        ALPHA 0.4
        NBROYDEN 8
      &END MIXING
    &END SCF
    
    &KPOINTS
      SCHEME MONKHORST-PACK 12 12 12
    &END KPOINTS
  &END DFT
  
  &SUBSYS
    &CELL
      ABC 4.05 4.05 4.05
      ALPHA_BETA_GAMMA 60 60 60
    &END CELL
    
    &COORD
      SCALED
      Al  0.00  0.00  0.00
    &END COORD
    
    &KIND Al
      BASIS_SET DZVP-MOLOPT-SR-GTH
      POTENTIAL GTH-PBE-q3
    &END KIND
  &END SUBSYS
&END FORCE_EVAL
```

## 🔗 Next Steps

- [Structure Optimization Tutorial](../relax/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- CP2K uses GPW method (Gaussians + plane waves)
- Test `CUTOFF` and `REL_CUTOFF` convergence
- OT method is efficient for insulators/semiconductors
- Metals need diagonalization + smearing
- Choose appropriate basis set (DZP, TZP)
