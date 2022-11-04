PIO
===

A. Description
~~~~~~~~~~~~~~

PIO (Parallel I/O) is a tunable I/O library that supports both NetCDF
and pNetCDF (parallel NetCDF) on the backend. Now packaged for use with
a variety of different applications, the PIO library was originally
created by the people at NCAR to improve the performance of CESM.

The PIO user's guide can be found
`here <http://ncar.github.io/ParallelIO/index.html>`__.

B. Downloading PIO
~~~~~~~~~~~~~~~~~~

The PIO source code may be found on github
`here <https://github.com/NCAR/ParallelIO>`__.

To download the code to Blue Waters, simply clone the repository:

::

   git clone https://github.com/PARALLELIO/ParallelIO

C. Building PIO
~~~~~~~~~~~~~~~

The `PIO user's guide <http://ncar.github.io/ParallelIO/index.html>`__
has both a `general
installation <http://ncar.github.io/ParallelIO/install.html>`__ page and
a page that provides `installation
walk-throughs <http://ncar.github.io/ParallelIO/mach_walkthrough.html>`__
for different systems, including Blue Waters.

The build procedure below was taken from the Blue Waters PGI
installation walk-through and updated for use with the current system
configuration as of April 12, 2017.

Obtain the source code using\ ``git clone``\ as shown above.

Paste the script below into a file, such as ``pio_build_script``, in the
same directory as the ``ParallelIO`` directory that was created with the
above git command. Set ``INSTALLATION_PATH`` to the full path where you
want to install the ``lib`` and ``include`` directories
(``CMAKE_INSTALL_PREFIX`` is set to ``INSTALLATION_PATH`` later in the
script), make the script executable with ``chmod +x <script_name>``, and
then run it.

::

   #!/bin/bash

   # specify your desired installation path
   INSTALLATION_PATH=/u/sciteam/$USER/libs/pio

   # =================================================================

   . /opt/modules/default/etc/modules.sh

   module unload PrgEnv-cray
   module unload PrgEnv-gnu
   module unload PrgEnv-pgi
   module unload PrgEnv-intel

   module load PrgEnv-pgi
   module swap pgi pgi/16.9.0
   module load cmake
   module load cray-hdf5-parallel/1.10.0
   module load cray-netcdf-hdf5parallel/4.4.1
   module load cray-parallel-netcdf/1.7.0

   echo "============================================================="
   echo "Configuring PIO with these modules loaded:"
   echo ""
   module list -t
   echo ""
   echo "============================================================="
   echo ""

   CC=cc FC=ftn cmake -DCMAKE_VERBOSE_MAKEFILE=TRUE \
   -DPREFER_STATIC=TRUE \
   -DNetCDF_PATH=${NETCDF_DIR} \
   -DPnetCDF_PATH=${PARALLEL_NETCDF_DIR} \
   -DHDF5_PATH=${HDF5_DIR} \
   -DMPI_C_INCLUDE_PATH=${MPICH_DIR}/include \
   -DMPI_Fortran_INCLUDE_PATH=${MPICH_DIR}/include \
   -DMPI_C_LIBRARIES=${MPICH_DIR}/lib/libmpich.a \
   -DMPI_Fortran_LIBRARIES=${MPICH_DIR}/lib/libmpichf90.a \
   -DCMAKE_SYSTEM_NAME=Catamount \
   -DCMAKE_INSTALL_PREFIX=${INSTALLATION_PATH} \
   ParallelIO

   echo ""
   echo "============================================================="
   echo "Building PIO"
   echo "============================================================="
   echo ""

   make

   echo ""
   echo "============================================================="
   echo "Building PIO tests (run in interactive session with 'ctest')"
   echo "============================================================="
   echo ""

   make tests

   echo ""
   echo "============================================================="
   echo "Installing PIO"
   echo "============================================================="
   echo ""

   make install

   echo ""
   echo "============================================================="
   echo "Finished building and installing PIO library"
   echo "============================================================="
   echo ""

The following relevant modules were loaded for the most recent test of
this installation on April 12, 2017:

::

   craype/2.5.8
   cray-mpich/7.5.0
   xalt/0.7.6
   darshan/3.1.3
   cmake/3.1.3
   cray-hdf5-parallel/1.10.0
   cray-netcdf-hdf5parallel/4.4.1
   cray-parallel-netcdf/1.7.0
   pgi/16.9.0
   cray-libsci/16.11.1
   atp/2.0.4
   PrgEnv-pgi/5.2.82

D. Sample tests
~~~~~~~~~~~~~~~

After successful completion of the build process, the included tests can
be run from an interactive job.

::

   qsub -I -lnodes=1:ppn=32:xe -lwalltime=2:00:00

After the job starts, run the commands below. Note that
``<pio_installation_directory>/tests`` does not mean
``ParallelIO/tests``. If you display the contents of
``<pio_installation_directory>/tests``, there should be a
``CTestTestfile.cmake`` file.

::

   cd <pio_installation_directory>/tests
   module load cmake/3.1.3
   ctest

E. Known issues
~~~~~~~~~~~~~~~

When running ctest with default timeout settings, three of the tests run
out of time:

::

   test_darray_multivar (Timeout)
   pio_rearr_opts (Timeout)
   pio_rearr_opts2_3p (Timeout)

Doubling the time limit in the appropriate ``CTestTestfile.cmake`` files
easily helps them pass. Tests on 5/16/2017 showed that
test_darray_multivar took 152 sec (default timeout 120 sec),
pio_rearr_opts took 273 sec (default timeout 240 sec), and
pio_rearr_opts2_3p took 256 sec (default 240 sec).
