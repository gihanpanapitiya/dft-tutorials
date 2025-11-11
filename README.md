# DFT Calculation Tutorials

A comprehensive collection of tutorials and examples for performing Density Functional Theory (DFT) calculations using popular codes: **Quantum ESPRESSO**, **ABINIT**, **SIESTA**, and **CP2K**.

## 📚 Contents

This repository contains step-by-step guides and working examples for three fundamental types of DFT calculations:

1. **SCF (Self-Consistent Field)** - Basic ground state energy calculations
2. **Structure Optimization** - Geometry relaxation to find equilibrium structures
3. **Band Structure** - Electronic band structure calculations

## 🧮 Supported DFT Codes

- [Quantum ESPRESSO](./quantum-espresso/) - Plane-wave pseudopotential code
- [ABINIT](./abinit/) - Plane-wave DFT code with extensive features
- [SIESTA](./siesta/) - Linear combination of atomic orbitals (LCAO) method
- [CP2K](./cp2k/) - Versatile code supporting various methods (GPW, GAPW)

## 🚀 Quick Start

Each code directory contains:
- `README.md` - Overview and installation instructions
- `scf/` - Self-consistent field calculation examples
- `relax/` - Structure optimization examples
- `bands/` - Band structure calculation examples
- `examples/` - Complete working examples for common systems

## 📖 Tutorial Structure

Each tutorial includes:
- **Theory Background** - Brief explanation of the calculation type
- **Input File Walkthrough** - Detailed explanation of input parameters
- **Example Input Files** - Ready-to-use input files
- **Running the Calculation** - Commands and execution instructions
- **Output Analysis** - How to interpret and visualize results
- **Common Issues** - Troubleshooting tips

## 🔗 General Resources

- [Common Concepts](./docs/common-concepts.md) - DFT basics applicable to all codes
- [Pseudopotentials](./docs/pseudopotentials.md) - Guide to pseudopotential selection
- [Convergence Testing](./docs/convergence.md) - k-point and cutoff convergence
- [Visualization Tools](./docs/visualization.md) - Tools for analyzing and plotting results

## 📝 Example Systems

All tutorials use Silicon (Si) as the primary example system for consistency, with additional examples for:
- Bulk metals (Al, Cu)
- Simple molecules (H₂O)
- 2D materials (graphene)

## 🛠️ Prerequisites

- Basic understanding of quantum mechanics and solid-state physics
- Linux/Unix environment (or WSL on Windows)
- Python 3.x with NumPy, Matplotlib (for post-processing)
- Respective DFT code installed

## 📦 Installation

Clone this repository:
```bash
git clone <repository-url>
cd dft
```

Navigate to the specific code directory for installation instructions.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues or pull requests to improve these tutorials.

## 📄 License

This repository is provided for educational purposes. Please cite the respective DFT codes in your research.

## 🔍 Additional Resources

- [Quantum ESPRESSO Official Documentation](https://www.quantum-espresso.org/)
- [ABINIT Official Documentation](https://www.abinit.org/)
- [SIESTA Official Documentation](https://siesta-project.org/)
- [CP2K Official Documentation](https://www.cp2k.org/)

---

*Last updated: November 2025*
