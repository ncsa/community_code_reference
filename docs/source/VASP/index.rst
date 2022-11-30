VASP
====

A. Description
~~~~~~~~~~~~~~

VASP is a package for performing ab initio quantum mechanical molecular
dynamics. It offers a wide range of Density Functional Theory (DFT)
methods along with a number of electron correlation methods, and
specializes in material simulation. VASP code is developed and
comercially distributed by VASP Group at the University of Vienna. See
VASP home page http://www.vasp.at for additional information.

B. How to obtain VASP
~~~~~~~~~~~~~~~~~~~~~

Due to commercial nature of VASP package, Blue Waters does not provide
precompiled binaries. However, help on compiling VASP package on Blue
Waters will be provided to those users who already have the VASP
license. The current VASP licensee may obtain a copy of VASP that
includes GPU support.

C. How to build VASP
~~~~~~~~~~~~~~~~~~~~

C.0 Building VASP 6.2
~~~~~~~~~~~~~~~~~~~~~

On Blue Waters, VASP 6.2 has been built with the Gnu programming
environment.

First, start a new shell, load some modules, and extract the software
package:

::

   $ module swap PrgEnv-cray PrgEnv-gnu
   $ module add fftw
   $ module add cudatoolkit
   $ tar zxvf vasp.6.2.0.tgz
   $ cd vasp.6.2.0

Put makefile.include in this directory.

::

   $ cat << EOF > makefile.include
   # Precompiler options
   CPP_OPTIONS= -DHOST=\"LinuxGNU\" \
                -DMPI -DMPI_BLOCK=8000 -Duse_collective \
                -DscaLAPACK \
                -DCACHE_SIZE=4000 \
                -Davoidalloc \
                -Dvasp6 \
                -Duse_bse_te \
                -Dtbdyn \
                -Dfock_dblbuf

   CPP        = cc -E -P -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

   FC         = ftn
   FCL        = ftn

   FREE       = -ffree-form -ffree-line-length-none

   FFLAGS     = -w -march=native
   OFLAG      = -O2
   OFLAG_IN   = $(OFLAG)
   DEBUG      = -O0

   #BLAS       = -L/opt/gnu/openblas-r0.3.8.dev-GNU-5.4.0/lib -lopenblas
   LAPACK     =
   BLACS      =
   #SCALAPACK  = -L/opt/gnu/scalapack-2.0.2-ompi-3.1.4-GNU-5.4.0 -lscalapack $(BLACS)

   LLIBS      = $(SCALAPACK) $(LAPACK) $(BLAS)

   FFTW       ?= $(FFTW_DIR)/..
   LLIBS      += -L$(FFTW_DIR) -lfftw3
   INCS       = -I$(FFTW_INC)

   OBJECTS    = fftmpiw.o fftmpi_map.o  fftw3d.o  fft3dlib.o

   OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
   OBJECTS_O2 += fft3dlib.o

   # For what used to be vasp.5.lib
   CPP_LIB    = $(CPP)
   FC_LIB     = $(FC)
   CC_LIB     = cc
   CFLAGS_LIB = -O
   FFLAGS_LIB = -O1
   FREE_LIB   = $(FREE)

   OBJECTS_LIB= linpack_double.o getshmem.o

   # For the parser library
   CXX_PARS   = CC
   LLIBS      += -lstdc++

   # Normally no need to change this
   SRCDIR     = ../../src
   BINDIR     = ../../bin

   #================================================
   # GPU Stuff

   CPP_GPU    = -DCUDA_GPU -DRPROMU_CPROJ_OVERLAP -DCUFFT_MIN=28 -UscaLAPACK -Ufock_dblbuf # -DUSE_PINNED_MEMORY

   OBJECTS_GPU= fftmpiw.o fftmpi_map.o fft3dlib.o fftw3d_gpu.o fftmpiw_gpu.o

   CC         = cc
   CXX        = CC
   CFLAGS     = -fPIC -DADD_ -openmp -DMAGMA_WITH_MKL -DMAGMA_SETAFFINITY -DGPUSHMEM=300 -DHAVE_CUBLAS

   # Minimal requirement is CUDA >= 10.X. For "sm_80" you need CUDA >= 11.X.
   CUDA_ROOT  ?= $(CUDATOOLKIT_HOME)
   NVCC       := $(CUDA_ROOT)/bin/nvcc
   CUDA_LIB   := -L$(CUDA_ROOT)/lib64 -lnvToolsExt -lcudart -lcuda -lcufft -lcublas

   #GENCODE_ARCH    := -gencode=arch=compute_60,code=\"sm_60,compute_60\" \
   #                   -gencode=arch=compute_70,code=\"sm_70,compute_70\" \
   #                   -gencode=arch=compute_80,code=\"sm_80,compute_80\"

   GENCODE_ARCH    := -gencode=arch=compute_60,code=\"sm_60,compute_60\" \
                      -gencode=arch=compute_70,code=\"sm_70,compute_70\"

   MPI_INC    = $(MPICH_DIR)/include
   EOF

