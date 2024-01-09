# gromacs-spica
Patch file to use the SPICA force field with GROMACS. 
This patch replaces the [bond-angle cross potential](https://manual.gromacs.org/current/reference-manual/functions/bonded-interactions.html#bond-angle-cross-term)
with the [SPICA angle potential](https://www.spica-ff.org/forcefield.html) that includes a correction LJ potential term
to prevent an implausibly bent structure in MD simulations.  

NOTE: After you apply this patch to gromacs source codes, **you can no longer use the bond-angle cross term (angle index = 4)** in gromacs programs built with the codes. In addition, **GPU acceleration in GROMACS does not work for MD simulations with SPICA**, because we must apply tabulated potentials for nonboneded interactions used in SPICA. 

# Prerequisites
Source code of GROMACS-2019.6 : gromacs-2019.6.tar.gz  
https://manual.gromacs.org/documentation/2019.6/download.html  
or  
Source code of GROMACS-5.1.5 : gromacs-5.1.5.tar.gz  
https://manual.gromacs.org/documentation/5.1.5/download.html  

# Installation  
Apply patch command in the top source directory  

    git clone git@github.com:SPICA-group/gromacs-SPICA.git  
    wget http://ftp.gromacs.org/pub/gromacs/gromacs-2019.6.tar.gz
    tar xvzf gromacs-2019.6.tar.gz  
    cd gromacs-2019.6/  
    patch -p2 < ../gromacs-SPICA/gromacs-2019.6-spica_angle.patch  

Then, build the codes by following the [GROMACS installation guide](https://manual.gromacs.org/current/install-guide/index.html). 

For reversed patch, excute the following command in the top source directory  

    cd gromacs-2019.6/  
    patch -p2 -R < ../gromacs-SPICA/gromacs-2019.6-spica_angle.patch  

# Usage example
Tutorials to prepare input files for CG-MD simulations with SPICA using LAMMPS are available in the [SPICA website](https://www.spica-ff.org/tutorial_protein.html).
For using GROMACS, we use a command of [spica-tools](https://github.com/SPICA-group/spica-tools) `cg_spica setup_gmx`, instead of `cg_spica setup_lmp` for LAMMPS, to generate topology, index, and parameter files.

For example, in [Step 2 of the tutorial for lipid membrane systems](https://www.spica-ff.org/tutorial_lipid2.html),
the procedure before using `cg_spica setup_lmp` will be the same for GROMACS simulations with SPICA. Instead of `cg_spica setup_lmp ...`,
the command-line to prepare some of GROMACS input files will be like:

    cg_spica setup_gmx DOPC.top 128 WAT.top 3000 spica_db.prm 

PDB files are not required for this command because the generation of GROMACS itp and index files by this command does not need information on system configuration.  
After excuting the command-line, you will obtain the following files:
* `topol.top` (System topology file)
* `toppar/`
  * `SPICA.itp` (parameter file)
  * `{Molecule1}.itp` (topology files for each molecule)
  * `{Molecule2}.itp`
  * `...`
* `CGindex.ndx` (GROMACS index file)
* `out.psf` (PSF file for visualization)

We use tabulated potentials for nonbonded interations applied in SPICA, namely, LJ12-4 and LJ9-6 potentials. See the [Force Field](https://www.spica-ff.org/forcefield.html) 
page in the SPICA website for more details of the energy function form.
Table files formatted following the [GROMACS manual for using tabulated potentials](https://manual.gromacs.org/current/reference-manual/special/tabulated-interaction-functions.html)
that are needed to use those potentials can be generated with the following command:

    cg_spica gen_gmxin -pdb final.pdb -ndx CGindex.ndx

An example GROMACS mdp file (`npt.mdp`, in default) for SPICA is also generated via this command. The output files will be as follows:
* `table_SOLW_SOLW.xvg` (water-water, LJ12-4)
* `table_LJ124W_SOLW.xvg` (water-others, LJ12-4)
* `table.xvg` (other pairs, LJ9-6)
* `npt.mdp` 

The command-lines to execute the GROMACS commands `grompp` and `mdrun` will look like:

    gmx_mpi grompp -f npt.mdp -c final.pdb -p topol.top -n CGindex.ndx -o npt.tpr -maxwarn 5
    gmx_mpi mdrun -v -deffnm npt -table table.xvg


# Authors

Yusuke Miyazaki
