# Band Structure Calculations with ABINIT

Electronic band structure calculations in ABINIT use a multi-dataset approach: first an SCF calculation on a uniform grid, then a non-SCF calculation along high-symmetry k-paths.

## 📖 Theory

Band structure workflow:
1. **SCF dataset** - Converge ground state on uniform k-mesh
2. **Non-SCF dataset** - Compute eigenvalues along k-path
3. Post-processing for plotting

## 🎯 What You'll Learn

- Use multiple datasets in ABINIT
- Define high-symmetry k-paths
- Extract and plot band structures
- Analyze band gaps

## 📝 Complete Example: Silicon Bands

### Input File (si_bands.abi)

```
# Silicon band structure calculation
# Multi-dataset approach: SCF + bands

ndtset 2              # Two datasets

#DATASET 1: Self-consistent calculation
#****************************************
toldfe1 1.0d-10       # Energy tolerance for dataset 1
prtden1 1             # Print density for dataset 2
prtwf1 1              # Print wavefunctions

#K-points for SCF (uniform grid)
kptopt1 1             # Automatic k-point generation
ngkpt1 8 8 8          # Monkhorst-Pack grid
nshiftk1 1
shiftk1 0.0 0.0 0.0

#DATASET 2: Band structure calculation
#***************************************
iscf2 -2              # Non-self-consistent calculation
getden2 1             # Get density from dataset 1
tolwfr2 1.0d-12       # Wavefunction tolerance
nband2 8              # Number of bands to compute

#K-points for band structure (high-symmetry path)
kptopt2 -6            # Band structure mode (FCC)
ndivsm2 10            # Number of divisions for smallest segment
ndivk2 40 40 40 40 40 # Points between high-symmetry points (optional)

# Alternative: manual k-path specification
# kptopt2 0
# nkpt2 X              # Total number of k-points
# kpt2                 # List k-points explicitly
#   ...

#Common parameters for all datasets
#***********************************

#Unit cell definition
acell 3*10.26
rprim 0.0 0.5 0.5
      0.5 0.0 0.5
      0.5 0.5 0.0

#Atomic species and positions
ntypat 1
znucl 14
natom 2
typat 1 1
xred
   0.0  0.0  0.0
   0.25 0.25 0.25

#Plane wave basis
ecut 30.0

#SCF parameters
nstep 50
diemac 12.0

#Pseudopotential
pp_dirpath "$ABINIT_PP_PATH"
pseudos "Si.psp8"
```

## 📊 Key Parameters for Band Structure

### Dataset control:
- `ndtset` - Number of datasets
- `jdtset` - (optional) Specify which datasets to run

### SCF dataset (dataset 1):
- `toldfe1` - Energy convergence
- `kptopt1 1` - Automatic k-point grid
- `ngkpt1` - K-point mesh
- `prtden1 1` - Save density
- `prtwf1 1` - Save wavefunctions

### Bands dataset (dataset 2):
- `iscf2 -2` - Non-SCF mode (read density, no update)
- `getden2 1` - Use density from dataset 1
- `tolwfr2` - Wavefunction residual tolerance
- `kptopt2` - K-path generation mode
  - `-1` to `-7`: Automatic paths for different lattices
  - `-6`: FCC lattice
  - `0`: Manual k-point specification
- `ndivsm2` - Number of divisions for path segments

### K-path generation:

ABINIT can automatically generate high-symmetry paths:
- `kptopt -1`: Line from (0,0,0) to (½,½,½)
- `kptopt -2`: Band structure for BCC
- `kptopt -3`: Band structure for FCC (partial)
- `kptopt -4`: Hexagonal lattice
- `kptopt -5`: Tetragonal lattice  
- `kptopt -6`: **FCC recommended path** (Γ-X-W-K-Γ-L-U-W-L-K)
- `kptopt -7`: Line in reciprocal space

## 🚀 Running the Calculation

```bash
abinit si_bands.abi > si_bands.log
```

The calculation will:
1. Run SCF (dataset 1)
2. Save density
3. Run bands calculation (dataset 2)
4. Save eigenvalues

## 📊 Reading the Output

### Dataset 1 completion:

```
================================================================================
 
== END DATASET(S) ==============================================================
 
 
 Calculation completed.
.Delivered   X  WARNINGs and   X  COMMENTs to log file.
```

### Dataset 2 - Band energies:

```
 k-point (reduced coord) :    0.00000   0.00000   0.00000
 band energies (Hartree):
  -0.20123456   0.21234567   0.21234567   0.21234567
   0.31234567   0.31234567   0.31234567   0.34123456
```

### Output files:

- `si_bandso_DS1_DEN` - Density from SCF
- `si_bandso_DS2_EIG` - Eigenvalues along k-path
- `si_bandso_DS2_WFK` - Wavefunctions (if requested)

## 🎨 Plotting Band Structure

### Using ABINIT's band2eps:

Some ABINIT versions include visualization tools. Modern approach uses Python:

### Python plotting script (plot_bands.py):