Next, run the compilation.

::

   $ cd src/lib
   $ ln -s ../../makefile.include .
   $ make
   $ cd ../..
   $ make

C.1 Building VASP 5.4.4
~~~~~~~~~~~~~~~~~~~~~~~

On Blue Waters Cray XE6 platform, VASP can be built under Cray or Intel
programming environments. The Cray compiler currently provides the best
performing code on Blue Waters.

This only documents how to use the Cray environment, using the Intel
compiler may work by adapting the methodology documented in the next
section.

First ensure that the expected compiler modules are loaded:

::

   module unload PrgEnv-cray
   module unload PrgEnv-gnu
   module unload PrgEnv-intel
   module unload PrgEnv-pgi

   module load PrgEnv-cray/5.2.82
   module load fftw/3.3.4.10
   module load cudatoolkit/9.1.85_3.10-1.0502.df1cc54.3.1

and verify your environment to look like this:

::

   module list
   Currently Loaded Modulefiles:
     1) modules/3.2.10.4                            18) darshan/3.1.3
     2) eswrap/1.3.3-1.020200.1280.0                19) cce/8.7.7
     3) craype-interlagos                           20) cray-libsci/18.12.1
     4) craype-network-gemini                       21) udreg/2.3.2-1.0502.10518.2.17.gem
     5) craype/2.5.16                               22) ugni/6.0-1.0502.10863.8.28.gem
     6) cray-mpich/7.7.4                            23) pmi/5.0.14
     7) torque/6.0.4                                24) dmapp/7.0.1-1.0502.11080.8.74.gem
     8) moab/9.1.2-sles11                           25) gni-headers/4.0-1.0502.10859.7.8.gem
     9) openssh/7.5p1                               26) xpmem/0.1-2.0502.64982.5.3.gem
    10) xalt/0.7.6.local                            27) dvs/2.5_0.9.0-1.0502.2188.1.113.gem
    11) scripts                                     28) alps/5.2.4-2.0502.9774.31.12.gem
    12) OpenSSL/1.0.2m                              29) rca/1.0.0-2.0502.60530.1.63.gem
    13) cURL/7.59.0                                 30) atp/2.0.4
    14) git/2.17.0                                  31) PrgEnv-cray/5.2.82
    15) wget/1.19.4                                 32) fftw/3.3.4.10
    16) user-paths                                  33) cudatoolkit/9.1.85_3.10-1.0502.df1cc54.3.1
    17) gnuplot/5.0.5

Next untar and build the vasp library tarball using the prepared
Makefile adapted for Blue Waters:

::

   tar xf vasp.5.lib.tar.gz
   pushd vasp.5.lib
   wget "https://bluewaters.ncsa.illinois.edu/liferay-content/document-library/VASP%205.3.3%20Cray%20Makefile%20for%20libraries" -O Makefile
   make
   popd

Untar the VASP source code:

