# Band Structure Calculations with SIESTA

SIESTA band structure calculations follow a two-step process similar to other codes.

## 📝 Workflow

### Step 1: SCF Calculation (si_scf.fdf)

Run standard SCF with uniform k-mesh (see SCF tutorial).

### Step 2: Band Structure Input (si_bands.fdf)

```
SystemName          Silicon bands
SystemLabel         si_bands

NumberOfAtoms       2
NumberOfSpecies     1

%block ChemicalSpeciesLabel
  1  14  Si
%endblock ChemicalSpeciesLabel

LatticeConstant     5.43 Ang

%block LatticeVectors
  0.000  0.500  0.500
  0.500  0.000  0.500
  0.500  0.500  0.000
%endblock LatticeVectors

AtomicCoordinatesFormat  Fractional
%block AtomicCoordinatesAndAtomicSpecies
  0.00  0.00  0.00  1
  0.25  0.25  0.25  1
%endblock AtomicCoordinatesAndAtomicSpecies

PAO.BasisSize       DZP
MeshCutoff          300.0 Ry

XC.functional       GGA
XC.authors          PBE

# Read density from SCF
DM.UseSaveDM        .true.
DM.NumberPulay      0                  # No mixing for bands

# Band structure k-path
BandLinesScale      pi/a
%block BandLines
  1  0.000  0.000  0.000  \Gamma       # Gamma
 40  2.000  0.000  0.000  X            # X
 40  2.000  1.000  0.000  W            # W
 40  1.500  1.500  0.000  K            # K
 40  0.000  0.000  0.000  \Gamma       # Gamma
 40  1.500  1.500  1.500  L            # L
%endblock BandLines

COOP.Write          .true.             # Write for analysis

WriteEigenvalues    .true.
WriteBands          .true.
```

## 🎯 Key Parameters

- `DM.UseSaveDM .true.` - Read converged density matrix
- `BandLinesScale` - Units for k-points (`pi/a` or `ReciprocalLatticeVectors`)
- `BandLines` block - High-symmetry path
  - Format: npoints kx ky kz label

## 🚀 Running

```bash
# Step 1: SCF
siesta < si_scf.fdf > si_scf.out

# Step 2: Bands
siesta < si_bands.fdf > si_bands.out
```

## 📊 Output

Creates `si_bands.bands` file with format:
```
# k-point(1/Ang)  Band1(eV)  Band2(eV)  ...
```

## 🎨 Plotting

Use `gnuband` (SIESTA utility) or Python:

```python
import numpy as np
import matplotlib.pyplot as plt

data = np.loadtxt('si_bands.bands')
kpoints = data[:, 0]
nbands = data.shape[1] - 1

plt.figure(figsize=(8, 6))
for i in range(1, nbands+1):
    plt.plot(kpoints, data[:, i], 'b-', linewidth=0.5)

plt.ylabel('Energy (eV)')
plt.xlabel('k-path')
plt.title('Silicon Band Structure')
plt.axhline(y=0, color='r', linestyle='--', alpha=0.5)
plt.savefig('si_bands.png', dpi=300)
```

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Relaxation Tutorial](../relax/README.md)