```python
import numpy as np
import matplotlib.pyplot as plt
from netCDF4 import Dataset

def read_abinit_eig(filename):
    """Read ABINIT eigenvalue file"""
    # This is a simple text parser
    # For NetCDF files, use proper NetCDF reader
    
    kpoints = []
    bands = []
    
    # Parser depends on output format
    # This is a template - adjust based on actual format
    
    return np.array(kpoints), np.array(bands)

def plot_bands(eigenvalues, kpath, high_sym_points):
    """Plot band structure"""
    
    fig, ax = plt.subplots(figsize=(8, 6))
    
    # Plot each band
    nbands = eigenvalues.shape[0]
    for i in range(nbands):
        ax.plot(kpath, eigenvalues[i, :] * 27.211, 'b-', linewidth=0.5)
    
    # Add high-symmetry points
    for label, pos in high_sym_points.items():
        ax.axvline(x=pos, color='k', linewidth=0.5, linestyle='--')
    
    ax.set_xticks(list(high_sym_points.values()))
    ax.set_xticklabels(list(high_sym_points.keys()))
    ax.set_ylabel('Energy (eV)')
    ax.set_xlabel('k-path')
    ax.set_title('Silicon Band Structure')
    ax.axhline(y=0, color='r', linewidth=0.5, linestyle='--', alpha=0.5)
    ax.set_ylim(-6, 16)
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('si_bands.png', dpi=300)
    plt.show()

# For FCC with kptopt -6, high-symmetry points are:
high_sym = {
    'Γ': 0.0,
    'X': 1.0,
    'W': 2.0,
    'K': 3.0,
    'Γ': 4.0,
    'L': 5.0,
    'U': 6.0,
    'W': 7.0,
    'L': 8.0,
    'K': 9.0
}

# Read and plot (implement read_abinit_eig based on your output)
# eigenvalues, kpath = read_abinit_eig('si_bandso_DS2_EIG')
# plot_bands(eigenvalues, kpath, high_sym)
```

### Alternative: Using AbiPy

[AbiPy](https://github.com/abinit/abipy) is a Python library for ABINIT:

```python
from abipy import abilab

# Open the output file
with abilab.abiopen("si_bandso_DS2_EIG.nc") as ebands:
    # Plot band structure
    ebands.plot(title="Silicon Band Structure")
    
    # Get band gap info
    print(f"Band gap: {ebands.fundamental_gaps[0].energy} eV")
    print(f"Direct: {ebands.direct_gaps[0].energy} eV")
```

## 🔍 Analysis

### Extract band gap from output:

```bash
# Find VBM and CBM
grep "Fermi" si_bands.log
```

### Manual k-path specification:

For more control, use `kptopt2 0`:

```
#Dataset 2
iscf2 -2
getden2 1
tolwfr2 1.0d-12
kptopt2 0
nkpt2 6               # Number of k-points
ndivk2 30 1 30 30 30  # Points between each pair

kpt2                  # High-symmetry points (crystal coords)
  0.000  0.000  0.000  # Gamma
  0.500  0.000  0.500  # X
  0.625  0.250  0.625  # U
  0.375  0.375  0.750  # K
  0.000  0.000  0.000  # Gamma
  0.500  0.500  0.500  # L
```

## ⚙️ Advanced Options

### Include more bands:

```
nband2 16             # Compute more unoccupied bands
```

### Projected band structure:

Use `prtdos` and related keywords, or use `cut3d` and post-processing tools.

### Spin-polarized calculation:

```
nsppol 2              # Spin-polarized
spinat                # Initial spins
  0 0 1               # Spin on each atom
  0 0 -1
```

## 🐛 Common Issues

### Issue: Not enough bands
**Solution**: Increase `nband2`

### Issue: K-path not as expected
**Solutions**:
- Check `kptopt2` value matches your lattice
- Use manual k-path (`kptopt2 0`)
- Verify high-symmetry points

### Issue: Discontinuous bands
**Solution**: Increase `ndivsm2` or `ndivk2`

### Issue: Wrong band gap
**Solution**: 
- DFT (PBE) underestimates gaps (known issue)
- Consider hybrid functionals or GW

### Issue: Dataset errors
**Solution**: 
- Ensure dataset 1 completed successfully
- Check `getden2 1` refers to correct dataset

## 📚 Complete Workflow Script

```bash
#!/bin/bash

# Run band structure calculation
echo "Running ABINIT band structure calculation..."
abinit si_bands.abi > si_bands.log

# Check if successful
if grep -q "Calculation completed" si_bands.log; then
    echo "Calculation completed successfully!"
    
    # Extract important info
    echo ""
    echo "=== Band Structure Results ==="
    grep -A 3 "Eigenvalues" si_bands.log | head -20
    
    echo ""
    echo "Output files:"
    ls -lh si_bandso_DS*
    
    # Plot (requires appropriate Python script)
    # python plot_bands.py
else
    echo "Calculation failed. Check si_bands.log"
    exit 1
fi
```

## 🔗 High-Symmetry Paths

### FCC (kptopt -6):
Γ → X → W → K → Γ → L → U → W → L → K

Coordinates:
- Γ: (0, 0, 0)
- X: (½, 0, ½)
- W: (½, ¼, ¾)
- K: (⅜, ⅜, ¾)
- L: (½, ½, ½)
- U: (⅝, ¼, ⅝)

### BCC (kptopt -2):
Γ → H → N → Γ → P → H

### For other lattices:
Use [SeeK-path](https://www.materialscloud.org/work/tools/seekpath) or consult literature.

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Relaxation Tutorial](../relax/README.md)
- Explore DOS calculations
- Learn about Fat bands with projwfc

---

**Key Takeaways:**
- Use `ndtset 2` for SCF + bands workflow
- Dataset 1: SCF with uniform k-mesh
- Dataset 2: Non-SCF (`iscf -2`) with k-path
- `kptopt -6` for automatic FCC path
- Use AbiPy for easy visualization
- DFT underestimates band gaps
