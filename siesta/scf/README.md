# SCF Calculations with SIESTA

Self-Consistent Field (SCF) calculations in SIESTA solve the Kohn-Sham equations using atomic orbital basis sets to obtain the ground state density matrix and total energy.

## 📖 Theory

SIESTA uses:
- **LCAO basis** - Linear Combination of Atomic Orbitals
- **Numerical atomic orbitals** - Strictly confined to finite range
- **Density matrix formulation** - Works with density matrix, not wavefunctions

SCF procedure:
1. Initialize density matrix
2. Compute Hamiltonian matrix
3. Diagonalize to get new density matrix
4. Mix old and new density matrices
5. Check convergence
6. Repeat until converged

## 🎯 What You'll Learn

- Create SIESTA input files (.fdf)
- Choose appropriate basis sets
- Set mesh and SCF parameters
- Read and interpret output

## 📝 Example: Silicon Crystal

### Input File (si_scf.fdf)

```
# Silicon SCF calculation with SIESTA
#
# System description
#

SystemName          Silicon bulk crystal
SystemLabel         si_scf

# Number of atoms and species
NumberOfAtoms       2
NumberOfSpecies     1

# Chemical species
%block ChemicalSpeciesLabel
  1  14  Si
%endblock ChemicalSpeciesLabel

# Lattice structure (FCC)
LatticeConstant     5.43 Ang

%block LatticeVectors
  0.000  0.500  0.500
  0.500  0.000  0.500
  0.500  0.500  0.000
%endblock LatticeVectors

# Atomic coordinates
AtomicCoordinatesFormat  Fractional

%block AtomicCoordinatesAndAtomicSpecies
  0.00  0.00  0.00  1  Si
  0.25  0.25  0.25  1  Si
%endblock AtomicCoordinatesAndAtomicSpecies

#
# Basis set definition
#

PAO.BasisType       split
PAO.BasisSize       DZP
PAO.EnergyShift     0.01 Ry

# Real-space mesh
MeshCutoff          300.0 Ry

#
# DFT functional
#

XC.functional       GGA
XC.authors          PBE

#
# k-point sampling
#

%block kgrid_Monkhorst_Pack
  8  0  0  0.0
  0  8  0  0.0
  0  0  8  0.0
%endblock kgrid_Monkhorst_Pack

#
# SCF parameters
#

MaxSCFIterations    50
DM.MixingWeight     0.25
DM.Tolerance        1.0d-4
DM.NumberPulay      5

SolutionMethod      diagon

#
# Output options
#

WriteDM             .true.
WriteCoorStep       .true.
WriteForces         .true.
WriteKpoints        .true.
WriteEigenvalues    .true.
```

### Parameter Explanation

#### System identification:
- `SystemName` - Descriptive name (for documentation)
- `SystemLabel` - Prefix for all output files

#### Chemical species:
```
%block ChemicalSpeciesLabel
  index  atomic_number  label
%endblock ChemicalSpeciesLabel
```

#### Lattice:
- `LatticeConstant` - Scaling factor with units
- `LatticeVectors` - Primitive vectors (scaled by LatticeConstant)

#### Atomic positions:
- `AtomicCoordinatesFormat` - `Fractional`, `Ang`, `Bohr`, `ScaledCartesian`
- Each line: x y z species_index [optional_label]

#### Basis set (CRUCIAL for SIESTA):
- `PAO.BasisType` - `split` (most common), `nodes`, `splitgauss`
- `PAO.BasisSize` - Overall size
  - `SZ` - Single-ζ (minimal, not recommended)
  - `DZ` - Double-ζ
  - `DZP` - Double-ζ + polarization (good default)
  - `TZP` - Triple-ζ + polarization (high accuracy)
- `PAO.EnergyShift` - Confinement energy (Ry)
  - Smaller = softer confinement, larger basis
  - Typical: 0.01-0.05 Ry

#### Mesh:
- `MeshCutoff` - Real-space grid fineness (Ry)
  - Typical: 200-400 Ry
  - Higher = more accurate but slower

#### Exchange-correlation:
- `XC.functional` - `LDA` or `GGA`
- `XC.authors` - `PBE`, `PW92`, `RPBE`, `revPBE`, `BLYP`, etc.

#### K-points (Monkhorst-Pack):
```
%block kgrid_Monkhorst_Pack
  nx  0  0  sx
  0  ny  0  sy
  0  0  nz  sz
%endblock kgrid_Monkhorst_Pack
```
- nx, ny, nz: grid dimensions
- sx, sy, sz: shifts (usually 0.0 or 0.5)