::

   tar xf vasp.5.4.4.tar.gz

Patch a bug in VASP that prevents compilation:

::

   pushd vasp.5.4.4
   patch -p0 <<"EOF"
   --- src/mgrid.F    2017-04-20 04:03:58.000000000 -0500
   +++ src/mgrid.F    2020-02-21 11:19:11.000000000 -0600
   @@ -102,8 +102,8 @@
         ! G vectors on the 3D FFT grid
         ! IND_IN_SPHERE stores the PW indices (within the sphere) with N1=0
         ! NINDPWCONJG stores the position where the conjugated coefficient needs to be stored
   -     REAL(q), POINTER:: NINDPWCONJG(:)
   -     REAL(q), POINTER:: IND_IN_SPHERE(:)
   +     INTEGER, POINTER:: NINDPWCONJG(:)
   +     INTEGER, POINTER:: IND_IN_SPHERE(:)
      END TYPE grid_3d
      ! comments:
      ! REAL2CPLX determines wether a real to complex FFT is use
   EOF

Write a new makefile fragment for Blue Waters:

::

   cat >./makefile.include <<"EOF"
   # Precompiler options
   CPP_OPTIONS= -DHOST=\"CrayXE6\" \
            -Dkind8 -DNGZhalf -DCACHE_SIZE=16000 -DPGF90 -Davoidalloc \
            -DRPROMU_DGEMV -DIFC \
            -DMPI -DscaLAPACK  -DMPI_BLOCK=100000 -Duse_collective -Drandom_array

   CPP        = cpp --traditional -P $(CPP_OPTIONS) $*$(FUFFIX) $*$(SUFFIX)

   FC         = ftn
   FCL        = ftn

   FREE       = -ffree

   FFLAGS     = -dC -rmo -emEb -hnetwork=gemini
   OFLAG      = -O2
   OFLAG_IN   = $(OFLAG)
   DEBUG      = -O0

   LIBDIR     = /opt/gfortran/libs/
   BLAS       =
   LAPACK     =
   BLACS      =
   SCALAPACK  =

   LLIBS      = -ldmy -lsci_cray_mp  -ldmapp

   FFTW       = $(FFTW_DIR)/..
   LLIBS      += -L$(FFTW)/lib -lfftw3f
   INCS       = -I$(FFTW)/include

   OBJECTS    = fftmpiw.o fftmpi_map.o  fftw3d.o  fft3dlib.o

   OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
   OBJECTS_O2 += fft3dlib.o

   # For what used to be vasp.5.lib
   CPP_LIB    = $(CPP)
   FC_LIB     = $(FC)
   CC_LIB     = cc
   CFLAGS_LIB = -O
   FFLAGS_LIB = -O1
   FREE_LIB   = $(FREE)

   OBJECTS_LIB= linpack_double.o getshmem.o

   # For the parser library
   CXX_PARS   = CC

   LIBS       += parser
   LLIBS      += -Lparser -lparser -lstdc++

   # Normally no need to change this
   SRCDIR     = ../../src
   BINDIR     = ../../bin

   #================================================
   # GPU Stuff

   CPP_GPU    = -DCUDA_GPU -DRPROMU_CPROJ_OVERLAP -DCUFFT_MIN=28 -UscaLAPACK # -DUSE_PINNED_MEMORY

   OBJECTS_GPU= fftmpiw.o fftmpi_map.o fft3dlib.o fftw3d_gpu.o fftmpiw_gpu.o

   CC         = gcc
   CXX        = g++
   CFLAGS     = -fPIC -DADD_ -openmp -DMAGMA_SETAFFINITY -DGPUSHMEM=300 -DHAVE_CUBLAS

   CUDA_ROOT  ?= $(CUDATOOLKIT_HOME)
   NVCC       := $(CUDA_ROOT)/bin/nvcc
   CUDA_LIB   := -L$(CUDA_ROOT)/lib64 -lnvToolsExt -lcudart -lcuda -lcufft -lcublas

   GENCODE_ARCH    := -gencode=arch=compute_35,code=\"sm_35,compute_35\"

   MPI_INC    = $(MPICH_DIR)/include
   EOF

