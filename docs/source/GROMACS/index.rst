GROMACS
=======

A. Description
~~~~~~~~~~~~~~

GROMACS is a popular classical molecular dynamics code, which is
designed for computer simulation of proteins, lipids, and nucleic acids.
GROMACS source code is released under the terms of GNU Lesser General
Public License. See GROMACS home page
`http://www.gromacs.org/ <http://www.gromacs.org>`__ for additional
information.

B. How to obtain GROMACS
~~~~~~~~~~~~~~~~~~~~~~~~

The following instructions assume the use of bash shell. To download a
stable version of the code following steps should be performed
(replacing gromacs-2016.4 with the desired version):

::

   export GROMACS=$HOME/gromacs
   export VERSION=gromacs-2016.4

   mkdir $GROMACS
   cd $GROMACS
   wget ftp://ftp.gromacs.org/pub/gromacs/${VERSION}.tar.gz
   tar zxvf ${VERSION}.tar.gz
   cd $VERSION

This will download and unpack GROMACS source code.

C. How to build GROMACS
~~~~~~~~~~~~~~~~~~~~~~~

On Blue Waters, GROMACS can be built under GNU programming environment.
See GROMACS
`manual <http://www.gromacs.org/Documentation/Installation_Instructions>`__
for additional insallation instructions.

C.1 Building the CPU-code under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Configure programming environment and set up environment variables

::

   module swap PrgEnv-cray PrgEnv-gnu
   module swap gcc gcc/6.3.0
   module add fftw
   module add boost
   module add cmake/3.17.3

   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes
   export CXX=CC
   export CC=cc
   export CMAKE_PREFIX_PATH=$FFTW_DIR/../
   export FLAGS="-dynamic -O3 -march=bdver1 -ftree-vectorize -ffast-math -funroll-loops"

   export INSTALL=$GROMACS/$VERSION/install-cpu
   mkdir $INSTALL

   export BUILD=$GROMACS/$VERSION/build-cpu
   mkdir $BUILD

- Run cmake. Make sure $GROMACS and $INSTALL are defined

::

   cd $BUILD

   cmake ../ -DGMX_MPI=ON -DGMX_OPENMP=ON -DGMX_GPU=OFF -DBUILD_SHARED_LIBS=OFF \
   -DGMX_PREFER_STATIC_LIBS=ON -DGMX_X11=OFF -DGMX_DOUBLE=OFF -DCMAKE_SKIP_RPATH=YES \
   -DCMAKE_INSTALL_PREFIX=$INSTALL -DCMAKE_C_FLAGS="$FLAGS" -DCMAKE_CXX_FLAGS="$FLAGS" \
   -DGMX_CPU_ACCELERATION="AVX_128_FMA"

- Compile and install

::

   make
   make install

Successful compilation will create bin, include, lib64, and share
directories under directory $INSTALL

The latest successful compilation was conducted under the following
environment:

::

   Currently Loaded Modulefiles:
     1) modules/3.2.10.4
     2) eswrap/1.3.3-1.020200.1280.0
     3) gcc/6.3.0
     4) craype-interlagos
     5) craype-network-gemini
     6) craype/2.5.16
     7) cray-mpich/7.7.4
     8) torque/6.0.4
     9) moab/9.1.2-sles11
    10) openssh/7.5p1
    11) xalt/0.7.6.local
    12) scripts
    13) OpenSSL/1.0.2m
    14) cURL/7.59.0
    15) git/2.17.0
    16) wget/1.19.4
    17) user-paths
    18) gnuplot/5.0.5
    19) darshan/3.1.3
    20) cray-libsci/18.12.1
    21) udreg/2.3.2-1.0502.10518.2.17.gem
    22) ugni/6.0-1.0502.10863.8.28.gem
    23) pmi/5.0.14
    24) dmapp/7.0.1-1.0502.11080.8.74.gem
    25) gni-headers/4.0-1.0502.10859.7.8.gem
    26) xpmem/0.1-2.0502.64982.5.3.gem
    27) dvs/2.5_0.9.0-1.0502.2188.1.113.gem
    28) alps/5.2.4-2.0502.9774.31.12.gem
    29) rca/1.0.0-2.0502.60530.1.63.gem
    30) atp/2.0.4
    31) PrgEnv-gnu/5.2.82
    32) fftw/3.3.4.10
    33) cmake/3.1.3
    34) boost/1.63.0

