# ABINIT Tutorials

[ABINIT](https://www.abinit.org/) is a software suite to calculate the optical, mechanical, vibrational, and other observable properties of materials using DFT. It uses pseudopotentials and a plane-wave basis set.

## 📋 Contents

- [SCF Calculations](./scf/) - Self-consistent field energy calculations
- [Structure Optimization](./relax/) - Geometry relaxation
- [Band Structure](./bands/) - Electronic band structure calculations

## 🔧 Installation

### Using package manager (Ubuntu/Debian):
```bash
sudo apt-get install abinit
```

### From source:
```bash
# Download from https://www.abinit.org/
tar -xzf abinit-X.X.X.tar.gz
cd abinit-X.X.X
./configure
make
make install
```

### Using conda:
```bash
conda install -c conda-forge abinit
```

## 🎯 Main Executables

- `abinit` - Main DFT calculation code
- `anaddb` - Analysis of phonon databases
- `cut3d` - Process density/potential files
- `mrgddb` - Merge derivative databases

## 📁 Input File Structure

ABINIT uses a keyword-based input format with datasets:

```
# Comments start with #

# Global variables
ecut 30.0
nband 8

# Dataset 1: SCF
ndtset 2
jdtset 1 2

# Dataset 1 specific
iscf1 7
toldfe1 1.0d-8

# Dataset 2 specific
iscf2 -2
getden2 1
tolwfr2 1.0d-12

# Structural parameters
ntypat 1
znucl 14
natom 2
typat 1 1
acell 3*10.26
rprim 0.0 0.5 0.5
      0.5 0.0 0.5
      0.5 0.5 0.0
xred 0.0 0.0 0.0
     0.25 0.25 0.25

# K-points
ngkpt 8 8 8
nshiftk 1
shiftk 0.0 0.0 0.0
```

## 🚀 Running Calculations

### Serial execution:
```bash
abinit < input.in > output.log
# or
abinit input.in > output.log
```

### Parallel execution (MPI):
```bash
mpirun -np 4 abinit input.in > output.log
```

### With separate input and output files:
```bash
abinit input.abi > log 2> err
```

## 📊 Output Files

- `*.log` or `*.out` - Main output with results
- `*_DEN` - Electron density file
- `*_WFK` - Wavefunction file
- `*_EIG` - Eigenvalues
- `*_GSR.nc` - Ground state results (NetCDF)

## 🔑 Key Parameters

### Critical convergence parameters:
- `ecut` - Plane-wave kinetic energy cutoff (Hartree)
- `ngkpt` - K-point grid (3 integers)
- `toldfe` - Tolerance on total energy difference (Hartree)
- `tolwfr` - Tolerance on wavefunction residual
- `nstep` - Maximum number of SCF iterations

### Common functional choices:
- `ixc 11` - PBE GGA (most common)
- `ixc 1` - LDA (Perdew-Wang)
- `ixc 7` - LDA (Perdew-Zunger)

### Calculation types:
- `iscf 7` - Standard SCF (Pulay mixing)
- `iscf -2` - Non-SCF calculation (bands)
- `ionmov 2` - BFGS geometry optimization
- `optcell 2` - Full cell optimization

## 📚 Pseudopotentials

ABINIT supports various formats:
- **Norm-conserving** (Troullier-Martins, HGH, FHI)
- **PAW** (JTH, GBRV)

Download from:
- [ABINIT Pseudopotentials](https://www.abinit.org/psp-tables)
- [Pseudo Dojo](http://www.pseudo-dojo.org/)
- [JTH PAW Table](https://www.abinit.org/sites/default/files/PAwaves/JTH-LDA-paw.txt)

File formats:
- `.psp8` - Modern format
- `.xml` - PAW XML format
- `.fhi` - Fritz-Haber Institute format

## 🔗 Useful Links

- [Official Documentation](https://docs.abinit.org/)
- [ABINIT Tutorials](https://docs.abinit.org/tutorial/)
- [Input Variables](https://docs.abinit.org/variables/)

## 💡 Tips

1. Test convergence with respect to `ecut` and `ngkpt`
2. Use datasets to chain calculations (SCF → bands)
3. `prtwf 1` to save wavefunctions for next step
4. For metals, use `occopt 3` or higher with `tsmear`
5. Check total energy and forces in output

## 📐 Unit Conversions

ABINIT uses atomic units (Hartree):
- 1 Hartree = 27.211 eV
- 1 Bohr = 0.529177 Angstrom
- Energy cutoff often given in Hartree
- K-point grids use `ngkpt` instead of density

## 🎨 Visualization

ABINIT produces data files that can be visualized with:
- **V_Sim** - Structure and density visualization
- **XCrySDen** - Crystal structure and Fermi surface
- **AbiPy** - Python framework for ABINIT
- **Cut3D** - Built-in tool to convert files

---

Navigate to specific tutorials:
- [SCF Tutorial](./scf/README.md)
- [Relaxation Tutorial](./relax/README.md)
- [Band Structure Tutorial](./bands/README.md)
