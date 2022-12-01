NWCHEM
======

A. Description
~~~~~~~~~~~~~~

NWChem is open source highly scalable high-performance electronic
structure computational package for computer simulation of molecular
systems, biomolecules, nanostructures, and solid-state systems. It
incorporates classical molecular dynamics, Hartree-Fock, Density
Functional Theory methods, and a wide range of post-Hartree-Fock methods
including Moeller-Plesset and Coupled Cluster techniques for the
treatment of electron correlation. See NWChem home page
`www.nwchem-sw.org <http://www.nwchem-sw.org/>`__ for additional
information.

B. How to obtain NWCHEM
~~~~~~~~~~~~~~~~~~~~~~~

NWCHEM is distributed under the terms of the Educational Community
License (ECL 2.0). See
`instructions <http://www.nwchem-sw.org/index.php/Download>`__ for
obtaining the code. The instrunctions in the section below are for
NWchem-6.6
(http://www.nwchem-sw.org/download.php?f=Nwchem-6.6.revision27746-src.2015-10-20.tar.gz).

C. How to build NWCHEM
~~~~~~~~~~~~~~~~~~~~~~

Blue Waters does not provide precompiled NWCHEM binaries. The users are
expected to obtain their own copy of the code and install it for the
personal use.

C.1 Building CPU version under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   cd $HOME
   # download Nwchem 6.6 source code using the link in section B above, or:
   wget -O Nwchem-6.6.revision27746-src.2015-10-20.tar.gz http://www.nwchem-sw.org/download.php?f=Nwchem-6.6.revision27746-src.2015-10-20.tar.gz
   tar zxvf Nwchem-6.6.revision27746-src.2015-10-20.tar.gz`
   # cd to "head" of install
   cd $HOME/nwchem-6.6
   `#revert to pevious PE for GNU
   module unload PrgEnv-cray/5.2.82
   module load PrgEnv-gnu/5.2.82-gcc.4.9.3``

``module add craype-hugepages8M``

``# specify your favourite modules``

``export NWCHEM_MODULES="nwdft nwpw driver stepper mp2_grad rimp2 ccsd property hessian vib tce"``

``# set env vars``

:literal:`export NWCHEM_TOP=`pwd\``

``export NWCHEM_MODULES="nwdft nwpw driver stepper mp2_grad rimp2 ccsd property hessian vib tce"``

``export FC=ftn``

``export CC=cc``

``export MSG_COMMS=MPI``

``export TARGET=LINUX64``

``export NWCHEM_TARGET=LINUX64``

``export ARMCI_NETWORK=MPI-PR``

``export HAS_BLAS=yes``

``export BLAS_OPT=''``

``export LIBMPI=''``

``export USE_MPI=y``

``export USE_MPIF=y``

``export USE_MPIF4=y``

``export USE_64TO32=y``

``export MA_USE_ARMCI_MEM=y``

``export BLAS_SIZE=4``

``export LAPACK_SIZE=4``

``export SCALAPACK_SIZE=4``

``export SCALAPACK=-lsci_gnu``

``export BLASOPT=-lsci_gnu``

``export GA_DIR=ga-5-4``

``export USE_NOFSCHECK=TRUE``

``export USE_NOIO=TRUE``

``# build NWChem``

``cd $NWCHEM_TOP/src``

``# do this one only once, othewise it will break the code. do not do it when recompiling already converted code.``

``make 64_to_32``

``# build header``

``make nwchem_config``

``# build binary``

``make FC=ftn GA_DIR=ga-5-4``

Successful compilation will create static executable:
$HOME/nwchem-6.6/bin/LINUX64/nwchem.

``# go back up to install location``

``cd ../..``

``# get test input file``\ ```input.nw`` <https://bluewaters.ncsa.illinois.edu/liferay-content/image-gallery/content/input.nw>`__\ ``.``

``wget https://bluewaters.ncsa.illinois.edu/liferay-content/image-gallery/content/input.nw``

``# submit interactive job``

``qsub -q debug -I -l nodes=10:ppn=32:xe -l walltime=00:30:00``

Now wait for job to start

``# once job starts``

``module unload PrgEnv-cray/5.2.82``

``module load PrgEnv-gnu/5.2.82-gcc.4.9.3``

``module load craype-hugepages8M``

``# cd to where test case is``

``cd $PBS_O_WORKDIR/``

``aprun -n160 -N16 -d 2 ./nwchem-6.6/bin/LINUX64/nwchem input.nw > output.nw``

# check for results

grep " Total CCSD energy" output.nw

output.nw: Total CCSD energy: -541.280804058835315
