PETSc: Building Your Own
========================

Description
~~~~~~~~~~~

We recommend that whenever possible, you use the PETSc release provided
by the vendor (Cray). Please see the `PETSc
page <https://bluewaters.ncsa.illinois.edu/petsc>`__ for more
information on using the Cray PETSc build that is optimized for Cray
systems.

If you need to build the latest release or a specific version then
please follow the instructions below.

We recommend that you first remove the bwpy python package to avoid
PETSCs python build system from mixing in bwpy itself which makes the
runtime more complicated. See the end of the page for more information.

How to build PETSc
~~~~~~~~~~~~~~~~~~

Downloading PETSc from `the PETSc
webpage <http://www.mcs.anl.gov/petsc/download/index.html>`__,

Download a specific version of PETSc tarball, and extract the sources as
follows:

::

   % wget http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-3.7.4.tar.gz
   % tar –zxvf  petsc-3.7.4.tar.gz

To move to the PETSc directory use:

::

   % cd petsc-3.7.4

Downloading PETSc using Git (an alternative way)

To download the release use:

::

   % git clone -b maint https://bitbucket.org/petsc/petsc petsc

To move to the PETSc directory use:

::

   % cd petsc

To obtain new patches that have been added since your "git clone" or
last "git pull" use:

::

   % git pull 

Installing PETSc with PrgEnv-gnu

To start an interactive mode use:

::

   % qsub -I -l nodes=1:ppn=32:xe -l walltime=2:00:00

To update the programming environment on Blue Waters use:

::

   % module unload PrgEnv-cray
   % module load PrgEnv-gnu
   % module load cmake
   % export CRAYPE_LINK_TYPE=dynamic
   % export CRAY_ADD_RPATH=yes

To set up PETSC_DIR and PETSC_ARCH use:

::

   % export PETSC_DIR=[the PETSc directory]
   % export PETSC_ARCH=BW-gnu

*You can specify PETSC_ARCH with any character string. For each build,
you have to specify different PETSC_ARCH. See an example for another
build with PrgEnv-intel at the bottom.*

To configure PETSc with some external packages (e.g., superlu_dist,
mumps, hypre, spooles, ml, metis, and parmetis libraries) use:

::

   % cd ${PETSC_DIR}
   % ./configure \
     --with-cc=cc --with-cxx=CC --with-fc=ftn \
     --download-fblaslapack=1 --download-blacs=1 --download-metis=1 --download-parmetis=1 \
     --download-superlu_dist=1 --download-mumps=1 --download-scalapack=1 --download-hypre=1 \
     --download-spooles=1 --download-ml=1 --with-pic=1 --with-debugging=0 \
     --with-shared-libraries=1 COPTFLAGS=-O3 CXXOPTFLAGS=-O3 FOPTFLAGS=-O3 \
     --with-mpiexec="aprun -q" 

*This step may take around 40 to 60 minutes on Blue Waters due to
downloading and compiling external libraries.*

To build and check PETSc use:

::

   % make all 

::

   % make check 

You should see output similar to

::

   Running test examples to verify correct installation
   Using PETSC_DIR=/scratch/staff/gbauer/petsc/petsc-latest and PETSC_ARCH=BW-gnu
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with 1 MPI process
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with 2 MPI processes
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with hypre
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with mumps
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with superlu_dist
   C/C++ example src/snes/examples/tutorials/ex19 run successfully with ml
   Fortran example src/snes/examples/tutorials/ex5f run successfully with 1 MPI process

Installing another PETSc with PrgEnv-intel (optional)

To update the programming environment on Blue Waters use:

::

   % module unload PrgEnv-gnu
   % module load PrgEnv-intel

To update PETSC_ARCH use:

::

   % export PETSC_ARCH=BW-intel

To configure, build and test PETSc, repeat the corresponding steps
presented for PreEnv-gnu at the above.

How to use PETSc
~~~~~~~~~~~~~~~~

To set up the PETSc environment use:

::

   % module unload PrgEnv-cray
   % module load PrgEnv-gnu            (or, % module load PrgEnv-intel)
   % export CRAYPE_LINK_TYPE=dynamic
   % export CRAY_ADD_RPATH=yes
   % export PETSC_DIR=[the PETSc directory]
   % export PETSC_ARCH=BW-gnu          (or, % export PETSC_ARCH=BW-intel)

Examples
~~~~~~~~

Linear Poisson equation on a 2D grid

To compile and build the example use:

::

   % cd ${PETSC_DIR}/src/ksp/ksp/examples/tutorials
   % make ex50

To run the example use:

::

   % qsub –I –l nodes=2:ppn=32 –l walltime=00:30:00
   % cd $PBS_O_WORKDIR
   % aprun -n 32 -N 16 -S 4 -j 1 ./ex50  -da_grid_x 100 -da_grid_y 100 \
     -pc_type lu -pc_factor_mat_solver_package superlu_dist -ksp_monitor -ksp_view

Additional Information / References

-  PETSc Home Page: http://www.mcs.anl.gov/petsc/
-  PETSc Download: http://www.mcs.anl.gov/petsc/download/
-  PETSc Installation Guide:
   http://www.mcs.anl.gov/petsc/documentation/installation.html
-  PETSc Manual Pages:
   http://www.mcs.anl.gov/petsc/documentation/index.html
-  Cray PETSc on Blue Waters: https://bluewaters.ncsa.illinois.edu/petsc

If you need to build with bwpy in the build environment then you will
need to invoke:

::

   bwpy-environ

before make and add bwpy-environ to aprun commands as

::

   aprun -n 1 bwpy-environ -- ...
