WRF
===

A. Description
~~~~~~~~~~~~~~

The Weather Research and Forecasting (WRF) model is a numerical weather
prediction (NWP) and atmospheric simulation system designed for both
research and operational applications. WRF is supported as a common tool
for the university/research and operational communities to promote
closer ties between them and to address the needs of both.

The development of WRF has been a multi-agency effort to build a
next-generation mesoscale forecast model and data assimilation system to
advance the understanding and prediction of mesoscale weather and
accelerate the transfer of research advances into operations.

The WRF effort has been a collaborative one among the National Center
for Atmospheric Research's (NCAR) Mesoscale and Microscale Meteorology
(MMM) Division, the National Oceanic and Atmospheric Administration's
(NOAA) National Centers for Environmental Prediction (NCEP) and Earth
System Research Laboratory (ESRL), the Department of Defense's Air Force
Weather Agency (AFWA) and Naval Research Laboratory (NRL), the Center
for Analysis and Prediction of Storms (CAPS) at the University of
Oklahoma, and the Federal Aviation Administration (FAA), with the
participation of university scientists.

WRF source code, examples, and documentation can be found at
https://www2.mmm.ucar.edu/wrf/users/

B. How to download WRF
~~~~~~~~~~~~~~~~~~~~~~

From the main WRF page, select "User Resources" and follow the link
below "WRF Releases". From the "Download" link at the top of the page,
you will be able to download the most recent versions of WRF (the main
weather forecasting model code) and WPS (the WRF pre-processing system),
as well as other related software and previous versions.

If you are a new WRF user, and have never downloaded WRF before, you
will be asked to answer a few questions in order to register as a WRF
user and create an account for future downloads. If you have already
registered as a WRF user, you will be asked to enter your account
information.

The installation files are in standard gzipped tar file format; place
the tarball in your home or projects directory on Blue Waters, and type
the following (WRF version 4.0 is used here as an example; other
versions of WRF, WPS, etc... can be downloaded and extracted in a
similar manner):

::

   cd ~
   tar -xzf WRFV4.0.TAR.gz

This will create a sub-directory called "WRF", from which you can
configure and install WRF.

C. How to build WRF
~~~~~~~~~~~~~~~~~~~

On Blue Waters, WRF has been built with the Cray, PGI, and Intel
Programming Environments. It should be possible to build with the GNU
Programming Environment, but this has not been tested. The build steps
for the different compilers differ only in the loaded PrgEnv module and
the choice of the compiler in the configure step.

The following examples assume you are using the Bourne again shell
(bash); if you prefer to use a different shell, then some syntax
modifications will be necessary.

C.1 Building WRF with the Cray environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The "PrgEnv-cray" Progamming Environment module is loaded by default on
Blue Waters; if you have changed to a different Programming Environment,
e.g., PGI, use the "module swap" command to return to the Cray
Programming Environment, for example:

::

   module swap PrgEnv-pgi PrgEnv-cray

Then load the NetCDF module and set a few environment variables (the
prefix "cray-" on the "netcdf" module name has nothing to do with which
compiler is being used):

::

   module load cray-netcdf
   export NETCDF=${NETCDF_DIR}
   export WRFIO_NCD_LARGE_FILE_SUPPORT=1

There are a few things that need to be manually changed before building
WRF on Blue Waters. When you run the configure script, you will be
presented with several choices based on the compiler and type of
parallelization desired - choose the first of the Cray XE and Cray
compiler options.

::

   ./configure

After running the configure script, a file called "configure.wrf" will
be generated. Many, if not all, WRF distributions have Makefiles with a
"-w" flag on some of the rules for C compiling; this flag causes
unwanted behavior with the Cray compilers. In version 3.7 of WRF, for
example, the following lines of the "makefile" in directory
"WRFV3/external/io_int" need to be modified in order to remove this
flag:

CHANGE

::

   io_int_idx.o: io_int_idx.c io_int_idx.h io_int_idx_tags.h
          $(CC) -o $@ -c -w $*.c

