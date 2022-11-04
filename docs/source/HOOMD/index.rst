HOOMD-blue
==========

A. Description
~~~~~~~~~~~~~~

HOOMD is a general-purpose particle simulation toolikit developed in
Glotzer lab at the University of Michigan. It is implemented as a python
package, and supports GPU accelerators.

B. How to obtain HOOMD-blue
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hoomd-blue is an open-source package, and is freely available for
download from the Bitbucket site:

::

  git clone https://bitbucket.org/glotzer/hoomd-blue.git
  git clone https://bitbucket.org/glotzer/hoomd-examples.git

Blue Waters provides a precompiled serial and MPI-version of GPU-enabled
HOOMD package as a part of Blue Waters python module. Serial version of
the package becomes available in python path after loading the bwpy
module:

::

  module swap PrgEnv-cray PrgEnv-gnu
  module swap gcc gcc/4.9.3
  module load cudatoolkit/7.5.18-1.0502.10743.2.1
  module load bwpy/0.3.0

MPI-version of HOOMD becomes available after additionally loading
bwpi-mpi module:

::

  module swap PrgEnv-cray PrgEnv-gnu
  module swap gcc gcc/4.9.3
  module load cudatoolkit/7.5.18-1.0502.10743.2.1
  module load bwpy/0.3.0
  module load bwpy-mpi

Serial version of HOOMD-package is intended for interactive use in CCM
mode on an XK node. See the `CCM
page <https://bluewaters.ncsa.illinois.edu/cluster-compatibility-mode>`__
for instructions on how to launch an interactive CCM job on an XK node.

MPI-version of the code is intended for production computations
submitted in a batch mode. Section D bellow provides the necessary
example.

C. How to build HOOMD-blue
~~~~~~~~~~~~~~~~~~~~~~~~~~

Blue Waters provides site-optimized precompiled serial and MPI versions
of GPU-enabled HOOMD package as a part of Blue Waters python module. Due
to performance reasons related to dynamic library linking nature of
python packages, it is highly recommended that the users use the
provided precompiled binaries from the optimized python suit. Developers
who need access to the source code and perform frequent modifications of
the code may compile HOOMD-blue on Blue Waters using the following
instructions.

C.1 Step-by-step compilation instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   #!/bin/bash
   module swap PrgEnv-cray PrgEnv-gnu
   module swap gcc gcc/4.9.3
   module load cmake
   module load bwpy/0.3.0
   module load cudatoolkit/7.5.18-1.0502.10743.2.1
   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes
   export SOFTWARE_ROOT=/projects/sciteam/psn/hoomd
   export CPATH="${BWPY_DIR}/usr/include"
   export LIBRARY_PATH="${BWPY_DIR}/lib64:${BWPY_DIR}/usr/lib64"
   export LD_LIBRARY_PATH="${BWPY_DIR}/lib64:${BWPY_DIR}/usr/lib64:${LD_LIBRARY_PATH}"
   
   export CC=cc
   export CXX=CC
   
   cd $SOFTWARE_ROOT
   git clone https://bitbucket.org/glotzer/hoomd-blue.git
   cd hoomd-blue
   git checkout v2.1.9
   mkdir build; cd build

   cmake .. \\
   -DCMAKE_INSTALL_PREFIX=$SOFTWARE_ROOT/install \\
   -DPYTHON_EXECUTABLE=\`which python\` \\
   -DCUDA_TOOLKIT_ROOT_DIR=$CRAY_CUDATOOLKIT_DIR \\
   -DCUDA_ARCH_LIST="35" \\
   -DCMAKE_C_FLAGS="-fPIC" \\
   -DCMAKE_CXX_FLAGS="-fPIC" \\
   -DMPIEXEC="aprun" \\
   -DMPIEXEC_NUMPROC_FLAG="-n" \\
   -DENABLE_CUDA=ON \\
   -DENABLE_MPI=ON \\
   -DBUILD_CGCMM=OFF \\
   -DBUILD_DEM=OFF \\
   -DBUILD_DEPRECATED=OFF \\
   -DBUILD_HPMC=OFF \\
   -DBUILD_METAL=OFF \\
   -DBUILD_MD=ON \\
   -DBUILD_TESTING=OFF

   export VERBOSE=1
   make -j6
   make install

Save these instructions in a file build.cmd, and type "source ./build.cmd" from command line.

Before compiling, replace "/projects/sciteam/psn/hoomd" with your own
software path pointing to projects file system. Do not use home
filesystem as the python package storage point to avoid slowing down
the login node during python execution.

D. Sample test
~~~~~~~~~~~~~~

Prepare input file lj.py, which describes MD simulation of a set of
Lennard-Jones particles:

::

   import hoomd
   import hoomd.md

   import inspect
   print inspect.getfile(hoomd)

   hoomd.context.initialize("");
   hoomd.init.create_lattice(unitcell=hoomd.lattice.sc(a=4.0), n=110);
   nl = hoomd.md.nlist.cell();
   lj = hoomd.md.pair.lj(r_cut=2.5, nlist=nl);
   lj.pair_coeff.set('A', 'A', epsilon=1.0, sigma=1.0);
   hoomd.md.integrate.mode_standard(dt=0.005);
   all = hoomd.group.all();
   hoomd.md.integrate.langevin(group=all, kT=0.2, seed=42);
   hoomd.analyze.log(filename="log-output.log",
     quantities=['potential_energy', 'temperature'],
     period=100,
     overwrite=True);
   hoomd.dump.gsd("trajectory.gsd", period=2e3, group=all, overwrite=True);
   hoomd.run(3000);

D.1 Run precompiled version of HOOMD-blue
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create file run.pbs having the following content:

::

   #!/bin/bash
   #PBS -N ztest
   #PBS -l nodes=1:ppn=16:xk
   #PBS -l walltime=00:30:00
   #PBS -q debug
   #PBS -j oe

   cd $PBS_O_WORKDIR

   . /opt/modules/default/init/bash
   module swap PrgEnv-cray PrgEnv-gnu
   module swap gcc gcc/4.9.3
   module load cudatoolkit/7.5.18-1.0502.10743.2.1
   module load bwpy/0.3.0
   module load bwpy-mpi
   module list

   aprun -n 1 -N 1 python lj.py &> job.out

Submit the job

::

  qsub run.pbs

.. container::

D.2 Run locally compiled version of HOOMD-blue
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create file run.pbs having the following content:

::

   #!/bin/bash
   #PBS -N ztest
   #PBS -l nodes=1:ppn=16:xk
   #PBS -l walltime=00:30:00
   #PBS -q debug
   #PBS -j oe

   cd $PBS_O_WORKDIR

   . /opt/modules/default/init/bash
   module swap PrgEnv-cray PrgEnv-gnu
   module swap gcc gcc/4.9.3
   module load cudatoolkit/7.5.18-1.0502.10743.2.1
   module load bwpy/0.3.0
   module list
   export LD_LIBRARY_PATH="${BWPY_DIR}/lib64:${BWPY_DIR}/usr/lib64:${LD_LIBRARY_PATH}"
   export PACKAGE_ROOT=/projects/sciteam/psn/hoomd
   export PYTHONPATH=${PACKAGE_ROOT}:$PYTHONPATH
   export CRAY_CUDA_MPS=1
   export MPICH_G2G_PIPELINE=16

   aprun -n 1 -N 1 python lj.py &> job.out


Submit the job

::

  qsub run.pbs
