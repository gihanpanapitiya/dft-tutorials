# Structure Optimization with SIESTA

SIESTA offers several geometry optimization algorithms to find equilibrium atomic positions and lattice parameters.

## 🎯 Key Parameters

- `MD.TypeOfRun` - Type of run: `CG`, `Broyden`, `FIRE`
- `MD.NumCGsteps` - Maximum optimization steps
- `MD.MaxForceTol` - Force tolerance (eV/Ang)
- `MD.MaxStressTol` - Stress tolerance (GPa)
- `MD.VariableCell` - `.true.` for variable cell

## 📝 Example: Atomic Relaxation (si_relax.fdf)

```
SystemName          Silicon relaxation
SystemLabel         si_relax

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
  0.26  0.26  0.26  1  # Slightly displaced
%endblock AtomicCoordinatesAndAtomicSpecies

PAO.BasisSize       DZP
MeshCutoff          300.0 Ry

XC.functional       GGA
XC.authors          PBE

%block kgrid_Monkhorst_Pack
  8  0  0  0.0
  0  8  0  0.0
  0  0  8  0.0
%endblock kgrid_Monkhorst_Pack

MaxSCFIterations    50
DM.MixingWeight     0.25
DM.Tolerance        1.0d-4

# Geometry optimization
MD.TypeOfRun        CG                 # Conjugate gradient
MD.NumCGsteps       100                # Max steps
MD.MaxForceTol      0.01 eV/Ang        # Force tolerance
MD.UseSaveXV        .true.             # Use .XV file if exists

WriteCoorStep       .true.
WriteForces         .true.
WriteMDHistory      .true.
```

## 📝 Variable Cell Optimization

```
# Add to above:
MD.VariableCell     .true.
MD.MaxStressTol     0.01 GPa           # Stress tolerance
MD.RelaxCellOnly    .false.            # Relax atoms too

# Optional: constrain specific cell parameters
%block MD.TargetStress
  0.0  0.0  0.0                        # Target stress (GPa)
  0.0  0.0  0.0
%endblock MD.TargetStress
```

## 🚀 Running

```bash
siesta < si_relax.fdf > si_relax.out
```

Output files:
- `si_relax.XV` - Final coordinates and velocities
- `si_relax.ANI` - Animation of optimization

## 📊 Output

Each MD step shows:
```
siesta: Ekin =       0.000000
siesta: Etot =     -15.847123
siesta: E_KS =     -15.847123

siesta: Atomic forces (eV/Ang):
siesta:      1    0.012345    0.012345    0.012345
siesta:      2   -0.012345   -0.012345   -0.012345
siesta: ----------------------------------------
siesta:    Tot    0.000000    0.000000    0.000000

siesta: Maximum force:     0.021356 eV/Ang
```

Convergence message:
```
outcoor: Relaxed atomic coordinates (fractional):
    0.00000000    0.00000000    0.00000000   1       1  Si
    0.25000000    0.25000000    0.25000000   1       2  Si

siesta: Program's energy decomposition (eV):
siesta: Etot     =       -15.847123
```

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Band Structure Tutorial](../bands/README.md)