#### SCF convergence:
- `MaxSCFIterations` - Max iterations
- `DM.MixingWeight` - Density matrix mixing (0-1)
  - Smaller = more stable, slower
  - Typical: 0.1-0.3
- `DM.Tolerance` - Convergence criterion
  - Typical: 1.0d-4 (standard), 1.0d-6 (tight)
- `DM.NumberPulay` - Pulay mixing history
- `SolutionMethod` - `diagon` (diagonalization) or `OMM` (for large systems)

## 🚀 Running the Calculation

```bash
# Serial
siesta < si_scf.fdf > si_scf.out

# Parallel (4 processors)
mpirun -np 4 siesta < si_scf.fdf > si_scf.out
```

## 📊 Reading the Output

### Header:
```
Siesta Version:
Architecture  :
Compiler flags:
```

### System information:
```
outcell: Unit cell vectors (Ang):
        0.000000    2.715000    2.715000
        2.715000    0.000000    2.715000
        2.715000    2.715000    0.000000

siesta: System type = bulk
```

### Basis set information:
```
PAO.BasisType: split
PAO.BasisSize: DZP

Species number:   1  Label: Si  Mass:  28.09
  Number of atoms of this species:   2
  
  Atomic orbitals:
    l=0 (s) n=1 ζ=2 Rc=5.12 [Ang]
    l=1 (p) n=2 ζ=2 Rc=6.25 [Ang]
    l=2 (d) n=3 ζ=1 Rc=6.25 [Ang] (polarization)
```

### K-point grid:
```
siesta: k-grid: Number of k-points =   120
```

### SCF cycle:
```
scf:    1    -15.845678     -7.922839  0.1234E+00    3.1234
scf:    2    -15.846234     -0.000556  0.5678E-01    1.5678
scf:    3    -15.846891     -0.000657  0.2345E-01    0.8765
...
scf:   12    -15.847123     -0.000001  0.1234E-04    0.0012

SCF Convergence by DM criterion
max |DM(i,j) - DM_out(i,j)| :     0.0001234000

siesta: E_KS(eV) =             -15.847123
```

Each SCF line:
- Iteration number
- Total energy (eV)
- Energy change
- DM maximum change
- Eigenvalue shift

### Final results:
```
siesta: Final energy (eV):
siesta:  Band Struct. =    -123.456789
siesta:  Kinetic     =     123.456789
siesta:  Hartree     =      45.678901
siesta:  Edftu       =       0.000000
siesta:  Eso         =       0.000000
siesta:  Eldau       =       0.000000
siesta:  DEna        =       1.234567
siesta:  DUscf       =       0.123456
siesta:  DUext       =       0.000000
siesta:  Enegf       =       0.000000
siesta:  Exc         =     -89.012345
siesta:  eta*DQ      =       0.000000
siesta:  Emadel      =       0.000000
siesta:  Emeta       =       0.000000
siesta:  Emolmec     =       0.000000
siesta:  Ekinion     =       0.000000
siesta:  Eharris     =     -15.847123
siesta:  Etot        =     -15.847123
siesta:  FreeEng     =     -15.847123

siesta: Atomic forces (eV/Ang):
siesta:      1    0.000000    0.000000    0.000000
siesta:      2    0.000000    0.000000    0.000000
siesta: ----------------------------------------
siesta:    Tot    0.000000    0.000000    0.000000
```

### Stress tensor:
```
siesta: Stress tensor (static) (eV/Ang**3):
siesta:    -0.000123   0.000000   0.000000
siesta:     0.000000  -0.000123   0.000000
siesta:     0.000000   0.000000  -0.000123
```

## 🔍 Important Checks

### Extract total energy:
```bash
grep "siesta: Etot" si_scf.out
```

### Check convergence:
```bash
grep "SCF Convergence" si_scf.out
```

### Get forces:
```bash
grep -A 10 "Atomic forces" si_scf.out
```

### Check basis size:
```bash
grep "Number of basis orbitals" si_scf.out
```

## 🎨 Post-Processing

### Python script to extract SCF energies:

```python
import re
import matplotlib.pyplot as plt

def parse_siesta_scf(filename):
    """Parse SIESTA output for SCF energies"""
    energies = []
    
    with open(filename, 'r') as f:
        for line in f:
            if line.startswith('scf:'):
                parts = line.split()
                if len(parts) >= 3:
                    try:
                        energy = float(parts[2])
                        energies.append(energy)
                    except:
                        pass
    
    return energies

# Extract and plot
energies = parse_siesta_scf('si_scf.out')

plt.figure(figsize=(8, 5))
plt.plot(energies, 'o-')
plt.xlabel('SCF Iteration')
plt.ylabel('Total Energy (eV)')
plt.title('SCF Convergence - Silicon')
plt.grid(True)
plt.savefig('scf_convergence.png', dpi=150)

if len(energies) > 0:
    print(f"Final energy: {energies[-1]:.6f} eV")
    print(f"Converged in {len(energies)} iterations")
```

