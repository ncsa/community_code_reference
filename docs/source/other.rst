Mathematical Libraries
======================

LibSci
------

Cray LibSci is a library of highly tuned linear algebra and FFT products
for Cray XE6 systems. Cray LibSci includes the following:

-  Highly Optimized Basic Linear Algebra Subroutines (BLAS). Cray LibSci
   includes a mixture of highly optimized x86 BLAS from the University
   of Texas' LibGoto library and custom BLAS tunings provided
   specifically for Cray processors. Cray's BLAS are tuned for both
   serial and SMP execution. Both serial and threaded versions of the
   BLAS library are provided.

-  Linear Algebra Package (LAPACK). Cray LibSci includes serial
   numerical linear algebra routines optimized specifically for Cray XE
   systems. The optimizations in Cray LibSci are mainly at the
   algorithmic level, affecting both single core and SMP performance.

-  Scalable LAPACK (ScaLAPACK). Cray provides an optimized ScaLAPACK
   library for Cray XE systems based on ScaLAPACK v1.8 from Netlib
   (http://www.netlib.org/scalapack). Cray's ScaLAPACK library is a
   distributed memory programming model using MPI. This library includes
   parallel communications improvements that are not part of the
   standard ScaLAPACK distribution, as well as some algorithmic
   improvements that mirror those made for the LAPACK library.

-  Iterative Refinement Toolkit (IRT). IRT is a custom, Cray specific
   library that allows users of LAPACK and ScaLAPACK to solve linear
   systems with increased efficiency, by using mixed precision iterative
   refinement. IRT includes a set of routines to help understand
   advanced performance available using iterative refinement, and a set
   of wrappers that allow IRT to be used without changing user-code
   where possible. For well conditioned problems, IRT can provide up to
   40% performance improvement with little or no code changes.

-  The custom Cray Adaptive FFT (CRAFFT) library provides a very simple
   interface into existing FFT functionality on Cray systems. CRAFFT
   uses offline and online testing information to adaptively select the
   best FFT algorithm from the available FFT options. CRAFFT provides a
   very simple user interface into advanced FFT functionality and
   performance. Planning and execution are combined into one call with
   CRAFFT. The library comes packaged with pre-computed plans so that in
   many cases the planning stage can be omitted. For Cray XE systems,
   CRAFFT is currently integrated with FFTW.

How to use LibSci
~~~~~~~~~~~~~~~~~

The modulefile is loaded by default. This is true for all the available
programming environments (PGI, GNU, and Cray) as long as you use the
Cray compiler wrappers (ftn, cc, and CC). All LibSci routines will be
loaded by all Cray-provided compiler wrappers by default. No user flags
or options are required for compiling or linking.

::

           % module list
   xt-libsci/<version of LibSci>

Example Code for Matrix Matrix Multiply using ScaLAPACK library from
SciLib,
`test_scilib_scalapack.f90 <https://bluewaters.ncsa.illinois.edu/documents/10157/03493d1b-39a9-4e97-814e-73d0a5cbc5c9>`__

Compile the code as

+---------------------------------------+
| ftn test_scilib_scalapack.f90 -o test |
+---------------------------------------+

Additional Information
~~~~~~~~~~~~~~~~~~~~~~

-  More information can be found in the `Cray Application Developer's
   Environment User's
   Guide. <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=Search;browse=1;this_sort=release_date%20desc;q=2396>`__
   Always refer to `docs.cray.com <http://docs.cray.com/>`__ for the
   version available on your system or the latest version of the
   document.

FFTW
----

`FFTW <http://www.fftw.org/>`__ is a C subroutine library for computing
the discrete Fourier transform in one or more dimensions of arbitrary
input size, and of both real and complex data. FFTW v2.1.5 and FFTW v3.3
are available for the Cray XE6 system and distributed with the Cray
Programming Environment.

How to use FFTW
~~~~~~~~~~~~~~~

% module load fftw

To see which versions of FFTW are available. Use

::

   %module avail fftw

To compile and link a sequential 1D code :

::

   % ftn test1D.f $FFTW_POST_LINK_OPTS -lfftw

To compile and link MPI 3D code :

::

   % ftn test3D.f $FFTW_POST_LINK_OPTS -ldfftw_mpi -ldfftw

The order in which libraries are given is important.

Examples (coming soon)

PETSc
-----

`PETSc <https://www.mcs.anl.gov/petsc/>`_ , the Portable, Extensible Toolkit
for Scientific computation, is provided as part of the Cray Programming
Environment software stack. It provides sets of tools for the parallel
(as well as serial), numerical solution of PDEs that require solving
large-scale, nonlinear and sparse linear systems of equations. Cray
provides an optimized version of the standard PETSc Library. This
package is highly tuned for Cray hardware via Cray's custom Cray
Adaptive Sparse Kernels (CASK).

CASK is an auto-tuned and adaptive framework for the optimization of
Sparse Matrix Vector operations. CASK uses a custom code generator and
the result of extensive offline research to select the best optimization
method according to the problem characteristics. This adaptive approach
is vital in the optimization of sparse linear solvers since the
performance of numeric kernels is highly dependent on sparsity
characteristics of the calling problem. CASK provides CSR (Compressed
Sparse Row format) and FBSR (Fixed Block Sparse Row) tuned kernels that
can improve PETSc performance by 5% to 30%. The exact performance
advantage from CASK depends on the matrix class of the calling problem.

How to use PetSC
~~~~~~~~~~~~~~~~

::

   % module load cray-petsc

This will load the default version of PetSc. to load a specific version
available on the system, use

::

   % module avail cray-petsc

then load the specific version, for example,

::

   % module load cray-petsc/<version>

Examples : (coming soon)

.. _additional-information-1:

Additional Information
~~~~~~~~~~~~~~~~~~~~~~

More information can be found in
`PETSc <https://www.mcs.anl.gov/petsc/>`__ and the `Cray Application
Developer's Environment User's
Guide. <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=Search;browse=1;this_sort=release_date%20desc;q=2396>`__
Always refer to `docs.cray.com <http://docs.cray.com/>`__ for the
version available on your system or the latest version of the document.

Trilinos
--------

Trilinos is also provided as part of the Cray Programming Environment
software stack. It provides a set of packages to solve linear and
non-linear systems of equations, eigensystems and other related problems
while leveraging the value of established libraries. Cray provides an
optimized version of the standard Trilinos library which is highly tuned
for Cray hardware via Cray's custom CASK.

CASK is an auto-tuned and adaptive framework for the optimization of
Sparse Matrix Vector operations. CASK uses a custom code generator and
the result of extensive offline research to select the best optimization
method according to the problem characteristics. This adaptive approach
is vital in the optimization of sparse linear solvers since the
performance of numeric kernels is highly dependent on sparsity
characteristics of the calling problem. CASK provides CSR (Compressed
Sparse Row format) and FBSR (Fixed Block Sparse Row) tuned kernels that
can improve Trilinos performance by 5% to 30%. The exact performance
advantage from CASK depends on the matrix class of the calling problem.

How to use Trilinos
~~~~~~~~~~~~~~~~~~~

::

   % module load cray-trilinos

More Information
~~~~~~~~~~~~~~~~

-  The Trilinos Project : https://trilinos.org/
-  List of available documents on Trilinos :
   http://trilinos.sandia.gov/documentation.html
-  More information can be found in the `Cray Application Developer's
   Environment User's
   Guide. <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=Search;browse=1;this_sort=release_date%20desc;q=2396>`__
   Always refer to `docs.cray.com <http://docs.cray.com/>`__ for the
   version available on your system or the latest version of the
   document.

ACML: AMD Core Math Library
---------------------------

The 64-bit AMD Core Math Library (ACML) has been optimized for the Cray
XE/XK-series and is included with the LibSci Scientific Library. It
includes:

-  Level 1, 2, and 3 Basic Linear Algebra Subroutines (BLAS)
-  A full suite of Linear Algebra (LAPACK) routines
-  A suite of Fast Fourier Transform routines (ACML FFTs) for
   single-precision, double-precision, single-precision complex, and
   double-precision complex data types
-  Random number generators

How to use ACML
~~~~~~~~~~~~~~~

::

   % module load acml

To compile and link, use

::

   % ftn test_acml.f -L$ACML_BASE_DIR/pgi64/lib -lacml

Example : (coming soon)

.. _additional-information-2:

Additional Information
~~~~~~~~~~~~~~~~~~~~~~

-  AMD webpage for ACML :
   http://developer.amd.com/libraries/acml/pages/default.aspx
-  `Latest version documentation at AMD web site (v 5.0 as of
   now) <http://developer.amd.com/libraries/acml/onlinehelp/Documents/index.html>`__
-  Cray Docs on ACML (v4.3) -
   [`PDF <../../../download/attachments/18711982/S-6511-43.pdf?version=1&modificationDate=1320769027000>`__
   ]

GSL : GNU Scientific Library
----------------------------

The GNU Scientific Library (GSL) is a numerical library for C and C++
programmers. It is free software under the GNU General Public License.

How to use GSL
~~~~~~~~~~~~~~

A programming envirnment should be loaded before loading this module.

%module load gsl

To see which enviornment veriables are set, use :

%module show gsl

To compile and link a code that uses GSL,

%cc -I$GSL_INCLUDE_PATH -L$GSL_LIBRARY_PATH test_gsl.c -lgsl

References
----------

-  Math & other optimized libraries for processor

   -  
      `Cray Application Developer's Environment User's Guide: Scientific
      and Math
      Libraries <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=Show;q=;f=/books/S-2396-601/html-S-2396-601//chapter-ck3x6qu8-brbethke.html>`__
      (CSML)

      -  LibSci
      -  FFTW
      -  PETSc (ParMetis, Scotch, SuperLU, SuperLU_DIST, MUMPS, SUNDIALS
         and Hypre), Trilinos, and CASK
      -  ACML
      -  HDF5 and NetCDF

   -  ACML - AMD Core Math Library

      -  latest version documentation at AMD web site (v 5.0 as of now)
         -
         `PDF <http://developer.amd.com/libraries/acml/downloads/assets/acml.pdf>`__
         `Text <http://developer.amd.com/libraries/acml/onlinehelp/Documents/index.html>`__
      -  Cray Docs (v4.3) -
         `PDF <http://docs.cray.com/books/S-6511-43/S-6511-43.pdf>`__

   -  Cray SuperLU -
      `PDF <http://docs.cray.com/books/S-6532-30/S-6532-30.pdf>`__
   -  PGI

      -  PGI documentation for various libraries & applications -
         `link <http://www.pgroup.com/resources/tips.htm>`__

   -  AMD LibM - AMD LibM is a software library containing a collection
      of basic math functions optimized for x86-64 processor based
      machines. It provides many routines from the list of standard C99
      math functions. AMD LibM is a C library, which users can link in
      to their applications to replace compiler-provided math functions.
      LibM is forked out from ACML. `Home Page on
      AMD <http://developer.amd.com/libraries/LibM/Pages/default.aspx>`__
      . (this link seems to be under development, user guide is not yet
      available.)
   -  Cray LibSci Documentation -
      `here <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=View;id=S-2396-601;idx=books_search;this_sort=release_date%20desc;q=;type=books;title=Cray%20Application%20Developer%27s%20Environment%20User%27s%20Guide>`__
      (Click on Chapter 5).The Cray LibSci collection contains the
      following Scientific Libraries:

      -  BLAS (Basic Linear Algebra Subroutines)
      -  BLACS (Basic Linear Algebra Communication Subprograms)
      -  LAPACK (Linear Algebra Routines)
      -  ScaLAPACK (Scalable LAPACK)
      -  FFT (Fast Fourier Transform Routines)
      -  FFTW2 (the Fastest Fourier Transforms in the West, release 2)
      -  FFTW3 (the Fastest Fourier Transforms in the West, release 3)
         In addition, the Cray LibSci collection contains three
         libraries developed by Cray.
      -  IRT (Iterative Refinement Toolkit)
      -  CASE (Cray Adaptive Sparse Eigensolvers)
      -  CRAFFT (Cray Adaptive Fast Fourier Transform Routines)
         LibSci documentation in **Cray Application Developer's
         Environment User's Guide** is
         `here <http://docs.cray.com/cgi-bin/craydoc.cgi?mode=View;id=S-2396-601;idx=books_search;this_sort=release_date%20desc;q=;type=books;title=Cray%20Application%20Developer%27s%20Environment%20User%27s%20Guide>`__
         Click on Chapter 5 for LibSci. Link to
         `PDF <http://docs.cray.com/books/S-2396-601/S-2396-601.pdf>`__
      -  ACML (no longer default)
      -  Fast Math Intrinsics
      -  PETSc (Portable, Extensible, Toolkit for Scientific
         Computation)
      -  Trilinos

   -  `Intrinsics <http://goo.gl/J93O1>`__
   -  IO Libraries - Link to the IO specific page

-  Math & other optimized libraries for accelerator (Fermi for now)

   -  `Nvidia GPU-Accelerated
      Libraries <http://developer.nvidia.com/gpu-accelerated-libraries>`__
   -  `High Performance DGEMM on
      Fermi <http://asg.ict.ac.cn/projects/dgemm/dgemm_nv.html>`__
   -  `An Improved MAGMA GEMM for Fermi
      GPUs <http://www.netlib.org/lapack/lawnspdf/lawn227.pdf>`__