C.2 Building the GPU-code under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Configure programming environment and set up environment variables

::

   module swap PrgEnv-cray PrgEnv-gnu
   module swap gcc gcc/6.3.0
   module add fftw
   module add boost
   module add cudatoolkit
   module add cmake/3.17.3

   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes
   export CXX=CC
   export CC=cc
   export CMAKE_PREFIX_PATH=$FFTW_DIR/../
   export FLAGS="-dynamic -O3 -march=bdver1 -ftree-vectorize -ffast-math -funroll-loops"

   export INSTALL=$GROMACS/$VERSION/install-gpu
   mkdir $INSTALL

   export BUILD=$GROMACS/$VERSION/build-gpu
   mkdir $BUILD

- Run cmake. Make sure $GROMACS and $INSTALL are defined as above

::

   cd $BUILD

   cmake ../ -DGMX_MPI=ON -DGMX_OPENMP=ON -DGMX_GPU=ON \
   -DCUDA_TOOLKIT_ROOT_DIR=$CUDATOOLKIT_HOME -DBUILD_SHARED_LIBS=OFF \
   -DGMX_PREFER_STATIC_LIBS=ON -DGMX_X11=OFF -DGMX_DOUBLE=OFF -DCMAKE_SKIP_RPATH=YES \
   -DCMAKE_INSTALL_PREFIX=$INSTALL -DCMAKE_C_FLAGS="$FLAGS" -DCMAKE_CXX_FLAGS="$FLAGS" \
   -DGMX_SIMD="AVX_128_FMA" -DCUDA_HOST_COMPILER=$CRAYPE_DIR/bin/cc \
   -DCUDA_NVCC_FLAGS="-gencode;arch=compute_35,code=sm_35;-gencode;arch=compute_35,code=compute_35;-use_fast_math;"

- Compile and install

::

   make
   make install

Successful compilation will create bin, include, lib64, and share
directories under directory $INSTALL

The latest successful compilation was conducted under the following
environment:

::

   Currently Loaded Modulefiles:
     1) modules/3.2.10.4
     2) eswrap/1.3.3-1.020200.1280.0
     3) gcc/6.3.0
     4) craype-interlagos
     5) craype-network-gemini
     6) craype/2.5.16
     7) cray-mpich/7.7.4
     8) torque/6.0.4
     9) moab/9.1.2-sles11
    10) openssh/7.5p1
    11) xalt/0.7.6.local
    12) scripts
    13) OpenSSL/1.0.2m
    14) cURL/7.59.0
    15) git/2.17.0
    16) wget/1.19.4
    17) user-paths
    18) gnuplot/5.0.5
    19) darshan/3.1.3
    20) cray-libsci/18.12.1
    21) udreg/2.3.2-1.0502.10518.2.17.gem
    22) ugni/6.0-1.0502.10863.8.28.gem
    23) pmi/5.0.14
    24) dmapp/7.0.1-1.0502.11080.8.74.gem
    25) gni-headers/4.0-1.0502.10859.7.8.gem
    26) xpmem/0.1-2.0502.64982.5.3.gem
    27) dvs/2.5_0.9.0-1.0502.2188.1.113.gem
    28) alps/5.2.4-2.0502.9774.31.12.gem
    29) rca/1.0.0-2.0502.60530.1.63.gem
    30) atp/2.0.4
    31) PrgEnv-gnu/5.2.82
    32) fftw/3.3.4.10
    33) cmake/3.1.3
    34) boost/1.63.0
    35) cudatoolkit/9.1.85_3.10-1.0502.df1cc54.3.1

Sample run.pbs file for XK-nodes:

::

   #!/bin/bash
   #PBS -l nodes=2:ppn=16:xk
   #PBS -l walltime=00:05:00

   cd $PBS_O_WORKDIR
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GROMACS/$VERSION/install-gpu/lib64

   export OMP_NUM_THREADS=16
   aprun -n 2 -N 1 -d $OMP_NUM_THREADS $HOME/gromacs/gromacs-2016.4/install-gpu/bin/gmx mdrun ...

Submit the job:

::

   qsub run.pbs
