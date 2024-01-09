# gromacs-spica
This patch file facilitates the use of the SPICA force field with GROMACS. 
This patch replaces the [bond-angle cross potential](https://manual.gromacs.org/current/reference-manual/functions/bonded-interactions.html#bond-angle-cross-term)
with the [SPICA angle potential](https://www.spica-ff.org/forcefield.html), incorporating a correction LJ potential term
to prevent an implausibly bent structure in MD simulations.  

**NOTE**: Upon applying this patch to GROMACS source codes, you can no longer use the bond-angle cross term (angle index = 4) in GROMACS programs built with the codes. Additionaly, GPU acceleration in GROMACS does not function for MD simulations with SPICA due to the necessity of applying tabulated potentials for nonboneded interactions used in SPICA. 

# Prerequisites
Source code of GROMACS-2019.6 : gromacs-2019.6.tar.gz
https://manual.gromacs.org/documentation/2019.6/download.html  
or  
Source code of GROMACS-5.1.5 : gromacs-5.1.5.tar.gz  
https://manual.gromacs.org/documentation/5.1.5/download.html  

# Installation  
Apply patch command in the top source directory  

```bash
git clone git@github.com:SPICA-group/gromacs-SPICA.git
wget http://ftp.gromacs.org/pub/gromacs/gromacs-2019.6.tar.gz
tar xvzf gromacs-2019.6.tar.gz  
cd gromacs-2019.6/  
patch -p2 < ../gromacs-SPICA/gromacs-2019.6-spica_angle.patch  
```
Then, build the codes by following the [GROMACS installation guide](https://manual.gromacs.org/current/install-guide/index.html). 

For reversing the patch, excute the following command in the top source directory  
```bash
cd gromacs-2019.6/  
patch -p2 -R < ../gromacs-SPICA/gromacs-2019.6-spica_angle.patch  
```
# Usage example
Tutorials for preparing input files for CG-MD simulations with SPICA using LAMMPS are available on the [SPICA website](https://www.spica-ff.org/tutorial_protein.html).
For GROMACS usage, employ the command `cg_spica setup_gmx` from [spica-tools](https://github.com/SPICA-group/spica-tools), 
instead of `cg_spica setup_lmp` for LAMMPS, to generate topology, index, and parameter files.

For instance, in [Step 2 of the tutorial for lipid membrane systems](https://www.spica-ff.org/tutorial_lipid2.html),
the process before using `cg_spica setup_lmp` will be the identical for GROMACS simulations with SPICA. 
The command-line to prepare some GROMACS input files will be like:
```bash
cg_spica setup_gmx DOPC.top 128 WAT.top 3000 spica_db.prm 
```
PDB files are unnecessary for this command because the generation of GROMACS itp and index files by this command does not require system configuration information.  
Upon excuting the command-line, the generated files will include:
* `topol.top` (system topology file)
* `toppar/`
  * `SPICA.itp` (parameter file)
  * `{Molecule1}.itp` (topology files for each molecule)
  * `{Molecule2}.itp`
  * `...`
* `CGindex.ndx` (index file)
* `out.psf` (PSF file for visualization)

Tabulated potentials for nonbonded interations applied in SPICA, namely, LJ12-4 and LJ9-6 potentials (see the [Force Field](https://www.spica-ff.org/forcefield.html) page) require table files formatted following the 
[GROMACS manual for using tabulated potentials](https://manual.gromacs.org/current/reference-manual/special/tabulated-interaction-functions.html).
Generate these files with the following command:
```bash
cg_spica gen_gmxin -pdb final.pdb -ndx CGindex.ndx
```
An example GROMACS mdp file (`npt.mdp`, by default) for SPICA will be also generated. The output files include:
* `table_SOLW_SOLW.xvg` (water-water, LJ12-4)
* `table_LJ124W_SOLW.xvg` (water-others, LJ12-4)
* `table.xvg` (other pairs, LJ9-6)
* `npt.mdp` 

The command-lines to execute the two commands, `grompp` and `mdrun`, of **GROMACS patched for SPICA** will look like:
```bash
gmx_mpi grompp -f npt.mdp -c final.pdb -p topol.top -n CGindex.ndx -o npt.tpr -maxwarn 5
gmx_mpi mdrun -v -deffnm npt -table table.xvg
```

# Authors

Yusuke Miyazaki
