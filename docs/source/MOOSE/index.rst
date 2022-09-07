MOOSE
=====

Description
-----------

The Multiphysics Object-Oriented Simulation Environment (MOOSE) is a
finite-element, multiphysics framework primarily developed by Idaho
National Laboratory. It provides a high-level interface to some of the
most sophisticated nonlinear solver technology. MOOSE presents a
straightforward API that aligns well with the real-world problems
scientists and engineers need to tackle.

Some of the capabilities of MOOSE are as follows:

-  Fully-coupled, fully-implicit multiphysics solver
-  Dimension independent physics
-  Automatically parallel (up to 100,000 cores!)
-  Modular development simplifies code reuse
-  Built-in mesh adaptivity
-  Continuous and Discontinuous Galerkin (DG)
-  Intuitive parallel multiscale solves
-  Dimension agnostic, parallel geometric search
-  Flexible, plugable graphical user interface
-  ~30 plugable interfaces allow specialization of every part of the
   solve
-  Physics modules providing general capability for solid mechanics,
   phase field modeling, Navier-Stokes, heat conduction and more.

How to build MOOSE
------------------

These instructions were adapted from the `official MOOSE build
instructions <https://mooseframework.org/getting_started/installation/manual_installation_gcc.html>`__
using gcc and MPICH. However please **do not follow those instructions**
since they instruct you to build your own gcc, PETSc and MPICH which
will (at best) give you a very slow build of MOOSE and (more likely) not
work at all.

Downloading MOOSE using Git
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To download the release use:

::

   mkdir ~/MOOSE_BUILD
   cd ~/MOOSE_BUILD
   git clone https://github.com/idaholab/moose.git
   cd moose
   git checkout master

Creating a file for module environments (e.g. ~/source_me_for_MOOSE)

::

   cat >~/source_me_for_MOOSE <<EOF
   module swap PrgEnv-cray PrgEnv-gnu
   module load cray-hdf5-parallel
   module load bwpy
   module load fftw
   module load cray-petsc
   module load cray-tpsl
   export CC=cc
   export CXX=CC
   export FC=ftn
   export EPYTHON=python2.7
   export CPATH="$CPATH:${BWPY_INCLUDE_PATH}:${BWPY_DIR}/usr/include"
   export LIBRARY_PATH="$LIBRARY_PATH:${BWPY_LIBRARY_PATH}:${BWPY_DIR}/usr/lib"
   export LDFLAGS="${LDFLAGS} -Wl,--rpath=${BWPY_LIBRARY_PATH}"
   export PETSCDIR=$PETSC_DIR
   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes
   EOF

Installing MOOSE
~~~~~~~~~~~~~~~~

::

   source ~/source_me_for_MOOSE
   bwpy-environ -- scripts/update_and_rebuild_libmesh.sh  2>&1 | tee Build_Log
   *** This takes about 3 hours.

   cd moose/tutorials/darcy_thermo_mech/step01_diffusion
   bwpy-environ -- make -j 8
   *** This takes about 1 hour. 

How to use MOOSE on an interactive mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   qsub -I -l nodes=1:ppn=32 -q debug
   *** after interactive mode starts

   cd ~/MOOSE_BUILD
   source ~/source_me_for_MOOSE
   cd moose/tutorials/darcy_thermo_mech/step01_diffusion/problems
   aprun -n 1 ../darcy_thermo_mech-opt -i step1.i

Sample test results
~~~~~~~~~~~~~~~~~~~

::

   Framework Information:
   MOOSE Version:           git commit f8a007d7a2 on 2019-06-24
   LibMesh Version:         82a1cfa7e84f28c361a0ee7b59dcc99c67e11425
   PETSc Version:           3.7.2
   Current Time:            Wed Jun 26 15:41:38 2019
   Executable Timestamp:    Wed Jun 26 15:41:29 2019

   Parallelism:
     Num Processors:          1
     Num Threads:             1

   Mesh:
     Parallel Type:           replicated
     Mesh Dimension:          2
     Spatial Dimension:       2
     Nodes:
       Total:                 1111
       Local:                 1111
     Elems:
       Total:                 1000
       Local:                 1000
     Num Subdomains:          1
     Num Partitions:          1

   Nonlinear System:
     Num DOFs:                1111
     Num Local DOFs:          1111
     Variables:               "pressure"
     Finite Element Types:    "LAGRANGE"
     Approximation Orders:    "FIRST"

   Execution Information:
     Executioner:             Steady
     Solver Mode:             NEWTON
     MOOSE Preconditioner:    SMP (auto)

    0 Nonlinear |R| = 1.326650e+04
         0 Linear |R| = 1.326650e+04
         1 Linear |R| = 2.780515e+01
         2 Linear |R| = 9.809271e-01
         3 Linear |R| = 3.423845e-02
    1 Nonlinear |R| = 3.423845e-02
         0 Linear |R| = 3.423845e-02
         1 Linear |R| = 5.746765e-03
         2 Linear |R| = 2.293476e-04
         3 Linear |R| = 1.162018e-05
         4 Linear |R| = 7.954630e-07
         5 Linear |R| = 4.426672e-08
    2 Nonlinear |R| = 4.426667e-08
    Solve Converged!

Additional Information / References
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  MOOSE Home Page: http://mooseframework.org

Last tested: 2019-06-26 by rhaas
