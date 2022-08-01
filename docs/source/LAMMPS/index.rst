LAMMPS
======

A. Description
~~~~~~~~~~~~~~

LAMMPS is a classical molecular dynamics code. It can be applied to
metals, semiconductors, biomolecules, polymers in all-atom or
coarse-grained models. The source code is distributed under the terms of
GPL by Sandia National Laboratories. See LAMMPS home page
http://lammps.sandia.gov/ for additional information.

B. How to download LAMMPS
~~~~~~~~~~~~~~~~~~~~~~~~~

There are several different mechanisms to download LAMMPS. See complete
`instructions <http://lammps.sandia.gov/download.html>`__ for
downloading the code. You can download a recent stable version from the
LAMMPS github repository https://github.com/lammps/lammps using wget.
For example, the October, 2020 stable release can be obtained by running

::

   wget https://github.com/lammps/lammps/archive/stable_29Oct2020.zip

C. How to build LAMMPS
~~~~~~~~~~~~~~~~~~~~~~

On Blue Waters, LAMMPS can be built under the GNU environment. Building
under other compiler environments may be possible. Please contact
help+bw@ncsa.illinois.edu if you need to use another environment.

C.1 Building under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   module swap PrgEnv-cray PrgEnv-gnu
   module add fftw
   module add cmake/3.17.3
   mkdir ~/lammps
   cd ~/lammps
   wget https://github.com/lammps/lammps/archive/stable_29Oct2020.zip
   unzip stable_29Oct2020.zip
   cd lammps-stable_29Oct2020
   mkdir build
   cd build
   cmake -D CMAKE_CXX_COMPILER=CC -D CMAKE_C_COMPILER=cc -D CMAKE_Fortran_COMPILER=ftn \
   -D CMAKE_INSTALL_PREFIX=$HOME/lammps -D PKG_MOLECULE=yes ../cmake 
   make
   make install

D. Sample test
~~~~~~~~~~~~~~

LAMMPS comes with an extensive suit of tests. To make sure that the
compiled code works, we wil run one of the supplied tests.

::

   cd ../examples/micelle

Create run.pbs file having the following content:

::

   #!/bin/bash
   #PBS -l nodes=1:ppn=32:xe
   #PBS -l walltime=00:05:00
   #PBS -q debug
   cd $PBS_O_WORKDIR
   export OMP_NUM_THREADS=8
   aprun -n 4 -N 4 -d 8 $HOME/lammps/bin/lmp -in in.micelle -log job.out -screen none

Submit the job

::

   qsub run.pbs

Open the file job.out with text editor and make sure that line "Loop
time of 0.121433 on 4 procs for 1000 steps with 1200 atoms" is present
near the end of the file. It indicates that the job performed 1,000
dynamics steps on 4 processors. Timing of 0.121433 will be different on
different machines and may somewhat vary from run to run, which is
normal.

E. How to download older versions of LAMMPS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are several different mechanisms to download older versions of
LAMMPS. See complete
`instructions <http://lammps.sandia.gov/download.html>`__ for
downloading the code. To download a stable version of the code to local
directory mylammps by using svn tool, use the following command

::

   svn co -r N svn://svn.lammps.org/lammps-ro/trunk mylammps

where N is SVN tag. For the recent stable version 1 Feb 2014, the tag N
is 11423. The LAMMPS web site supplies the list of a few stable
`versions <http://lammps.sandia.gov/bug.html>`__. To download the latest
version, use

::

   svn co svn://svn.lammps.org/lammps-ro/trunk mylammps

F. How to build LAMMPS (1 Feb 2014 version)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Cray XE6 platform, LAMMPS can be built under GNU or PGI environments.
Following instructions are tested on 1 Feb 2014 release.

F.1 Building under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   module swap PrgEnv-cray PrgEnv-gnu
   module add fftw
   cd mylammps/src
   cp /u/staff/anisimov/lammps/lammps-2014-Feb1-gnu/src/MAKE/Makefile.xe6 MAKE
   make yes-molecule
   make xe6

Successful compilation will create lmp_xe6 binary file. A few comments
should be made. Althought the February 1, 2014 release comes with
Makefile.xe6 for Cray XE6 pltaform included into the distribution, that
Makefile.xe6 is not functional. We made following changes to correct it:

::

   CCFLAGS = -O2 -funroll-loops -fstrict-aliasing -Wall -W -Wno-uninitialized
   LIB = FFT_INC = -DFFT_FFTW3
   FFT_PATH = -L$(FFTWDIR)/lib
   FFT_LIB = -lfftw3

The compilation was conducted under the following environment:

::

   PrgEnv-gnu/4.2.34
   gcc/4.8.2
   cray-mpich/6.2.0
   fftw/3.3.0.4

F.2 Building under PGI environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   module swap PrgEnv-cray PrgEnv-pgi
   module add fftw
   cd mylammps/src
   cp /u/staff/anisimov/lammps/lammps-2014-Feb1-pgi/src/MAKE/Makefile.xe6 MAKE
   make yes-molecule
   make xe6

Successful compilation will create lmp_xe6 binary file. Again, as in the
previous case, Makefile.xe6 needs a few corrections:

::

   CCFLAGS = -fastsse
   LIB = 
   MPI_INC = 
   FFT_INC = -DFFT_FFTW3
   FFT_PATH = -L$(FFTWDIR)/lib
   FFT_LIB = -lfftw3

The compilation was conducted under the following environment:

::

   PrgEnv-pgi/4.2.34
   pgi/13.10.0
   cray-mpich/6.2.0
   fftw/3.3.0.4
