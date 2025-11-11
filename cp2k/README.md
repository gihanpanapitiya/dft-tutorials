# CP2K Tutorials

[CP2K](https://www.cp2k.org/) is a versatile quantum chemistry and solid-state physics software package that can perform atomistic simulations of solid state, liquid, molecular, periodic, material, crystal, and biological systems using DFT.

## 📋 Contents

- [SCF Calculations](./scf/) - Self-consistent field energy calculations
- [Structure Optimization](./relax/) - Geometry relaxation
- [Band Structure](./bands/) - Electronic band structure calculations

## 🔧 Installation

### Using package manager (Ubuntu/Debian):
```bash
sudo apt-get install cp2k
```

### From source:
```bash
# Download from https://www.cp2k.org/
tar -xzf cp2k-X.X.tar.gz
cd cp2k-X.X
cd tools/toolchain
./install_cp2k_toolchain.sh
cd ../../
make -j 4 ARCH=local VERSION="ssmp"
```

### Using conda:
```bash
conda install -c conda-forge cp2k
```

## 🎯 Main Executables

- `cp2k.ssmp` - Serial/OpenMP version
- `cp2k.popt` - MPI parallel version
- `cp2k.psmp` - Hybrid MPI+OpenMP version

## 📁 Input File Structure

CP2K uses hierarchical, section-based input:

```
&GLOBAL
  PROJECT silicon
  RUN_TYPE ENERGY
&END GLOBAL

&FORCE_EVAL
  METHOD Quickstep
  
  &DFT
    BASIS_SET_FILE_NAME  BASIS_MOLOPT
    POTENTIAL_FILE_NAME  GTH_POTENTIALS
    
    &QS
      EPS_DEFAULT 1.0E-10
    &END QS
    
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
      &OUTER_SCF
        EPS_SCF 1.0E-6
        MAX_SCF 10
      &END OUTER_SCF
    &END SCF
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
&END FORCE_EVAL
```

## 🚀 Running Calculations

### Serial/OpenMP:
```bash
cp2k.ssmp -i input.inp -o output.out
```

### MPI:
```bash
mpirun -np 4 cp2k.popt -i input.inp -o output.out
```

### Hybrid MPI+OpenMP:
```bash
export OMP_NUM_THREADS=2
mpirun -np 4 cp2k.psmp -i input.inp -o output.out
```

## 📊 Output Files

- `*.out` - Main output file
- `*-RESTART.wfn` - Restart wavefunction
- `*-1.cube` - Electron density (if requested)
- `*.ener` - Energy file
- `*.xyz` - Trajectory file

## 🔑 Key Parameters

### Run control:
- `RUN_TYPE` - `ENERGY`, `GEO_OPT`, `CELL_OPT`, `MD`, `BAND`

### Method:
- `METHOD` - Usually `Quickstep` (DFT)

### DFT parameters:
- `CUTOFF` - Plane-wave cutoff (Ry)
- `REL_CUTOFF` - Relative cutoff for Gaussian (Ry)
- `EPS_SCF` - SCF convergence threshold

### Basis sets and pseudopotentials:
- `BASIS_SET` - Basis set for each atom type
  - `DZVP-MOLOPT-GTH` - Double-ζ + polarization
  - `TZVP-MOLOPT-GTH` - Triple-ζ + polarization
  - `TZV2P-MOLOPT-GTH` - Triple-ζ + 2 polarization
- `POTENTIAL` - Pseudopotential
  - `GTH-PBE-qN` - Goedecker-Teter-Hutter

### XC functional:
- Common: `PBE`, `BLYP`, `PADE` (LDA)

## 📚 Basis Sets and Pseudopotentials

CP2K uses:
- **MOLOPT basis sets** - Optimized for molecules and solids
- **GTH pseudopotentials** - Goedecker-Teter-Hutter

Files included with CP2K:
- `BASIS_MOLOPT` - MOLOPT basis sets
- `GTH_POTENTIALS` - GTH pseudopotentials
- `BASIS_SET` - Various other basis sets

## 🔗 Useful Links

- [Official Documentation](https://manual.cp2k.org/)
- [Input Reference](https://manual.cp2k.org/trunk/CP2K_INPUT.html)
- [Tutorials](https://www.cp2k.org/howto)

## 💡 Tips

1. CP2K uses GPW (Gaussian and Plane Waves) method by default
2. Test both `CUTOFF` and `REL_CUTOFF` for convergence
3. Use `RESTART` files to save computation time
4. CP2K excels at molecular dynamics and large systems
5. Mixed Gaussian/plane-wave basis is efficient

## 🎨 Key Features

- **GPW method** - Combines Gaussians (orbitals) + plane waves (density)
- **GAPW method** - All-electron frozen-core approximation
- **Linear scaling** - For very large systems
- **Extensive XC functionals** - Including hybrid functionals
- **Born-Oppenheimer MD** - Efficient molecular dynamics

## 📐 Unit Conventions

CP2K defaults:
- Energy: Hartree
- Length: Angstrom or Bohr (specify in input)
- Can specify units explicitly

Example:
```
&CELL
  ABC [angstrom] 5.43 5.43 5.43
&END CELL

&MGRID
  CUTOFF [Ry] 400
&END MGRID
```

---

Navigate to specific tutorials:
- [SCF Tutorial](./scf/README.md)
- [Relaxation Tutorial](./relax/README.md)
- [Band Structure Tutorial](./bands/README.md)
