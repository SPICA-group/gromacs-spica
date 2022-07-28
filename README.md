# gromacs-SPICA
Patch file to use SPICA FF with GROMACS 

# Functions

This patch replaces the urey-bradley angle potentials with the SPICA (formerly called SDK) angle potential that includes a correction LJ potential term to prevent an implausibly bent structure in CG-MD (J. Phys. Chem. B, 2010, 114, 6836–6849).  

NOTE: After you apply this patch to gromacs source codes, YOU CAN NO LONGER USE THE UREY-BRADLEY ANGLE POTENTIAL (angle index = 5) in gromacs programs built with the codes.  

# Prerequisites

Source code of GROMACS-2019.6 : gromacs-2019.6.tar.gz  
https://manual.gromacs.org/documentation/2019.6/download.html

# Installation  
Apply patch command in the top source directory.  

    tar xvzf gromacs-2019.6.tar.gz  
    cd gromacs-2019.6/  
    patch -p2 < gromacs-2019.6-spica_angle.patch  

For reversed patch    

    patch -p2 -R < gromacs-2019.6-spica_angle.patch  


# Authors

Yusuke Miyazaki
