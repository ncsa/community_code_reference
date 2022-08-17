AMBER
=====

A. Description
~~~~~~~~~~~~~~

AMBER is a package of molecular simulation programs that stands for
Assisted Model Building with Energy Refinement. It represents a set of
classical molecula mechanics force fields for the simulation of
biomolecules. See AMBER home page http://ambermd.org for additional
information.

B. How to obtain AMBER
~~~~~~~~~~~~~~~~~~~~~~

AMBER is a licensed package. See
`instructions <http://ambermd.org/#obtain>`__ for obtaining the code.

C. How to build AMBER
~~~~~~~~~~~~~~~~~~~~~

Blue Waters does not provide precompiled AMBER binaries. The users are
expected to obtain their own copy of the code and install it for the
personal use.

C.1 Building CPU and GPU versions under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

module swap PrgEnv-cray PrgEnv-gnu

module add cray-netcdf

module add cray-hdf5

module add cray-fftw

module add bwpy

module add bwpy-mpi

module add cudatoolkit # only for Amber16 or newer

module add cudatoolkit/6.5.14-1.0502.9836.8.1 # only for Amber14 GPU

export AMBERHOME=$HOME/amber

export CUDA_HOME=$CRAY_CUDATOOLKIT_DIR

export CRAYPE_LINK_TYPE=dynamic

export CRAY_ADD_RPATH=yes

export PYTHON=`which python\`

export CC=cc # for Amber18 or newer

export CXX=CC # for Amber18 or newer

export FC=ftn # for Amber18 or newer

export MPICC=cc # for Amber18 or newer

export MPICXX=CC # for Amber18 or newer

export MPIFC=ftn # for Amber18 or newer

exporty MPIF90=ftn # for Amber18 or newer

cd $AMBERHOME

# build serial GPU version

make clean

::

       find . -name "*.a" -exec rm {} \;
   find . -name "*.so" -exec rm {} \;
   find . -name "*.so.*" -exec rm {} \;
   find . -name netcdf.mod -exec rm {} \;
   export PYTHON=`which python`


./configure -nomtkpp -crayxt5 -noX11 --with-python $PYTHON --with-netcdf
$NETCDF_DIR -cuda gnu

or

./configure -noX11 --with-python $PYTHON --with-netcdf $NETCDF_DIR -cuda
gnu # for Amber18 or newer

make install

# build parallel GPU version

make clean

./configure -nomtkpp -crayxt5 -noX11 --with-python $PYTHON --with-netcdf
$NETCDF_DIR -cuda -mpi gnu

or

./configure -noX11 --with-python $PYTHON --with-netcdf $NETCDF_DIR -cuda
-mpi gnu # for Amber18 or newer

make install

# build serial CPU version

make clean

./configure -nomtkpp -crayxt5 -noX11 --with-python $PYTHON --with-netcdf
$NETCDF_DIR gnu

or

./configure -noX11 --with-python $PYTHON --with-netcdf $NETCDF_DIR gnu #
for Amber18 or newer

make install

# build parallel CPU version

make clean

./configure -nomtkpp -crayxt5 -noX11 --with-python $PYTHON --with-netcdf
$NETCDF_DIR -mpi gnu

or

./configure -noX11 --with-python $PYTHON --with-netcdf $NETCDF_DIR -mpi
gnu # for Amber18 or newer

make install

Successful compilation will create executables in $AMBERHOME/bin
directory, e.g. sander.MPI.

D. Sample test
~~~~~~~~~~~~~~

AMBER comes with an extensive suit of tests located in $AMBERHOME/test
directory. To make sure that the compiled code works, we will run one of
the supplied tests.

cd $AMBERHOME/test/ubiquitine

create run.pbs file having the following content:

#!/bin/bash

#PBS -l nodes=2:ppn=32:xe

#PBS -l walltime=00:05:00

#PBS -q debug

cd $PBS_O_WORKDIR

export AMBERHOME=/insert_absolute_path_to_amber14

aprun -n 32 -N 16 -d 2 $AMBERHOME/bin/sander.MPI -O -i mdin -p prmtop -c
inpcrd -o job.out

Submit the job

qsub run.pbs