## ⚙️ Convergence Testing

### Basis set convergence:

```
# Test different basis sizes
# Run separate calculations with:
PAO.BasisSize  SZ
PAO.BasisSize  DZ
PAO.BasisSize  DZP
PAO.BasisSize  TZP
```

### MeshCutoff convergence:

```
# Test different mesh cutoffs (in separate runs)
MeshCutoff  200.0 Ry
MeshCutoff  250.0 Ry
MeshCutoff  300.0 Ry
MeshCutoff  350.0 Ry
MeshCutoff  400.0 Ry
```

### K-point convergence:

Test different grids: 4×4×4, 6×6×6, 8×8×8, 10×10×10, 12×12×12

## 🐛 Common Issues

### Issue: SCF not converging
**Solutions**:
- Reduce `DM.MixingWeight` (try 0.1)
- Increase `DM.NumberPulay`
- For metals, add `ElectronicTemperature 300 K`
- Check if basis set is reasonable

### Issue: "Mesh too coarse" warning
**Solution**: Increase `MeshCutoff`

### Issue: Forces too large
**Solutions**:
- Structure may not be at equilibrium - run relaxation
- Tighten `DM.Tolerance`
- Increase `MeshCutoff`

### Issue: Basis set too large (memory)
**Solutions**:
- Reduce `PAO.BasisSize` (TZP → DZP → DZ)
- Increase `PAO.EnergyShift` (tighter confinement)
- Use parallelization

### Issue: Inconsistent results
**Solution**: Test basis and mesh convergence!

## 📚 Additional Examples

### Aluminum (metal):

```
SystemName          Aluminum bulk
SystemLabel         al_scf

NumberOfAtoms       1
NumberOfSpecies     1

%block ChemicalSpeciesLabel
  1  13  Al
%endblock ChemicalSpeciesLabel

LatticeConstant     4.05 Ang
%block LatticeVectors
  0.0  0.5  0.5
  0.5  0.0  0.5
  0.5  0.5  0.0
%endblock LatticeVectors

AtomicCoordinatesFormat  Fractional
%block AtomicCoordinatesAndAtomicSpecies
  0.00  0.00  0.00  1
%endblock AtomicCoordinatesAndAtomicSpecies

PAO.BasisSize       DZP
PAO.EnergyShift     0.01 Ry
MeshCutoff          300.0 Ry

XC.functional       GGA
XC.authors          PBE

# Metal-specific: electronic temperature
ElectronicTemperature  300 K

%block kgrid_Monkhorst_Pack
  12  0  0  0.0
  0  12  0  0.0
  0   0 12  0.0
%endblock kgrid_Monkhorst_Pack

MaxSCFIterations    50
DM.MixingWeight     0.1
DM.Tolerance        1.0d-4
DM.NumberPulay      5
```

### Water molecule:

```
SystemName          Water molecule
SystemLabel         h2o_scf

NumberOfAtoms       3
NumberOfSpecies     2

%block ChemicalSpeciesLabel
  1   8  O
  2   1  H
%endblock ChemicalSpeciesLabel

LatticeConstant     1.0 Ang
%block LatticeVectors
  20.0   0.0   0.0
   0.0  20.0   0.0
   0.0   0.0  20.0
%endblock LatticeVectors

AtomicCoordinatesFormat  Ang
%block AtomicCoordinatesAndAtomicSpecies
  0.000   0.000   0.000  1  O
  0.757   0.586   0.000  2  H
 -0.757   0.586   0.000  2  H
%endblock AtomicCoordinatesAndAtomicSpecies

PAO.BasisSize       DZP
MeshCutoff          300.0 Ry

XC.functional       GGA
XC.authors          PBE

%block kgrid_Monkhorst_Pack
  1  0  0  0.0
  0  1  0  0.0
  0  0  1  0.0
%endblock kgrid_Monkhorst_Pack

MaxSCFIterations    100
DM.MixingWeight     0.3
DM.Tolerance        1.0d-4
```

## 🔗 Next Steps

- [Structure Optimization Tutorial](../relax/README.md)
- [Band Structure Tutorial](../bands/README.md)

---

**Key Takeaways:**
- SIESTA uses atomic orbital basis - test basis size!
- `PAO.BasisSize DZP` is good default
- `MeshCutoff` controls grid accuracy
- Always test convergence: basis + mesh + k-points
- For metals, use `ElectronicTemperature`
