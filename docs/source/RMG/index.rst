RMG
===

A. Description
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

B. How to download RMG
~~~~~~~~~~~~~~~~~~~~~~

Executable binaries for Blue Waters are available for downloading.

`CPU
version <http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Binaries/BlueWaters/rmg_CPU/download>`__

`GPU
version <http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Binaries/BlueWaters/rmg_GPU/download>`__

.. container::

   Download and unpack the source code:

.. container::

.. container::

   .. container::

      mkdir $HOME/rmg

   .. container::

      wget
      http://sourceforge.net/projects/rmgdft/files/Releases/1.2/Sources/rmg-release1.2.0.tar.gz/download

   .. container::

      tar zxvf rmg-release1.2.0.tar.gz

C. How to build RMG
~~~~~~~~~~~~~~~~~~~

On Blue Waters, RMG can be built under GNU programming environment.

C.1 Building the CPU-code under GNU environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. container::

   - Configure programming environment

.. container::

.. container::

   module swap PrgEnv-cray PrgEnv-gnu

.. container::

   module add fftw

.. container::

   module add cmake

.. container::

   module add boost

.. container::

.. container::

   - Run cmake and make after that

.. container::

.. container::

   cd $HOME/rmg/rmg-release1.2.0

.. container::

   mkdir build-cpu

.. container::

   cd build-cpu

.. container::

   cmake ..

.. container::

   make -j 16

.. container::

   .. container::

      .. container::

         .. container::

         .. container::

            - Executable file **rmg** will be generated

         .. container::

         .. container::

            .. container::

               The latest successful compilation was conducted under the
               following environment:

.. container::

   .. container::
      :name: cke_pastebin

      PrgEnv-gnu/5.2.40

   .. container::
      :name: cke_pastebin

      gcc/4.8.2

   .. container::
      :name: cke_pastebin

      cray-mpich/7.2.0

   .. container::
      :name: cke_pastebin

      fftw/3.3.4.1

   .. container::

      cmake/3.1.3

   .. container::

      boost/1.53.0

   .. container::

      .. rubric:: C.2 Building the GPU-code under GNU environment
         :name: c.2-building-the-gpu-code-under-gnu-environment

      .. container::

         - Configure programming environment

      .. container::

      .. container::

         module swap PrgEnv-cray PrgEnv-gnu

      .. container::

         module add fftw

      .. container::

         module add cmake

      .. container::

         module add boost

      .. container::

         module add cudatoolkit

      .. container::

      .. container::

         - Run cmake and make after that

      .. container::

      .. container::

         cd $HOME/rmg/rmg-release1.2.0

      .. container::

         mkdir build-gpu

      .. container::

         cd build-gpu

      .. container::

      .. container::

         - Edit ../CMakelists.txt and set RMG_GPU_ENABLED to 1

      .. container::

      .. container::

         - Run cmake and make after that

      .. container::

      .. container::

         cmake ..

      .. container::

         make -j 16

      .. container::

         .. container::

         .. container::

            - Executable file **rmg** will be generated

         .. container::

         .. container::

            The latest successful compilation was conducted under the
            following environment:

      .. container::

         .. container::
            :name: cke_pastebin

            PrgEnv-gnu/5.2.40

         .. container::
            :name: cke_pastebin

            gcc/4.8.2

         .. container::
            :name: cke_pastebin

            cray-mpich/7.2.0

         .. container::
            :name: cke_pastebin

            fftw/3.3.4.1

         .. container::

            cmake/3.1.3

         .. container::

            boost/1.53.0

         .. container::

            cudatoolkit/6.5.14-1.0502.9613.6.1

.. container::

   .. rubric:: D. Tests
      :name: d.-tests

RMG source code comes with input examples. One of those will be used to
demonstrate the use of RMG on Blue Waters.

.. container::

   cd $HOME/rmg/rmg-release1.2.0/Examples/C60/PotentialAcceleration

.. container::

.. container::

   Sample run.pbs file for CPU code:

.. container::

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

.. container::

   aprun-n2-N1-d$OMP_NUM_THREADS$HOME/rmg/rmg-release1.2.0/build-cpu/rmg
   in.c60potential_acc

.. container::

.. container::

   Sample run.pbs file for GPU code:

.. container::
   :name: cke_pastebin

   .. container::

   .. container::

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

      aprun -m6Gh -n 2 -N 1 -d $OMP_NUM_THREADS -cc numa_node
      $HOME/rmg/rmg-release1.2.0/build-gpu/rmg in.c60potential_acc

   .. container::

   .. container::

      Submit the job:

   qsub run.pbs
