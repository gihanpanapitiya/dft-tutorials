# Visualization Tools for DFT Results

This guide covers tools and techniques for visualizing and analyzing DFT calculation results.

## 🎨 Crystal Structure Visualization

### VESTA (Recommended)
**Best for**: Crystal structures, charge density, orbitals

**Features**:
- Cross-platform (Windows, Mac, Linux)
- Read many formats
- Beautiful publication-quality graphics
- Isosurfaces, slices, bonds
- Free!

**Download**: [jp-minerals.org/vesta](https://jp-minerals.org/vesta/)

**Supported formats**:
- CIF, XSF, CUBE
- VASP (POSCAR, CHGCAR)
- QE output
- And many more

### XCrySDen
**Best for**: Periodic structures, Fermi surfaces, band structure

**Features**:
- K-path selection tool
- Fermi surface visualization
- Force vectors
- Free and open-source

**Download**: [xcrysden.org](http://www.xcrysden.org/)

**Formats**: XSF, CIF, QE input/output

### Avogadro
**Best for**: Molecules, building structures

**Features**:
- Build molecules interactively
- Optimize geometry
- Simple and intuitive
- Free

**Download**: [avogadro.cc](https://avogadro.cc/)

### Jmol
**Best for**: Web-based viewing, animations

**Features**:
- Java-based (runs anywhere)
- Interactive rotation
- Scripting
- Free

**Download**: [jmol.sourceforge.net](http://jmol.sourceforge.net/)

## 📊 Band Structure and DOS Plotting

### Python (Matplotlib)
**Best for**: Customizable publication plots

Basic example:
```python
import numpy as np
import matplotlib.pyplot as plt

# Read band structure data
data = np.loadtxt('bands.dat')
kpath = data[:, 0]
nbands = data.shape[1] - 1

# Plot
fig, ax = plt.subplots(figsize=(6, 8))
for i in range(1, nbands + 1):
    ax.plot(kpath, data[:, i], 'b-', linewidth=0.5)

# Add Fermi level
ax.axhline(y=0, color='r', linestyle='--', label='E_F')

# High-symmetry points
high_sym = {'Γ': 0, 'X': 1.5, 'W': 3.0, 'K': 4.2, 'Γ': 5.5, 'L': 7.0}
for label, pos in high_sym.items():
    ax.axvline(x=pos, color='k', linewidth=0.5, alpha=0.3)

ax.set_xticks(list(high_sym.values()))
ax.set_xticklabels(list(high_sym.keys()))
ax.set_ylabel('Energy (eV)')
ax.set_ylim(-8, 8)
ax.legend()
plt.tight_layout()
plt.savefig('bands.png', dpi=300)
```

### pymatgen
**Best for**: Automated analysis, Materials Project integration

```python
from pymatgen.electronic_structure.plotter import BSPlotter
from pymatgen.io.vasp import BSVasprun

# Read band structure
bs_vasprun = BSVasprun("vasprun.xml")
bs = bs_vasprun.get_band_structure(line_mode=True)

# Plot
plotter = BSPlotter(bs)
plotter.get_plot().savefig("bands.png")
```

### BoltzTraP
**Best for**: Transport properties from band structure

Calculate:
- Seebeck coefficient
- Electrical conductivity
- Electronic thermal conductivity

### sumo
**Best for**: Pretty band structure and DOS plots

```bash
# Install
pip install sumo

# Plot bands
sumo-bandplot --filename bands.png

# Plot DOS
sumo-dosplot --filename dos.png
```

## 📈 Density of States (DOS)

### Python Example

```python
import numpy as np
import matplotlib.pyplot as plt

# Read DOS data (energy, DOS)
energy, dos = np.loadtxt('dos.dat', unpack=True)

plt.figure(figsize=(6, 5))
plt.plot(energy, dos, 'b-', linewidth=1.5)
plt.axvline(x=0, color='r', linestyle='--', label='Fermi level')
plt.fill_between(energy[energy <= 0], dos[energy <= 0], alpha=0.3)
plt.xlabel('Energy (eV)')
plt.ylabel('DOS (states/eV)')
plt.xlim(-10, 10)
plt.ylim(0, None)
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('dos.png', dpi=300)
```

### Projected DOS (PDOS)

```python
# Read PDOS for different orbitals
energy, dos_s, dos_p, dos_d = np.loadtxt('pdos.dat', unpack=True)

fig, ax = plt.subplots(figsize=(6, 5))
ax.plot(energy, dos_s, label='s', linewidth=1.5)
ax.plot(energy, dos_p, label='p', linewidth=1.5)
ax.plot(energy, dos_d, label='d', linewidth=1.5)
ax.axvline(x=0, color='k', linestyle='--', alpha=0.5)
ax.set_xlabel('Energy (eV)')
ax.set_ylabel('PDOS (states/eV)')
ax.legend()
plt.tight_layout()
plt.savefig('pdos.png', dpi=300)
```

## 🗺️ Charge Density Visualization

### Using VESTA

1. Generate cube file from your DFT code
2. Open in VESTA
3. `Edit → Volumetric Data → New Surface`
4. Adjust isosurface value
5. `Style → Surfaces` to adjust appearance

### Python (for 2D slices)

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.io import FortranFile

# Read charge density (example for QE format)
# Adjust reader based on your code's output format

def plot_density_2d(density_3d, plane='xy', z_index=None):
    """Plot 2D slice of 3D charge density"""
    
    if plane == 'xy':
        data = density_3d[:, :, z_index]
    elif plane == 'xz':
        data = density_3d[:, z_index, :]
    elif plane == 'yz':
        data = density_3d[z_index, :, :]
    
    plt.figure(figsize=(6, 5))
    plt.imshow(data.T, origin='lower', cmap='viridis', 
               interpolation='bilinear')
    plt.colorbar(label='Charge density (e/Bohr³)')
    plt.xlabel('x')
    plt.ylabel('y' if plane == 'xy' else 'z')
    plt.title(f'Charge Density ({plane} plane)')
    plt.tight_layout()
    plt.savefig(f'density_{plane}.png', dpi=300)

# Usage
# density = load_your_density_file()
# plot_density_2d(density, 'xy', z_index=50)
```

## 🎬 Animation and Trajectories

### ASE (Atomic Simulation Environment)

```python
from ase.io import read, write
from ase.visualize import view

# Read trajectory
atoms = read('trajectory.xyz', index=':')

# View animation
view(atoms)

# Save as animated GIF
write('animation.gif', atoms, interval=100)

# Create movie
from ase.io.trajectory import Trajectory
traj = Trajectory('md.traj')
write('movie.xyz', traj)
```

### VMD (Visual Molecular Dynamics)

**Best for**: MD trajectories, large systems

```bash
# Load and visualize
vmd structure.xyz trajectory.xyz

# Create movie
vmd -dispdev text -e render_movie.tcl
```

## 📐 Phonon Visualization

### Phonopy

```bash
# Install
pip install phonopy

# Create animation of phonon mode
phonopy-bandplot --gnuplot > phonon.dat

# Visualize specific mode
phonopy --anime=10  # Mode 10
```

### Python plotting

```python
import numpy as np
import matplotlib.pyplot as plt

# Read phonon dispersion
data = np.loadtxt('phonon.dat')
kpath = data[:, 0]
frequencies = data[:, 1:]  # THz or cm^-1

plt.figure(figsize=(6, 5))
for i in range(frequencies.shape[1]):
    plt.plot(kpath, frequencies[:, i], 'b-', linewidth=0.5)

plt.ylabel('Frequency (THz)')
plt.xlabel('k-path')
plt.axhline(y=0, color='r', linestyle='--', linewidth=0.5)
plt.savefig('phonons.png', dpi=300)
```

## 🔧 Post-Processing Tools by Code

### Quantum ESPRESSO

**pp.x**: Post-processing tool
```bash
# Extract charge density
pp.x < pp.in > pp.out
```

**plotband.x**: Plot band structure
```bash
plotband.x
# Follow prompts
```

**dos.x**: Calculate DOS
```bash
dos.x < dos.in > dos.out
```

**projwfc.x**: Projected DOS
```bash
projwfc.x < projwfc.in > projwfc.out
```

### ABINIT

**cut3d**: Analyze 3D files
```bash
cut3d
# Interactive prompts
```

**abipy**: Python library
```python
from abipy import abilab

# Open GSR file
with abilab.abiopen("run_GSR.nc") as gsr:
    # Plot band structure
    gsr.ebands.plot()
    
    # Plot DOS
    gsr.ebands.get_edos().plot()
```

### SIESTA

**denchar**: Charge density analysis
```bash
denchar < denchar.fdf
```

**gnubands**: Plot bands
```bash
# Part of SIESTA utils
gnubands < input.bands > output.gp
gnuplot output.gp
```

### CP2K

**Cubecruncher**: Process cube files
- Built into CP2K for density analysis

**Python**: Direct reading
```python
import numpy as np

def read_cube(filename):
    """Read Gaussian cube file"""
    with open(filename, 'r') as f:
        # Skip comments
        f.readline()
        f.readline()
        
        # Read header
        natoms = int(f.readline().split()[0])
        origin = [float(x) for x in f.readline().split()[1:]]
        
        # Grid points
        nx, dx = int(f.readline().split()[0]), float(f.readline().split()[1])
        ny, dy = int(f.readline().split()[0]), float(f.readline().split()[2])
        nz, dz = int(f.readline().split()[0]), float(f.readline().split()[3])
        
        # Skip atoms
        for _ in range(natoms):
            f.readline()
        
        # Read data
        data = np.fromfile(f, sep=' ').reshape((nx, ny, nz))
    
    return data, (nx, ny, nz), origin
```

## 📊 Publication-Quality Figures

### Matplotlib Styling

```python
import matplotlib.pyplot as plt

# Set publication style
plt.rcParams.update({
    'font.size': 12,
    'font.family': 'serif',
    'font.serif': ['Times New Roman'],
    'axes.linewidth': 1.5,
    'lines.linewidth': 2,
    'xtick.major.width': 1.5,
    'ytick.major.width': 1.5,
    'xtick.direction': 'in',
    'ytick.direction': 'in',
    'figure.dpi': 300,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight'
})

# Create plot
fig, ax = plt.subplots(figsize=(3.5, 2.5))  # Single column width
# ... your plot ...
plt.savefig('figure.pdf')  # Vector format for publication
```

### Multi-panel Figures

```python
from matplotlib.gridspec import GridSpec

fig = plt.figure(figsize=(7, 5))
gs = GridSpec(2, 2, figure=fig, hspace=0.3, wspace=0.3)

ax1 = fig.add_subplot(gs[0, 0])
# Plot 1

ax2 = fig.add_subplot(gs[0, 1])
# Plot 2

ax3 = fig.add_subplot(gs[1, :])  # Span both columns
# Plot 3

plt.savefig('multi_panel.pdf')
```

## 🎨 Color Maps

```python
import matplotlib.pyplot as plt

# Recommended colormaps
# Perceptually uniform:
- 'viridis' (default, good for most)
- 'plasma'
- 'cividis' (colorblind-friendly)

# Diverging:
- 'RdBu_r' (red-blue, reversed)
- 'seismic'

# Sequential:
- 'Blues'
- 'Reds'

# Usage
plt.imshow(data, cmap='viridis')
```

## 🛠️ Useful Python Libraries

### Essential
```bash
pip install numpy matplotlib scipy
```

### Advanced
```bash
pip install ase pymatgen phonopy abipy
```

### Plotting
```bash
pip install seaborn plotly  # Enhanced plots
```

## 📚 Resources

### Tutorials
- [ASE Tutorial](https://wiki.fysik.dtu.dk/ase/tutorials/tutorials.html)
- [Pymatgen Docs](https://pymatgen.org/)
- [Matplotlib Gallery](https://matplotlib.org/stable/gallery/)

### Examples
- Look at published papers in your field
- Check code-specific tutorials
- Materials Project examples

---

**Key Takeaways**:
- **VESTA** for structures and densities
- **Python/Matplotlib** for customizable plots
- **ASE** for trajectories and automation
- **Code-specific tools** for specialized tasks
- Always create **publication-quality figures** from the start!
