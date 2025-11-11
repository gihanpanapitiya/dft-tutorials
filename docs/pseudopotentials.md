# Pseudopotential Guide

Pseudopotentials replace the core electrons and strong nuclear potential with an effective potential, making DFT calculations tractable.

## 🎯 Why Pseudopotentials?

### The Problem
- Core electrons oscillate rapidly → need huge basis set
- Core electrons don't participate in bonding
- Waste computational resources on chemically inactive electrons

### The Solution
Pseudopotentials:
1. **Freeze core electrons** (treat as part of nucleus)
2. **Keep only valence electrons** (chemically active)
3. **Use smoother potential** (easier to represent)

## 📚 Types of Pseudopotentials

### 1. Norm-Conserving (NC)

**Properties**:
- Pseudo and all-electron charge match inside core region
- Scattering properties preserved
- Most transferable

**Pros**:
- Accurate
- Transferable to different environments
- Theoretically clean

**Cons**:
- Higher plane-wave cutoffs required
- Less efficient for heavy elements

**Common formats**:
- Troullier-Martins
- HGH/GTH (Goedecker-Teter-Hutter)
- FHI98PP

**When to use**: High accuracy needed, light elements

### 2. Ultrasoft Pseudopotentials (USPP)

**Properties**:
- Relax norm-conservation requirement
- Introduce augmentation charges
- Much softer potentials

**Pros**:
- Lower cutoffs (50-75% of NC)
- Efficient for heavy elements (transition metals)

**Cons**:
- More complex formalism
- Need charge augmentation
- Slightly less transferable

**Common in**: Quantum ESPRESSO (Vanderbilt type)

**When to use**: Standard production calculations, heavy elements

### 3. Projector Augmented Wave (PAW)

**Properties**:
- Reconstruction of all-electron wavefunctions
- "All-electron" accuracy with pseudopotential efficiency
- Exact core-valence orthogonality

**Pros**:
- Most accurate (all-electron like)
- Good for magnetic properties
- Hyperfine parameters accessible
- Lower cutoffs

**Cons**:
- More expensive than USPP
- More complex

**Common formats**:
- JTH (Jollet-Torrent-Holzwarth)
- GBRV
- VASP PAW

**When to use**: High accuracy, magnetic systems, need core properties

## 🔍 Key Pseudopotential Properties

### 1. Core Radius (r_c)

Radius inside which pseudo wavefunction differs from all-electron.

- Smaller r_c → harder potential → higher cutoff needed
- Larger r_c → softer potential → lower cutoff, but less transferable

### 2. Number of Valence Electrons

Example for Silicon:
- **Full**: 1s² 2s² 2p⁶ 3s² 3p² (14 electrons)
- **Typical**: 3s² 3p² (4 valence electrons)
- Sometimes: 2s² 2p⁶ 3s² 3p² (12 electrons - semicore)

**When to include semicore**:
- Transition metals: often include (n-1)d electrons
- If bonding involves "semicore" electrons
- High-pressure calculations
- When in doubt: test both!

### 3. Cutoff Recommendations

Each pseudopotential has a recommended cutoff:
- **Ecutwfc** (wavefunction): Essential parameter
- **Ecutrho** (density): Usually 4-8× ecutwfc for NC, 8-12× for USPP

**Always test cutoff convergence!**

## 📊 Pseudopotential Libraries

### Quantum ESPRESSO

