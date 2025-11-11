# SIESTA Tutorials

[SIESTA](https://siesta-project.org/) (Spanish Initiative for Electronic Simulations with Thousands of Atoms) is a DFT code that uses linear combinations of atomic orbitals (LCAO) as basis sets, making it efficient for large systems.

## 📋 Contents

- [SCF Calculations](./scf/) - Self-consistent field energy calculations
- [Structure Optimization](./relax/) - Geometry relaxation
- [Band Structure](./bands/) - Electronic band structure calculations

## 🔧 Installation

### Using package manager (Ubuntu/Debian):
```bash
sudo apt-get install siesta
```

### From source:
```bash
# Download from https://siesta-project.org/
tar -xzf siesta-X.X.tar.gz
cd siesta-X.X/Obj
sh ../Src/obj_setup.sh
# Edit arch.make for your system
make
```

### Using conda:
```bash
conda install -c conda-forge siesta
```

## 🎯 Main Executables

- `siesta` - Main DFT code
- `denchar` - Density and wavefunction analysis
- `vibra` - Vibrational analysis
- `mprop` - Macroscopic polarization
- `Util/` - Various utility programs

## 📁 Input File Structure

SIESTA uses a flexible free-format input:

```
SystemName         silicon
SystemLabel        si

NumberOfAtoms      2
NumberOfSpecies    1

%block ChemicalSpeciesLabel
  1  14  Si
%endblock ChemicalSpeciesLabel

AtomicCoordinatesFormat  Fractional
%block AtomicCoordinatesAndAtomicSpecies
  0.00  0.00  0.00  1
  0.25  0.25  0.25  1
%endblock AtomicCoordinatesAndAtomicSpecies

LatticeConstant  5.43 Ang
%block LatticeVectors
  0.0  0.5  0.5
  0.5  0.0  0.5
  0.5  0.5  0.0
%endblock LatticeVectors

PAO.BasisSize     DZP
MeshCutoff        300.0 Ry

XC.functional     GGA
XC.authors        PBE

%block kgrid_Monkhorst_Pack
  8  0  0  0.0
  0  8  0  0.0
  0  0  8  0.0
%endblock kgrid_Monkhorst_Pack

DM.Tolerance      1.0d-4
MaxSCFIterations  50
```

## 🚀 Running Calculations

### Serial execution:
```bash
siesta < input.fdf > output.out
# or
siesta input.fdf > output.out
```

### Parallel execution (MPI):
```bash
mpirun -np 4 siesta < input.fdf > output.out
```

## 📊 Output Files

- `*.out` - Main output file
- `SystemLabel.DM` - Density matrix
- `SystemLabel.XV` - Final positions and velocities
- `SystemLabel.EIG` - Eigenvalues
- `SystemLabel.bands` - Band structure data
- `SystemLabel.RHO` - Charge density

## 🔑 Key Parameters

### System definition:
- `SystemName` - Descriptive name
- `SystemLabel` - Prefix for output files
- `NumberOfAtoms` - Total atoms
- `NumberOfSpecies` - Number of species

### Basis set (unique to SIESTA):
- `PAO.BasisSize` - Basis size
  - `SZ` - Single-ζ (minimal)
  - `DZ` - Double-ζ
  - `DZP` - Double-ζ + polarization (recommended)
  - `TZP` - Triple-ζ + polarization
- `PAO.EnergyShift` - Energy shift for basis confinement (default 0.01 Ry)
- `MeshCutoff` - Real space mesh cutoff (Ry)

### Exchange-correlation:
- `XC.functional` - `LDA` or `GGA`
- `XC.authors` - `PBE`, `PW92`, `BLYP`, etc.

### SCF parameters:
- `DM.Tolerance` - Density matrix tolerance
- `DM.MixingWeight` - Mixing weight (0-1)
- `MaxSCFIterations` - Max iterations
- `ElectronicTemperature` - For metals (eV or K)

### Optimization:
- `MD.TypeOfRun` - `CG`, `Broyden`, `FIRE` (optimization)
- `MD.NumCGsteps` - Max optimization steps
- `MD.MaxForceTol` - Force tolerance (eV/Ang)
- `MD.VariableCell` - `.true.` for cell optimization

## 📚 Pseudopotentials

SIESTA uses its own pseudopotential format:
- `.psf` - SIESTA pseudopotential format
- Generated with ATOM program (part of SIESTA)

Download from:
- [SIESTA Pseudopotential Database](https://siesta-project.org/databases/pseudopotentials/)
- [PseudoDojo SIESTA format](http://www.pseudo-dojo.org/)

Or specify generation parameters in input:

```
%block PS.lmax
  Si 2
%endblock PS.lmax

%block PS.KBprojectors
  Si DZP
%endblock PS.KBprojectors
```

## 🔗 Useful Links

- [Official Documentation](https://siesta-project.org/SIESTA_MATERIAL/Docs/Manuals/)
- [SIESTA Manual](https://siesta-project.org/SIESTA_MATERIAL/Docs/Manuals/siesta.pdf)
- [Tutorials](https://siesta-project.org/SIESTA_MATERIAL/Docs/Tutorials/)

## 💡 Tips

1. **Basis set convergence** is crucial - test DZ vs DZP vs TZP
2. **MeshCutoff** affects accuracy - test 200-400 Ry
3. For metals, use `ElectronicTemperature` (e.g., 300 K)
4. SIESTA is O(N) capable for very large systems
5. Check `PAO.EnergyShift` for hard/soft confinement

## 🎨 Key Differences from Plane-Wave Codes

| Aspect | SIESTA (LCAO) | QE/ABINIT (Plane-Wave) |
|--------|---------------|------------------------|
| Basis | Atomic orbitals | Plane waves |
| Scaling | Linear (O(N) capable) | N log N to N³ |
| Best for | Large systems, molecules | Periodic solids |
| Cutoff | MeshCutoff (grid) | Energy cutoff |
| Convergence | Basis size + MeshCutoff | ecutwfc + k-points |

## 📐 Unit Conventions

SIESTA is flexible with units:
- Lengths: Ang, Bohr
- Energy: eV, Ry, Hartree
- Specify units explicitly

Example:
```
LatticeConstant 10.26 Bohr
MeshCutoff 300.0 Ry
ElectronicTemperature 300 K
```

## 🎨 Visualization

SIESTA output can be visualized with:
- **XCrySDen** - Structure and charge density
- **VESTA** - Crystal structure
- **denchar** - Built-in density analysis
- **gnuplot** - For band structures and DOS

---

Navigate to specific tutorials:
- [SCF Tutorial](./scf/README.md)
- [Relaxation Tutorial](./relax/README.md)
- [Band Structure Tutorial](./bands/README.md)
