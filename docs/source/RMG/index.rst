RMG
===

Description
~~~~~~~~~~~~~~

RMG is a density functional theory (DFT) based electronics structure
code that uses real space grids to represent wavefunctions, charge
densities, and ionic potentials. Designed for scalability, it has been
run succesfully on systems with thousands of nodes (including GPU nodes)
and hundreds of thousands of CPU cores. RMG reached 1 petaflops on 3,872
Cray XK nodes. It is currently under active development, and
contributions are gladly accepted.

RMG source code, precompiled binaries for Microsoft Windows and some
Linux systems, and examples can be found at
http://rmgdft.sourceforge.net

How to download RMG
~~~~~~~~~~~~~~~~~~~~~~

Executable binaries for Blue Waters are available for downloading.

`CPU
version <http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Binaries/BlueWaters/rmg_CPU/download>`__

`GPU
version <http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Binaries/BlueWaters/rmg_GPU/download>`__

Download and unpack the source code:

::

   mkdir $HOME/rmg
   wget http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Sources/rmg-release1.2.0.tar.gz/download
   tar zxvf rmg-release1.2.0.tar.gz

How to build RMG
~~~~~~~~~~~~~~~~~~~

On Blue Waters, RMG can be built under GNU programming environment.

Building the CPU-code under GNU environment
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$


- Configure programming environment

::

   module swap PrgEnv-cray PrgEnv-gnu
   module add fftw
   module add cmake
   module add boost

- Run cmake and make after that

::

   cd $HOME/rmg/rmg-release1.2.0
   mkdir build-cpu
   cd build-cpu
   cmake ..
   make -j 16


- Executable file **rmg** will be generated


The latest successful compilation was conducted under the
following environment:

:: 

      PrgEnv-gnu/5.2.40
      gcc/4.8.2
      cray-mpich/7.2.0
      fftw/3.3.4.1
      cmake/3.1.3
      boost/1.53.0


Building the GPU-code under GNU environment
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ 

- Configure programming environment

:: 

    module swap PrgEnv-cray PrgEnv-gnu
    module add fftw
    module add cmake
    module add boost
    module add cudatoolkit

- Run cmake and make after that

:: 

   cd $HOME/rmg/rmg-release1.2.0
   mkdir build-gpu
   cd build-gpu

- Edit ../CMakelists.txt and set RMG_GPU_ENABLED to 1

- Run cmake and make after that

:: 

   cmake ..
   make -j 16

- Executable file **rmg** will be generated

The latest successful compilation was conducted under the
            following environment:

::

   PrgEnv-gnu/5.2.40
   gcc/4.8.2
   cray-mpich/7.2.0
   fftw/3.3.4.1
   cmake/3.1.3
   boost/1.53.0
   cudatoolkit/6.5.14-1.0502.9613.6.1

Tests
$$$$$$$$$

RMG source code comes with input examples. One of those will be used to
demonstrate the use of RMG on Blue Waters.

::

   cd $HOME/rmg/rmg-release1.2.0/Examples/C60/PotentialAcceleration


Sample run.pbs file for CPU code:

::

   #!/bin/bash
   #PBS -j oe
   #PBS -l walltime=00:30:00
   #PBS -l nodes=2:ppn=32:xe
   #PBS -q debug
   #PBS -N ztest

   source /opt/modules/default/init/bash
   module swap PrgEnv-cray PrgEnv-gnu
   module list

   export MPICH_MAX_THREAD_SAFETY=serialized
   export OMP_WAIT_POLICY=passive
   export MPICH_ENV_DISPLAY=1
   export MPICH_ALLREDUCE_NO_SMP=1
   export OMP_NUM_THREADS=16

   cd$PBS_O_WORKDIR

   aprun-n2-N1-d$OMP_NUM_THREADS$HOME/rmg/rmg-release1.2.0/build-cpu/rmg in.c60potential_acc

Sample run.pbs file for GPU code:

:: 

   #!/bin/bash
   #PBS -j oe
   #PBS -l walltime=00:30:00
   #PBS -l nodes=2:ppn=16:xk
   #PBS -q debug
   #PBS -N ztest
   
   source /opt/modules/default/init/bash

   module swap PrgEnv-cray PrgEnv-gnu
   module list
   export MPICH_MAX_THREAD_SAFETY=serialized
   export OMP_WAIT_POLICY=passive
   export MPICH_ENV_DISPLAY=1
   export MPICH_ALLREDUCE_NO_SMP=1
   export CRAY_CUDA_PROXY=1
   export OMP_NUM_THREADS=16
   
   cd$PBS_O_WORKDIR

   aprun -m6Gh -n 2 -N 1 -d $OMP_NUM_THREADS -cc numa_node $HOME/rmg/rmg-release1.2.0/build-gpu/rmg in.c60potential_acc

Submit the job:

:: 

   qsub run.pbs
