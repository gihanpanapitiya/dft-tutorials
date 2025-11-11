# Band Structure Calculations with Quantum ESPRESSO

Electronic band structure calculations reveal the energy levels of electrons as a function of momentum (k-vector), providing insight into electronic and optical properties.

## 📖 Theory

Band structure calculation is a two-step process:
1. **SCF calculation** - Get converged charge density
2. **Non-SCF (bands) calculation** - Compute eigenvalues along high-symmetry k-path

The band structure shows:
- Band gap (for semiconductors/insulators)
- Valence and conduction bands
- Effective masses
- Direct vs. indirect gaps

## 🎯 What You'll Learn

- Perform SCF + bands calculations
- Choose high-symmetry k-paths
- Post-process band structure data
- Plot band diagrams

## 📝 Workflow

### Step 1: SCF Calculation

First, perform a standard SCF calculation with a good k-point mesh.

**File: si_scf.in**

```fortran
&CONTROL
  calculation = 'scf'
  prefix = 'silicon'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
/

&SYSTEM
  ibrav = 2
  celldm(1) = 10.26
  nat = 2
  ntyp = 1
  ecutwfc = 30.0
  ecutrho = 240.0
  nbnd = 8
/

&ELECTRONS
  conv_thr = 1.0d-8
/

ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25

K_POINTS (automatic)
  8 8 8 0 0 0
```

Run:
```bash
pw.x < si_scf.in > si_scf.out
```

### Step 2: Bands Calculation

Non-self-consistent calculation along high-symmetry path.

**File: si_bands.in**

```fortran
&CONTROL
  calculation = 'bands'
  prefix = 'silicon'
  outdir = './tmp/'
  pseudo_dir = './pseudo/'
  verbosity = 'high'
/

&SYSTEM
  ibrav = 2
  celldm(1) = 10.26
  nat = 2
  ntyp = 1
  ecutwfc = 30.0
  ecutrho = 240.0
  nbnd = 8
/

&ELECTRONS
  conv_thr = 1.0d-8
/

ATOMIC_SPECIES
  Si 28.086 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS (alat)
  Si 0.00 0.00 0.00
  Si 0.25 0.25 0.25

K_POINTS {crystal_b}
  6
  0.0000 0.0000 0.0000 30  !Gamma
  0.5000 0.0000 0.5000 30  !X
  0.6250 0.2500 0.6250 1   !U
  0.3750 0.3750 0.7500 30  !K
  0.0000 0.0000 0.0000 30  !Gamma
  0.5000 0.5000 0.5000 30  !L
```

Run:
```bash
pw.x < si_bands.in > si_bands.out
```

### Step 3: Post-Processing

Extract and format band data using `bands.x`.

**File: si_bands_pp.in**

```fortran
&BANDS
  prefix = 'silicon'
  outdir = './tmp/'
  filband = 'si_bands.dat'
/
```

Run:
```bash
bands.x < si_bands_pp.in > si_bands_pp.out
```

This creates `si_bands.dat` and `si_bands.dat.gnu` files.

## 📊 Key Parameters

### For bands calculation:

- `calculation = 'bands'` - Non-SCF band structure calculation
- `nbnd` - Number of bands to compute (should include empty bands)
- `verbosity = 'high'` - Prints band energies

### K-point path specification:

**Format**: `K_POINTS {crystal_b}`
```
N_points
kx ky kz  n_points_to_next
...
```

- `crystal_b` - k-points in crystal coordinates with bands path
- Number after coordinates = points between this and next k-point
- Use `1` for last point in a segment

### High-Symmetry Points for FCC (Silicon):

- Γ (Gamma): (0.0, 0.0, 0.0)
- X: (0.5, 0.0, 0.5)
- W: (0.5, 0.25, 0.75)
- K: (0.375, 0.375, 0.75)
- L: (0.5, 0.5, 0.5)

Common path: Γ → X → U/K → Γ → L

## 🎨 Plotting Band Structure

### Python script (plot_bands.py):