TO

::

   io_int_idx.o: io_int_idx.c io_int_idx.h io_int_idx_tags.h
          $(CC) -o $@ -c $*.c

Different versions of WRF may have other instances of this "-w" flag in
various makefiles that need to be removed in a similar fashion. Now we
are ready to begin the build process - it can take several hours to
complete, and there may be long periods with no output generated, even
though the build is continuing.

::

   ./clean
   ./compile -j 8 wrf

Executable file "wrf.exe" will be generated in the "WRF/main" directory.

C.2 Building WRF with the PGI environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure your programming environment (the prefix "cray-" on the
"netcdf" module name has nothing to do with which compiler is being
used):

| module swap PrgEnv-cray PrgEnv-pgi
| module load cray-netcdf
| export NETCDF=${NETCDF_DIR}
| export WRFIO_NCD_LARGE_FILE_SUPPORT=1

There are a few things that need to be manually changed before building
WRF on Blue Waters. When you run the configure script, you will be
presented with several choices based on the compiler and type of
parallelization desired - choose the first of the PGI compiler options.

::

   ./configure

After running the configure script, a file called "configure.wrf" will
be generated. It is be necessary to edit this file, and manually change
the names of the Fortran and C compilers to "ftn" and "cc" (don't use
"pgcc"), respectively. You can use these sed lines to affect the
changes:

::

   sed -i '/^SFC/s/pgf90/ftn/g' configure.wrf
   sed -i '/^DM_FC/s/mpif90/ftn/g' configure.wrf
   sed -i '/^SCC/s/pgcc/cc/g' configure.wrf
   sed -i '/^CCOMP/s/pgcc/cc/g' configure.wrf
   sed -i '/^DM_CC/s/mpicc/cc/g' configure.wrf

Now we are ready to begin the build process - it can take several hours
to complete, and there may be long periods with no output generated,
even though the build is continuing.

::

   ./clean
   ./compile -j 8 wrf

Executable file "wrf.exe" will be generated in the "WRF/main" directory

C.3 Building WRF with the Intel environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure your programming environment (the prefix "cray-" on the
"netcdf" module name has nothing to do with which compiler is being
used):

::

   module swap PrgEnv-cray PrgEnv-intel
   module load cray-netcdf
   export NETCDF=${NETCDF_DIR}
   export WRFIO_NCD_LARGE_FILE_SUPPORT=1

There are a few things that need to be manually changed before building
WRF on Blue Waters. When you run the configure script, you will be
presented with several choices based on the compiler and type of
parallelization desired - choose the first of the Intel compiler
options.

::

   ./configure

After running the configure script, a file called "configure.wrf" will
be generated. It may be necessary to edit this file, and manually change
the names of the Fortran and C compilers to "ftn" and "cc" (don't use
"icc"), respectively. You can use these sed lines to affect the changes:

::

   sed -i '/^SFC/s/ifort/ftn/g' configure.wrf
   sed -i '/^DM_FC/s/mpif90/ftn/g' configure.wrf
   sed -i '/^SCC/s/icc/cc/g' configure.wrf
   sed -i '/^CCOMP/s/icc/cc/g' configure.wrf
   sed -i '/^DM_CC/s/mpicc/cc/g' configure.wrf

Now we are ready to begin the build process - it can take several hours
to complete, and there may be long periods with no output generated,
even though the build is continuing.

::

   ./clean
   ./compile -j 8 wrf

Executable file "wrf.exe" will be generated in the "WRF/main" directory

D. Building WPS utilities
~~~~~~~~~~~~~~~~~~~~~~~~~

WRF has an associated set of utilies in the WPS tool suite, also
available from https://www2.mmm.ucar.edu/wrf/users/

WPS interacts with images in png and jpeg format using the png and
JasPer libraries that need to be installed on the system. On Blue Waters
JasPer is provided in the JasPer module and png by the operating system
but only as a dynamic library.

