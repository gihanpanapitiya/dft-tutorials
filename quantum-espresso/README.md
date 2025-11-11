# Quantum ESPRESSO Tutorials

[Quantum ESPRESSO](https://www.quantum-espresso.org/) (QE) is an integrated suite of open-source computer codes for electronic-structure calculations and materials modeling at the nanoscale, based on density-functional theory, plane waves, and pseudopotentials.

## 📋 Contents

- [SCF Calculations](./scf/) - Self-consistent field energy calculations
- [Structure Optimization](./relax/) - Geometry relaxation
- [Band Structure](./bands/) - Electronic band structure calculations

## 🔧 Installation

### Using package manager (Ubuntu/Debian):
```bash
sudo apt-get install quantum-espresso
```

### From source:
```bash
# Download from https://www.quantum-espresso.org/
tar -xzf qe-X.X.tar.gz
cd qe-X.X
./configure
make all
```

### Using conda:
```bash
conda install -c conda-forge qe
```

## 🎯 Main Executables

- `pw.x` - Main code for SCF, relaxation, MD
- `ph.x` - Phonon calculations
- `bands.x` - Band structure post-processing
- `dos.x` - Density of states
- `projwfc.x` - Projected DOS and band structure

## 📁 Input File Structure

QE uses a namelist-based input format:

```fortran
&CONTROL
  ...
/

&SYSTEM
  ...
/

&ELECTRONS
  ...
/

&IONS      ! Only for relaxation/MD
  ...
/

&CELL      ! Only for cell optimization
  ...
/

ATOMIC_SPECIES
  ...

ATOMIC_POSITIONS { crystal | alat | bohr | angstrom }
  ...

K_POINTS { automatic | gamma | tpiba | crystal | ... }
  ...

CELL_PARAMETERS { alat | bohr | angstrom }
  ...
```

## 🚀 Running Calculations

### Serial execution:
```bash
pw.x < input.in > output.out
```

### Parallel execution (MPI):
```bash
mpirun -np 4 pw.x -nk 2 < input.in > output.out
```

where `-nk` specifies k-point parallelization.

## 📊 Output Files

- `*.out` - Main output file with results
- `*.save/` - Directory containing wavefunction data
- `*.xml` - XML data file
- `pwscf.wfc*` - Wavefunction files

## 🔑 Key Parameters

### Critical convergence parameters:
- `ecutwfc` - Kinetic energy cutoff for wavefunctions (Ry)
- `ecutrho` - Kinetic energy cutoff for charge density (Ry)
- `conv_thr` - Convergence threshold for SCF
- K-points mesh - Specified in K_POINTS card

### Common functional choices:
- `'PBE'` - GGA functional (most common)
- `'LDA'` - Local density approximation
- `'PBEsol'` - PBE for solids

## 📚 Pseudopotentials

QE uses various pseudopotential formats:
- **USPP** - Ultrasoft pseudopotentials
- **PAW** - Projector augmented wave
- **NC** - Norm-conserving

Download from:
- [Quantum ESPRESSO Pseudopotentials](https://www.quantum-espresso.org/pseudopotentials/)
- [SSSP Library](https://www.materialscloud.org/discover/sssp/)

## 🔗 Useful Links

- [Official Documentation](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE User Guide](https://www.quantum-espresso.org/resources/users-manual)
- [Tutorials](https://www.quantum-espresso.org/resources/tutorials)

## 💡 Tips

1. Always test convergence with respect to `ecutwfc` and k-points
2. For metals, use `occupations='smearing'` with appropriate `smearing` and `degauss`
3. Check the total force and stress in the output
4. Use `tstress=.true.` and `tprnfor=.true.` to print stress and forces

---

Navigate to specific tutorials:
- [SCF Tutorial](./scf/README.md)
- [Relaxation Tutorial](./relax/README.md)
- [Band Structure Tutorial](./bands/README.md)
