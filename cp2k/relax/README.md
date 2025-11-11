# Structure Optimization with CP2K

CP2K provides efficient geometry optimization using various algorithms.

## 🎯 Key Parameters

- `RUN_TYPE GEO_OPT` - Atomic position optimization
- `RUN_TYPE CELL_OPT` - Full cell + atomic optimization
- `OPTIMIZER` - BFGS, LBFGS, CG
- `MAX_ITER` - Maximum optimization steps
- `MAX_FORCE` - Force convergence criterion

## 📝 Example: Geometry Optimization (si_relax.inp)

```
&GLOBAL
  PROJECT silicon_relax
  RUN_TYPE GEO_OPT
  PRINT_LEVEL MEDIUM
&END GLOBAL

&MOTION
  &GEO_OPT
    OPTIMIZER BFGS
    MAX_ITER 200
    MAX_FORCE 1.0E-4
    RMS_FORCE 1.0E-5
    
    &BFGS
      TRUST_RADIUS 0.25
    &END BFGS
  &END GEO_OPT
  
  &PRINT
    &TRAJECTORY
      &EACH
        GEO_OPT 1
      &END EACH
    &END TRAJECTORY
    &RESTART_HISTORY OFF
    &END RESTART_HISTORY
  &END PRINT
&END MOTION

&FORCE_EVAL
  METHOD Quickstep
  
  &DFT
    BASIS_SET_FILE_NAME  BASIS_MOLOPT
    POTENTIAL_FILE_NAME  GTH_POTENTIALS
    
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
      &OT
        MINIMIZER DIIS
        PRECONDITIONER FULL_SINGLE_INVERSE
      &END OT
    &END SCF
    
    &KPOINTS
      SCHEME MONKHORST-PACK 8 8 8
    &END KPOINTS
  &END DFT
  
  &SUBSYS
    &CELL
      ABC 5.43 5.43 5.43
      ALPHA_BETA_GAMMA 60 60 60
    &END CELL
    
    &COORD
      SCALED
      Si  0.00  0.00  0.00
      Si  0.26  0.26  0.26  # Displaced
    &END COORD
    
    &KIND Si
      BASIS_SET DZVP-MOLOPT-SR-GTH
      POTENTIAL GTH-PBE-q4
    &END KIND
  &END SUBSYS
  
  &PRINT
    &FORCES ON
    &END FORCES
  &END PRINT
&END FORCE_EVAL
```

## 📝 Variable Cell Optimization

```
&GLOBAL
  PROJECT silicon_cell_opt
  RUN_TYPE CELL_OPT
&END GLOBAL

&MOTION
  &CELL_OPT
    OPTIMIZER BFGS
    MAX_ITER 200
    MAX_FORCE 1.0E-4
    PRESSURE_TOLERANCE 100  # Pressure in bar
    
    &BFGS
      TRUST_RADIUS 0.25
    &END BFGS
  &END CELL_OPT
&END MOTION

&FORCE_EVAL
  METHOD Quickstep
  STRESS_TENSOR ANALYTICAL
  
  &DFT
    # ... same as above ...
  &END DFT
  
  &SUBSYS
    &CELL
      ABC 5.40 5.40 5.40  # Approximate values
      ALPHA_BETA_GAMMA 60 60 60
    &END CELL
    # ... rest same as above ...
  &END SUBSYS
&END FORCE_EVAL
```

## 🚀 Running

```bash
cp2k.ssmp -i si_relax.inp -o si_relax.out
```

## 📊 Output

Optimization progress:
```
 --------  Informations at step =     1 ------------
  Optimization Method        =                 BFGS
  Total Energy               =       -15.84512345
  Real energy change         =        -0.00112345
  Max. gradient              =         0.01234567
  Conv. limit for gradients  =         0.00010000
  Conv. in gradients         =               NO
```

Convergence:
```
 *** Geometry optimization run converged in  12 steps ***
 
 GEOMETRY OPTIMIZATION COMPLETED
```

Final structure written to `silicon_relax-pos-1.xyz`

## 🔗 Next Steps

- [SCF Tutorial](../scf/README.md)
- [Band Structure Tutorial](../bands/README.md)
