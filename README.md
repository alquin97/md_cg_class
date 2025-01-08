# Coarse-Grained Molecular Dynamics

This repository is aimed at showing how to build coarse-grained membrane-embedded systems with CHARMM-GUI, run simulations with GROMACS and perform a very basic protomer contact analysis.

## Distribution

Under [theory/](theory/) you have the theoretical presentation to the topic. Under [practical/](practical/) you have a protocol with the different steps to be followed. The protocol covers the following how-tos.

- Build a coarse-grained membrane-embedded system with CHARMM-GUI.
- Run coarse-grained simulations with GROMACS.
- Analyze the residue-residue contacts between two protomers in a trajectory.

## Setting up

### Installing GROMACS

Check if GROMACS is installed in your system by typing `gmx help` or `which gmx` in your shell. If there's no prompt or the software is not installed, do the following.
```
sudo apt install gromacs
```
> Note: This version of GROMACS does not have GPU support.