```python
import numpy as np
import matplotlib.pyplot as plt

def read_bands_data(filename):
    """Read band structure data from QE bands.x output"""
    with open(filename, 'r') as f:
        lines = f.readlines()
    
    # Skip header, read data
    data = []
    for line in lines:
        if line.strip() and not line.startswith('#'):
            data.append([float(x) for x in line.split()])
    
    data = np.array(data)
    return data

# Read the bands.dat.gnu file
data = np.loadtxt('si_bands.dat.gnu')

# Extract k-points and bands
# Format: each block is separated by blank line
# k-point, band1, band2, ..., bandN

# Reshape data
nkpoints = len(np.unique(data[:, 0]))
nbands = len(data) // nkpoints

kpoints = data[::nbands, 0]
bands = data[:, 1].reshape(nbands, nkpoints)

# Plot
plt.figure(figsize=(8, 6))
for i in range(nbands):
    plt.plot(kpoints, bands[i, :], 'b-', linewidth=0.5)

# Add high-symmetry point labels (you need to determine positions)
# These positions depend on your k-path
high_sym_points = {
    'Γ': 0.0,
    'X': 1.0,
    'K': 2.0,
    'Γ': 3.0,
    'L': 4.0
}

for label, pos in high_sym_points.items():
    plt.axvline(x=pos, color='k', linewidth=0.5, linestyle='--')

plt.xticks(list(high_sym_points.values()), 
           list(high_sym_points.keys()))
plt.ylabel('Energy (eV)')
plt.xlabel('k-path')
plt.title('Silicon Band Structure')
plt.ylim(-6, 6)
plt.axhline(y=0, color='r', linewidth=0.5, linestyle='--', alpha=0.5)
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('si_bands.png', dpi=300)
plt.show()
```

### Alternative: Using plotband.x

QE provides `plotband.x` utility:

```bash
plotband.x
```

Interactive prompts:
```
Input file > si_bands.dat.gnu
Emin, Emax > -6, 16
file with bands > bands.ps
```

## 📊 Reading the Output

### From bands calculation (si_bands.out):

```
     k = 0.0000 0.0000 0.0000 (   137 PWs)   bands (ev):
    -5.4475   5.9157   5.9157   5.9157   8.5245   8.5245   8.5245   9.2850
```

### From bands.dat file:

```
    0.0000    -5.4475
    0.0000     5.9157
    0.0000     5.9157
    ...
```

First column: k-point distance, Second column: energy (eV)

## 🔍 Analysis

### Find band gap:

```python
# From band energies
valence_max = np.max(bands[:4, :])  # Assuming 4 valence bands
conduction_min = np.min(bands[4:, :])  # Conduction bands
band_gap = conduction_min - valence_max
print(f"Band gap: {band_gap:.3f} eV")

# Check if direct or indirect
vb_max_k = np.argmax(bands[3, :])  # Top valence band
cb_min_k = np.argmin(bands[4, :])  # Bottom conduction band

if vb_max_k == cb_min_k:
    print("Direct band gap")
else:
    print("Indirect band gap")
```

## ⚙️ Advanced Topics

### Including spin-orbit coupling:

```fortran
&SYSTEM
  ...
  noncolin = .true.
  lspinorb = .true.
  nbnd = 16  ! Double the bands for spin-orbit
/
```

### Projected band structure:

Use `projwfc.x` to project bands onto atomic orbitals:

```bash
projwfc.x < projwfc.in > projwfc.out
```

### Fat bands (orbital character):

After `projwfc.x`, use auxiliary tools or custom scripts to create "fat bands" showing orbital contributions.

## 🐛 Common Issues

### Issue: Too few bands shown
**Solution**: Increase `nbnd` in input files

### Issue: Discontinuous bands
**Solution**: 
- Increase k-points between high-symmetry points
- Check k-path is continuous

### Issue: Wrong band gap value
**Solution**: 
- DFT (PBE) underestimates band gaps - this is expected
- Use hybrid functionals (HSE) for better gaps
- Apply scissor correction for optical properties

### Issue: Plotting errors
**Solution**: 
- Check format of bands.dat.gnu
- Verify high-symmetry point positions
- Use correct number of bands

## 📚 High-Symmetry Paths for Common Lattices

### FCC (Face-Centered Cubic):
Γ → X → W → K → Γ → L → U → W → L → K

### BCC (Body-Centered Cubic):
Γ → H → N → Γ → P → H

### Hexagonal:
Γ → M → K → Γ → A → L → H → A

### Simple Cubic:
Γ → X → M → Γ → R → X

Use [SeeK-path](https://www.materialscloud.org/work/tools/seekpath) to find paths automatically.

## 🔗 Complete Example Script

**run_bands.sh**:
```bash
#!/bin/bash

# Step 1: SCF
echo "Running SCF calculation..."
pw.x < si_scf.in > si_scf.out

# Step 2: Bands
echo "Running bands calculation..."
pw.x < si_bands.in > si_bands.out

# Step 3: Post-process
echo "Post-processing bands..."
bands.x < si_bands_pp.in > si_bands_pp.out

# Step 4: Plot
echo "Plotting bands..."
python plot_bands.py

echo "Done! Check si_bands.png"
```

Make executable and run:
```bash
chmod +x run_bands.sh
./run_bands.sh
```

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Relaxation Tutorial](../relax/README.md)
- Explore DOS calculations
- Learn about projected band structures

---

**Key Takeaways:**
- Band structure requires SCF + bands calculation
- Choose appropriate high-symmetry k-path
- Use bands.x for post-processing
- DFT underestimates band gaps (known limitation)
- Visualize to understand electronic properties
