SLEPc
=====

Description
~~~~~~~~~~~

SLEPc (i.e., Scalable Library for Eigenvalue Problem Computations) is a
software library for the solution of large scale sparse eigenvalue
problems on parallel computers. It is an extension of PETSc and can be
used for linear eigenvalue problems in either standard or generalized
form, with real or complex arithmetic. It can also be used for computing
a partial singular value decomposition of a large, sparse, rectangular
matrix, and to solve nonlinear eigenvalue problems (polynomial or
general). Additionally, SLEPc provides solvers for the computation of
the action of a matrix function on a vector.

How to build SLEPc
~~~~~~~~~~~~~~~~~~

Downloading SLEPc on Blue Waters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   To download a specific version of SLEPc tarball and extract the
   sources, use:

::

   % mkdir ~/SLEPc
   % cd ~/SLEPc
   % wget http://slepc.upv.es/download/distrib/slepc-3.6.1.tar.gz
   % tar -zxvf slepc-3.6.1.tar.gz

.. container::
   :name: cke_pastebin

   .. rubric:: Building
      :name: building

.. container::

   To load PETSc (e.g., cray-petsc, or cray-petsc-complex) and TPSL
   modules for SLEPc use:

::

   % module load cray-petsc-complex/3.6.1.0
   % module load cray-tpsl
   % module swap cray-mpich cray-mpich/7.2.5

To set other environmental variables use:

::

   % export CRAYPE_LINK_TYPE=dynamic
   % export CRAY_ADD_RPATH=yes
   % export SLEPC_DIR=~/SLEPc/slepc-3.6.1

.. container::
   :name: cke_pastebin

   To build SLEPc use:

::

   % cd $SLEPC_DIR
   % ./configure
   % make

How to use SLEPc
~~~~~~~~~~~~~~~~

To set up the environment variables use:

::

   % module load cray-petsc-complex/3.6.1.0
   % module load cray-tpsl
   % module swap cray-mpich cray-mpich/7.2.5
   % export CRAYPE_LINK_TYPE=dynamic
   % export CRAY_ADD_RPATH=yes
   % export SLEPC_DIR=~/SLEPc/slepc-3.6.1

Additional Information / References
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  SLEPc Home Page: http://slepc.upv.es
-  Cray PETSc on Blue Waters: https://bluewaters.ncsa.illinois.edu/petsc
