# Band Structure Calculations with CP2K

CP2K band structure calculations require an SCF calculation followed by a band structure run along a k-path.

## 📝 Workflow

### Step 1: SCF Calculation

Run standard SCF (see SCF tutorial) to get converged wavefunction.

### Step 2: Band Structure Input (si_bands.inp)

```
&GLOBAL
  PROJECT silicon_bands
  RUN_TYPE ENERGY
  PRINT_LEVEL MEDIUM
&END GLOBAL

&FORCE_EVAL
  METHOD Quickstep
  
  &DFT
    BASIS_SET_FILE_NAME  BASIS_MOLOPT
    POTENTIAL_FILE_NAME  GTH_POTENTIALS
    WFN_RESTART_FILE_NAME silicon_scf-RESTART.wfn
    
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
      SCF_GUESS RESTART
      &OT
        MINIMIZER DIIS
        PRECONDITIONER FULL_SINGLE_INVERSE
      &END OT
    &END SCF
    
    &PRINT
      &BAND_STRUCTURE
        &KPOINT_SET
          NPOINTS 40
          UNITS B_VECTOR
          SPECIAL_POINT GAMMA 0.0 0.0 0.0
          SPECIAL_POINT X     0.5 0.0 0.5
        &END KPOINT_SET
        &KPOINT_SET
          NPOINTS 40
          UNITS B_VECTOR
          SPECIAL_POINT X     0.5 0.0 0.5
          SPECIAL_POINT W     0.5 0.25 0.75
        &END KPOINT_SET
        &KPOINT_SET
          NPOINTS 40
          UNITS B_VECTOR
          SPECIAL_POINT W     0.5 0.25 0.75
          SPECIAL_POINT K     0.375 0.375 0.75
        &END KPOINT_SET
        &KPOINT_SET
          NPOINTS 40
          UNITS B_VECTOR
          SPECIAL_POINT K     0.375 0.375 0.75
          SPECIAL_POINT GAMMA 0.0 0.0 0.0
        &END KPOINT_SET
        &KPOINT_SET
          NPOINTS 40
          UNITS B_VECTOR
          SPECIAL_POINT GAMMA 0.0 0.0 0.0
          SPECIAL_POINT L     0.5 0.5 0.5
        &END KPOINT_SET
        FILE_NAME silicon.bs
        &END BAND_STRUCTURE
    &END PRINT
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

## 🎯 Key Parameters

- `WFN_RESTART_FILE_NAME` - Use restart file from SCF
- `&BAND_STRUCTURE` - Band structure print section
- `&KPOINT_SET` - Define k-path segments
  - `NPOINTS` - Points between special k-points
  - `SPECIAL_POINT label kx ky kz`
  - `UNITS` - `B_VECTOR` (reciprocal) or `CART_ANGSTROM`

## 🚀 Running

```bash
# Step 1: SCF
cp2k.ssmp -i si_scf.inp -o si_scf.out

# Step 2: Bands
cp2k.ssmp -i si_bands.inp -o si_bands.out
```

## 📊 Output

Creates `silicon.bs` file with band eigenvalues.

Format:
```
# Set  Point  Kind    kx      ky      kz      Weight  Eigenvalue1  Eigenvalue2 ...
  1    1      1    0.0000  0.0000  0.0000   1.0000   -0.20123     0.21234 ...
```

## 🎨 Plotting

Python script:

```python
import numpy as np
import matplotlib.pyplot as plt

# Read band structure file
data = []
with open('silicon.bs', 'r') as f:
    for line in f:
        if not line.startswith('#'):
            data.append([float(x) for x in line.split()])

data = np.array(data)

# Extract eigenvalues (columns 7 onward)
eigenvalues = data[:, 7:]
nkpoints = len(data)
nbands = eigenvalues.shape[1]

# Create k-point path (simple linear for now)
kpath = np.arange(nkpoints)

# Plot
plt.figure(figsize=(8, 6))
for i in range(nbands):
    plt.plot(kpath, eigenvalues[:, i] * 27.211, 'b-', linewidth=0.5)  # Convert to eV

plt.ylabel('Energy (eV)')
plt.xlabel('k-path')
plt.title('Silicon Band Structure')
plt.axhline(y=0, color='r', linestyle='--', alpha=0.5)
plt.ylim(-6, 16)
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('si_bands.png', dpi=300)
```

## 💡 Tips

- CP2K band structure requires explicit k-path definition
- Use restart file from SCF for efficiency
- For complex paths, define multiple KPOINT_SET sections
- Output is in Hartree - convert to eV (×27.211)

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Relaxation Tutorial](../relax/README.md)