Setting up the required environment requires several steps:

::

   # do this before loading any PrgEnv since it switches to PrgEnv-gnu
   module load JasPer/2.0.14-CrayGNU-2018.12
   module swap PrgEnv-gnu PrgEnv-cray

   # load PrgEnv modules and enable NETCDF output
   module swap PrgEnv-cray PrgEnv-intel # or whichever you lik
   module load cray-netcdf
   export NETCDF=${NETCDF_DIR}
   export WRFIO_NCD_LARGE_FILE_SUPPORT=1

   # JasPer and png libraries are only provided as a dynamic library, not static ones
   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes

Once the modules are loaded run the configure script and pick the
compiler you would like to use e.g. 18 for the Intel compiler with
distributed memory (MPI) parallelism.

::

   ./configure

Similar to WRF the file configure.wps must be adjusted to use only the
Cray compiler wrappers ftn and cc and not the "bare" compilers as well
as accounting for OpenMP used in NETCDF and other linked libraries and
to ensure that JasPer is found at runtime.

::

   # have to hack configure.wps to use Cray Wrappers
   # here intel compilers are used but similar changes are required for 
   # GNU, and PGI compilers
   echo "Editing configure.wps to use Cray wrappers"
   sed -i '/^SFC/s/ifort/ftn/g' configure.wps
   sed -i '/^DM_FC/s/mpif90 .*/ftn/g' configure.wps
   sed -i '/^SCC/s/icc/cc/g' configure.wps
   sed -i '/^DM_CC/s/mpicc .*/cc/g' configure.wps

   # add -fopenmp for OpenMP support (link failures otherwise)
   # here intel compilers are used but similar changes are required for
   # GNU, and PGI compilers
   sed -i '/^FFLAGS/s/$/ -fopenmp/g' configure.wps
   sed -i '/^F77FLAGS/s/$/ -fopenmp/g' configure.wps
   sed -i '/^LDFLAGS/s/$/ -fopenmp/g' configure.wps
   sed -i '/^CFLAGS/s/$/ -fopenmp/g' configure.wps
   sed -i '/^CPPFLAGS/s/$/ -fopenmp/g' configure.wps

   # (optional) ensure that libjasper is found at runtime
   sed -i "/^COMPRESSION_LIBS/s#-ljasper#-Wl,--rpath,$EBROOTJASPER/lib64 -ljasper#" configure.wps

Finally you can compile the tools

::

   ./compile

and you shoud find executables

::

   -rwx------ 1 rhaas bw_staff 3555836 Mar 24 12:19 ./geogrid/src/geogrid.exe
   -rwx------ 1 rhaas bw_staff 3482519 Mar 24 12:25 ./metgrid/src/metgrid.exe
   -rwx------ 1 rhaas bw_staff 1198213 Mar 24 12:25 ./ungrib/src/g1print.exe
   -rwx------ 1 rhaas bw_staff 1461851 Mar 24 12:25 ./ungrib/src/g2print.exe
   -rwx------ 1 rhaas bw_staff 2190893 Mar 24 12:23 ./ungrib/src/ungrib.exe
   -rwx------ 1 rhaas bw_staff 1212581 Mar 24 12:25 ./util/src/avg_tsfc.exe
   -rwx------ 1 rhaas bw_staff 1266336 Mar 24 12:25 ./util/src/calc_ecmwf_p.exe
   -rwx------ 1 rhaas bw_staff 1243278 Mar 24 12:26 ./util/src/height_ukmo.exe
   -rwx------ 1 rhaas bw_staff 1130093 Mar 24 12:26 ./util/src/int2nc.exe
   -rwx------ 1 rhaas bw_staff 1143334 Mar 24 12:25 ./util/src/mod_levs.exe
   -rwx------ 1 rhaas bw_staff  986067 Mar 24 12:25 ./util/src/rd_intermediate.exe

You will now be able to use the tools, and if you included the
-Wl,--rpath bit will not have to load the JasPer module to use them
either.