Build VASP:

::

   make
   popd

This was last tested 2020-Feb-21 by rhaas.

C.2 Building VASP 5.3.3
~~~~~~~~~~~~~~~~~~~~~~~

C.2.1 Building under Cray environment

Upon opening a new terminal on Blue Waters login node, Cray programming
environment is loaded by default. At the time of writing this document,
these are PrgEnv-cray/4.2.34, cce/8.3.3, cray-libsci/13.0.1, and
cray-mpich/7.0.3. Additional module to be loaded is fftw. Tests showed,
that Cray programming environment is fully compatible with VASP 5.x, and
that compatibility will presumably remain. At the present moment, using
the latest (default) version of modules should be fine, and one can
simply resort to using the default modules without worrying about their
particular version number. Just for the record, loading default version
of module fftw gives fftw/3.3.4.0 at this particular moment.

Following are step-by-step instructions for the freshly opened terminal
on Blue Waters login node:

::

   module add fftw
   tar zxvf vasp.5.lib.tar.gz
   tar zxvf vasp.5.3.3.tar.gz

   cd vasp.5.lib
   wget https://bluewaters.ncsa.illinois.edu/liferay-content/document-library/VASP%205.3.3%20Cray%20Makefile%20for%20libraries -O Makefile
   make

   cd ../vasp.5.3
   wget https://bluewaters.ncsa.illinois.edu/liferay-content/document-library/VASP%205.3.3%20Cray%20Makefile%20for%20source%20code -O Makefile
   make

Successful compilation will create a vasp binary file in the source code
directory.

C.2.2 Building under Intel environment

In a freshly opened terminal, we will swap the default Cray programming
environment to Intel. Since we intend to use Intel MKL, it is necessary
to unload cray-libsci module.

::

   module swap PrgEnv-cray PrgEnv-intel
   module unload cray-libsci

At the time of writing this document, the programming environment get
loaded with PrgEnv-intel/4.2.34, intel/15.0.090, and cray-mpich/7.0.3.
The MKL library is the one that comes with this particular version of
Intel compiler. We will need to compile FFTW interface comping with MKL
library.

::

   cp -r $MKLROOT/interfaces/fftw3xf/ $PWD
   cd fftw3xf
   make libintel64

The compilation will create libfftw3xf_intel.a in directory fftw3x/

::

   cd ..
   tar zxvf vasp.5.lib.tar.gz
   tar zxvf vasp.5.3.3.tar.gz

   cd vasp.5.lib
   wget https://bluewaters.ncsa.illinois.edu/liferay-content/document-library/VASP%205.3.3%20Intel%20Makefile%20for%20libraries -O Makefile
   make

   cd ../vasp.5.3
   wget https://bluewaters.ncsa.illinois.edu/liferay-content/document-library/VASP%205.3.3%20Intel%20Makefile%20for%20source%20code -O Makefile
   make

Successful compilation will create a vasp binary file in the source code
directory.

D. Sample test
~~~~~~~~~~~~~~

VASP tests can be downloaded from the public repository.

::

   git clone https://github.com/egplar/vasptest

The tests come without the POTCAR library. The README.md file inside
vasptest/ directory explains how to get that library.

Following is sample script to run vasp jobs on Blue Waters. Create
run.pbs file having the following content:

::

   #!/bin/bash
   #PBS -l nodes=4:ppn=32:xe
   #PBS -l walltime=00:30:00
   #PBS -q debug

   cd $PBS_O_WORKDIR
   aprun -n 64 -N 16 -d 2 ../vasp.5.3/vasp > job.out

**Submit the job**

::

   qsub run.pbs

Note the use of 16 cores per node, as instructed by aprun commands -N16
-d2. Using 32 cores per node leads to suboptimal performance.
