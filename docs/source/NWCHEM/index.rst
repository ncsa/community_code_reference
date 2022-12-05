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
   tar zxvf Nwchem-6.6.revision27746-src.2015-10-20.tar.gz
   
   # cd to "head" of install
   cd $HOME/nwchem-6.6
   
   #revert to pevious PE for GNU
   module unload PrgEnv-cray/5.2.82
   module load PrgEnv-gnu/5.2.82-gcc.4.9.3
   module add craype-hugepages8M
   
   # specify your favourite modules
   export NWCHEM_MODULES="nwdft nwpw driver stepper mp2_grad rimp2 ccsd property hessian vib tce"
   
   # set env vars
   export NWCHEM_TOP=`pwd`
   export NWCHEM_MODULES="nwdft nwpw driver stepper mp2_grad rimp2 ccsd property hessian vib tce"
   export FC=ftn
   export CC=cc
   export MSG_COMMS=MPI
   export TARGET=LINUX64
   export NWCHEM_TARGET=LINUX64
   export ARMCI_NETWORK=MPI-PR
   export HAS_BLAS=yes
   export BLAS_OPT=''
   export LIBMPI=''
   export USE_MPI=y
   export USE_MPIF=y
   export USE_MPIF4=y
   export USE_64TO32=y
   export MA_USE_ARMCI_MEM=y
   export BLAS_SIZE=4
   export LAPACK_SIZE=4
   export SCALAPACK_SIZE=4
   export SCALAPACK=-lsci_gnu
   export BLASOPT=-lsci_gnu
   export GA_DIR=ga-5-4
   export USE_NOFSCHECK=TRUE
   export USE_NOIO=TRUE

   # build NWChem
   cd $NWCHEM_TOP/src

   # do this one only once, othewise it will break the code. do not do it when recompiling already converted code.
   make 64_to_32

   # build header
   make nwchem_config

   # build binary
   make FC=ftn GA_DIR=ga-5-4

Successful compilation will create static executable: $HOME/nwchem-6.6/bin/LINUX64/nwchem.

::

   # go back up to install location
   cd ../..

   # use a heredoc to create the test input file input.nw
   cat << EOF > input.nw
   start guanine
   memory total 2000 MB  global 1500 MB

   title "guanine molecule MP2/6-311++G** optimized"

   geometry
   H     2.53990943     1.81612952    -0.03603309
   N     1.53491517     1.70711802    -0.01439495
   C     0.58326878     2.70125519    -0.01071739
   N    -0.65372605     2.23059061     0.01237075
   C    -0.48791625     0.86409028     0.03207422
   C    -1.47741654    -0.18802307    -0.00084033
   N    -0.81292365    -1.45618262     0.03459304
   C     0.54207616    -1.67606401     0.01606196
   N     1.43956105    -0.72422698     0.01693488
   C     0.86318483     0.51291841    -0.00091378
   H     0.85261439     3.74864971    -0.02225421
   O    -2.69273986    -0.13623948    -0.04504172
   N     0.93784894    -3.00204830    -0.06751858
   H     1.93349427    -3.10150657     0.08438005
   H     0.39727060    -3.64903707     0.49191052
   H    -1.43645739    -2.24608584    -0.09555577
   end

   basis spherical
     * library  6-311++G**
   end

   scf
     noprint "final vectors analysis"
   end

   ccsd
     freeze core atomic
     maxiter 50
   #  thresh 1000.0d0
   end

   #set ccsd:niter 10
   set ccsd:useinmemst2 T
   #set ccsd:usediskst2  F
   set ccsd:st2parallel T

   task ccsd energy
   EOF

   # submit interactive job
   qsub -q debug -I -l nodes=10:ppn=32:xe -l walltime=00:30:00

Now wait for job to start

::

   # once job starts
   module unload PrgEnv-cray/5.2.82
   module load PrgEnv-gnu/5.2.82-gcc.4.9.3
   module load craype-hugepages8M
   
   # cd to where test case is
   cd $PBS_O_WORKDIR/
   aprun -n160 -N16 -d 2 ./nwchem-6.6/bin/LINUX64/nwchem input.nw > output.nw

check for results

::

   grep " Total CCSD energy" output.nw

output.nw: Total CCSD energy: -541.280804058835315
