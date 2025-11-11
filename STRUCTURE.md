# Repository Structure

```
dft/
│
├── README.md                          # Main repository overview
│
├── docs/                           # General resources
│   ├── theory.md                  # **NEW!** Theoretical foundations: Kohn-Sham orbitals, pseudopotentials, DFT formalism
│   ├── common-concepts.md         # DFT basics, XC functionals, k-points
│   ├── pseudopotentials.md        # Pseudopotential guide
│   ├── convergence.md             # Convergence testing methodology
│   └── visualization.md           # Visualization tools
│
├── quantum-espresso/                  # Quantum ESPRESSO tutorials
│   ├── README.md                     # QE overview and setup
│   ├── scf/                          # SCF calculations
│   │   ├── README.md                 # SCF tutorial
│   │   ├── si_scf.in                 # Silicon SCF example
│   │   └── al_scf.in                 # Aluminum (metal) example
│   ├── relax/                        # Structure optimization
│   │   ├── README.md                 # Relaxation tutorial
│   │   ├── si_relax.in               # Atomic relaxation
│   │   └── si_vc-relax.in            # Variable cell relaxation
│   └── bands/                        # Band structure
│       ├── README.md                 # Band structure tutorial
│       ├── si_scf.in                 # Step 1: SCF
│       ├── si_bands.in               # Step 2: Bands calculation
│       └── si_bands_pp.in            # Step 3: Post-processing
│
├── abinit/                            # ABINIT tutorials
│   ├── README.md                     # ABINIT overview and setup
│   ├── scf/                          # SCF calculations
│   │   ├── README.md                 # SCF tutorial
│   │   └── si_scf.abi                # Silicon SCF example
│   ├── relax/                        # Structure optimization
│   │   ├── README.md                 # Relaxation tutorial
│   │   └── si_relax.abi              # Atomic relaxation example
│   └── bands/                        # Band structure
│       ├── README.md                 # Band structure tutorial
│       └── si_bands.abi              # Multi-dataset example
│
├── siesta/                            # SIESTA tutorials
│   ├── README.md                     # SIESTA overview and setup
│   ├── scf/                          # SCF calculations
│   │   ├── README.md                 # SCF tutorial
│   │   └── si_scf.fdf                # Silicon SCF example
│   ├── relax/                        # Structure optimization
│   │   ├── README.md                 # Relaxation tutorial
│   │   └── si_relax.fdf              # Atomic relaxation example
│   └── bands/                        # Band structure
│       ├── README.md                 # Band structure tutorial
│       └── si_bands.fdf              # Band structure example
│
└── cp2k/                              # CP2K tutorials
    ├── README.md                     # CP2K overview and setup
    ├── scf/                          # SCF calculations
    │   ├── README.md                 # SCF tutorial
    │   └── si_scf.inp                # Silicon SCF example
    ├── relax/                        # Structure optimization
    │   ├── README.md                 # Relaxation tutorial
    │   └── si_relax.inp              # Geometry optimization example
    └── bands/                        # Band structure
        ├── README.md                 # Band structure tutorial
        └── si_bands.inp              # Band structure example
```

## Quick Navigation Guide

## 🔍 Quick Navigation

### "I want to understand the theory behind DFT..."
→ Start here: **[docs/theory.md](docs/theory.md)** - Learn about Kohn-Sham orbitals, pseudopotentials, many-body problem, exchange-correlation functionals

### "I want to learn DFT basics..."
→ Start here: **[docs/common-concepts.md](docs/common-concepts.md)**

### I want to understand pseudopotentials
→ Read [`docs/pseudopotentials.md`](docs/pseudopotentials.md)

### I want to learn convergence testing
→ Check [`docs/convergence.md`](docs/convergence.md)

### I want to visualize my results
→ See [`docs/visualization.md`](docs/visualization.md)

### I want to use Quantum ESPRESSO
→ Start with [`quantum-espresso/README.md`](quantum-espresso/README.md)
→ Then: [SCF](quantum-espresso/scf/), [Relax](quantum-espresso/relax/), [Bands](quantum-espresso/bands/)

### I want to use ABINIT
→ Start with [`abinit/README.md`](abinit/README.md)
→ Then: [SCF](abinit/scf/), [Relax](abinit/relax/), [Bands](abinit/bands/)

### I want to use SIESTA
→ Start with [`siesta/README.md`](siesta/README.md)
→ Then: [SCF](siesta/scf/), [Relax](siesta/relax/), [Bands](siesta/bands/)

### I want to use CP2K
→ Start with [`cp2k/README.md`](cp2k/README.md)
→ Then: [SCF](cp2k/scf/), [Relax](cp2k/relax/), [Bands](cp2k/bands/)

## Learning Path

### Beginner
1. Read `docs/theory.md` to understand the fundamentals
2. Read `docs/common-concepts.md` for practical DFT concepts
3. Choose a code (Quantum ESPRESSO recommended for beginners)
4. Follow the SCF tutorial for your chosen code
5. Run the provided examples
6. Understand the output

### Intermediate
1. Learn about pseudopotentials (`docs/pseudopotentials.md`)
2. Perform convergence tests (`docs/convergence.md`)
3. Try structure optimization
4. Calculate band structures
5. Visualize your results (`docs/visualization.md`)

### Advanced
1. Compare different codes for the same system
2. Optimize your workflows
3. Automate calculations with scripts
4. Explore advanced features (phonons, defects, surfaces)
5. Contribute to this repository!

## File Naming Conventions

- **README.md**: Tutorial and documentation
- **\*.in**: Quantum ESPRESSO input files
- **\*.abi**: ABINIT input files
- **\*.fdf**: SIESTA input files
- **\*.inp**: CP2K input files
- **\*.md**: Markdown documentation

## Getting Help

1. Check the relevant README.md for your code
2. Look at example input files
3. Read the troubleshooting sections
4. Consult official documentation (links in each code's README)
5. Search for error messages online
6. Ask questions on code-specific forums/mailing lists

## Contributing

Found an error? Want to add an example? Contributions welcome!

---

*Repository last updated: November 2025*