#### SSSP (Standard Solid State Pseudopotentials)
- **Location**: [Materials Cloud SSSP](https://www.materialscloud.org/discover/sssp/)
- **Quality**: Precision and efficiency versions
- **Format**: UPF (USPP and PAW)
- **Functional**: PBE, PBEsol
- **Recommended**: Yes! Well-tested, documented

#### Pslibrary
- **Location**: [QE website](https://www.quantum-espresso.org/pseudopotentials/)
- **Formats**: NC, USPP, PAW
- **Functionals**: Multiple (PBE, PW91, etc.)

#### Pseudo Dojo
- **Location**: [pseudo-dojo.org](http://www.pseudo-dojo.org/)
- **Quality**: High-quality NC and PAW
- **Testing**: Extensively validated
- **Formats**: UPF, ABINIT

### ABINIT

#### JTH PAW Table
- **Quality**: High-quality PAW
- **Functional**: LDA and PBE
- **Format**: XML (PAW)

#### FHI98PP
- **Type**: Norm-conserving
- **Format**: FHI format

#### HGH/GTH
- **Type**: Separable dual-space Gaussian
- **Efficiency**: Very efficient

### SIESTA

#### SIESTA Pseudopotentials
- **Location**: [SIESTA website](https://siesta-project.org/databases/pseudopotentials/)
- **Format**: .psf (SIESTA format)
- **Generation**: ATOM program

**Note**: SIESTA uses different format - not interchangeable with QE/ABINIT

### CP2K

#### GTH Pseudopotentials
- **Type**: Goedecker-Teter-Hutter
- **Format**: Built into CP2K
- **Efficiency**: Very efficient with Gaussian basis
- **Files**: `GTH_POTENTIALS`

## 🎯 Choosing the Right Pseudopotential

### Decision Tree

```
1. What code are you using?
   → Use native format (or convert)

2. What accuracy do you need?
   → High: PAW or good NC
   → Standard: USPP or PAW
   → Quick: USPP

3. What functional?
   → Match functional! (PBE pseudo for PBE calculation)

4. Light elements (H, C, N, O, Si)?
   → NC or USPP fine

5. Heavy/transition metals?
   → USPP or PAW recommended
   → Consider semicore electrons

6. Magnetic properties?
   → PAW preferred

7. Time/resources limited?
   → USPP or efficient NC (GTH)
```

### Recommended Choices (2024)

**Quantum ESPRESSO**:
- Start with: **SSSP efficiency** library
- High accuracy: **SSSP precision** or **Pseudo Dojo PAW**

**ABINIT**:
- Standard: **JTH PAW** (PBE or LDA)
- Alternative: **Pseudo Dojo**

**SIESTA**:
- Use **SIESTA pseudopotential database**
- Or generate with ATOM program

**CP2K**:
- Standard: **GTH-PBE** potentials (built-in)
- Match with MOLOPT basis sets

## ⚠️ Common Mistakes

### 1. Functional Mismatch
❌ **Wrong**: PBE calculation with LDA pseudopotential
✅ **Right**: PBE calculation with PBE pseudopotential

### 2. Cutoff Too Low
- Always check recommended cutoff
- Test convergence!

### 3. Wrong Valence
- Check number of valence electrons makes sense
- For Si: should be 4 (3s² 3p²), sometimes 12 if including 2s²2p⁶

### 4. Old/Untested Pseudopotentials
- Use well-maintained libraries
- Check if pseudopotential is validated

### 5. Mixing Incompatible Types
- Don't mix NC and USPP in same calculation (some codes allow, but careful)
- All atoms should use same library/quality level

## 🧪 Testing Pseudopotentials

### Basic Tests

#### 1. Cutoff Convergence
```python
cutoffs = [30, 40, 50, 60, 80, 100]  # Ry
for ecut in cutoffs:
    # Run calculation
    # Plot energy vs cutoff
```

#### 2. Transferability
Test in different chemical environments:
- Isolated atom
- Dimer
- Bulk
- Surface

#### 3. Lattice Constant
Compare to:
- Experiment
- All-electron calculations
- Other pseudopotentials

#### 4. Cohesive Energy
Check if binding energies reasonable.

### Advanced Tests

#### 1. Pressure Dependence
Calculate at different volumes - check if smooth.

#### 2. Ghost States
Plot wavefunction energies vs cutoff - should be smooth.

#### 3. Phonons
Calculate phonon frequencies - check for imaginary modes (instabilities).

## 📝 Pseudopotential Nomenclature

### Example: `Si.pbe-n-rrkjus_psl.1.0.0.UPF`

- **Si**: Element
- **pbe**: Functional (PBE)
- **n**: Norm-conserving
- **rrkjus**: Generation method/author
- **psl**: Pslibrary
- **1.0.0**: Version
- **UPF**: Format (Unified Pseudopotential Format)

### Example: `GTH-PBE-q4`

- **GTH**: Goedecker-Teter-Hutter type
- **PBE**: Functional
- **q4**: 4 valence electrons

## 🔗 Useful Resources

### Pseudopotential Databases
- [SSSP](https://www.materialscloud.org/discover/sssp/)
- [Pseudo Dojo](http://www.pseudo-dojo.org/)
- [Quantum ESPRESSO](https://www.quantum-espresso.org/pseudopotentials/)
- [ABINIT](https://www.abinit.org/psp-tables)
- [SIESTA](https://siesta-project.org/databases/pseudopotentials/)

### Papers
- Vanderbilt (1990) - Soft pseudopotentials
- Blöchl (1994) - PAW method
- Hamann (2013) - Optimized norm-conserving
- Jollet et al. (2014) - JTH PAW
- Prandini et al. (2018) - SSSP validation

### Tools
- **ONCVPSP**: Generate NC pseudopotentials
- **LD1.x**: Generate pseudopotentials (QE)
- **ATOMPAW**: Generate PAW datasets
- **ATOM**: Generate SIESTA pseudopotentials

## 📊 Quick Reference Table

| Element | Valence | Typical Cutoff (Ry) | Notes |
|---------|---------|---------------------|-------|
| H | 1 | 40-60 | Needs high cutoff |
| C | 4 | 40-60 | - |
| N, O | 5, 6 | 50-70 | - |
| Si | 4 | 30-40 | Easy |
| Al | 3 | 30-40 | Easy, metal |
| Fe | 8 or 16 | 50-80 | Include 3d, semicore? |
| Cu | 11 or 19 | 50-80 | Include 3d |
| Au | 11 or 19 | 40-60 | Relativistic effects |

## 💡 Best Practices

1. **Always** use well-tested libraries (SSSP, Pseudo Dojo, JTH)
2. **Match** functional between pseudopotential and calculation
3. **Test** cutoff convergence for your system
4. **Document** which pseudopotentials you used (including version!)
5. **Check** number of valence electrons is correct
6. **Consider** semicore for transition metals and high-pressure
7. **Be consistent** - use same library for all elements
8. **Cite** the pseudopotential library in publications

---

**Key Takeaway**: Modern pseudopotential libraries (SSSP, Pseudo Dojo, JTH) are well-tested and reliable. Start there, match your functional, and always test cutoff convergence!
