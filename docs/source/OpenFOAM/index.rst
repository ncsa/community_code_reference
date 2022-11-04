OpenFOAM
========

Description
~~~~~~~~~~~

OpenFOAM is a C++ toolbox for the development of customized numerical
solvers, and pre-/post-processing utilities for the solution of
continuum mechanics problems, including computational fluid dynamics
(CFD). The code is released as free and open-source software under the
GNU General Public License from the OpenFOAM Foundation.

Building OpenFOAM-5.x
~~~~~~~~~~~~~~~~~~~~~

Updating your shell/module environments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   Add the following lines to your ~/.bashrc:

::

   export CRAYPE_LINK_TYPE=dynamic
   export CRAY_ADD_RPATH=yes
   source ~/OpenFOAM/OpenFOAM-5.x/etc/bashrc

.. container::

   Add the following lines to ~/.modules (if not exist, create
   ~/.modules with the following lines):

::

   module swap PrgEnv-cray PrgEnv-gnu
   module load cmake boost

.. container::

   Run the followings:

::

   % source ~/.bashrc
   % source ~/.modules

.. container::

   It will complain about "~/OpenFOAM/OpenFOAM-5.x/etc/bashrc" since it
   does not exist yet. We will set it up later, so please ignore it at
   this step.

.. container::

Downloading OpenFOAM-5.x and ThirdParty-5.x from the GitHub
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   Create ~/OpenFOAM folder:

::

   % mkdir ~/OpenFOAM
   % cd ~/OpenFOAM

.. container::

   Download the source files:

::

   % git clone https://github.com/OpenFOAM/OpenFOAM-5.x.git
   % git clone https://github.com/OpenFOAM/ThirdParty-5.x.git

.. container::

   The following installation process was confirmed with the commit
   ee536e6 of OpenFOAM-5.x and the commit 62703a4 of ThirdParty-5.x. You
   may do the followings to use the tested commits:

::

   % cd ~/OpenFOAM/OpenFOAM-5.x
   % git checkout ee536e6
   % cd ~/OpenFOAM/ThirdParty-5.x
   % git checkout 62703a4

Updating ~/OpenFOAM/OpenFOAM-5.x/
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/etc/bashrc as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/etc/bashrc

   At line 89:
   [original] export WM_MPLIB=SYSTEMOPENMPI
   [modified] export WM_MPLIB=SYSTEMMPI

   Add the following lines as well:
   export MPI_ROOT=$MPICH_DIR
   export MPI_ARCH_FLAGS=" "
   export MPI_ARCH_INC=" "
   export MPI_ARCH_LIBS=" "

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/etc/config.sh/settings as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/etc/config.sh/settings

   At lines 64-65:
   [original]  export WM_CC='gcc'
   [original]  export WM_CXX='g++'
   [modified]  export WM_CC='cc'
   [modified]  export WM_CXX='CC'

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/wmake/rules/linux64Gcc/c as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/wmake/rules/linux64Gcc/c

   At line 5:
   [original] cc          = gcc -m64
   [modified] cc          = cc -m64

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/wmake/rules/linux64Gcc/c++ as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/wmake/rules/linux64Gcc/c++

   At line 8:
   [original] CC          = g++ -std=c++11 -m64
   [modified] CC          = CC -std=c++11 -m64

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/src/Pstream/Allwmake as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/src/Pstream/Allwmake

   At line 20:
   [original] touch "$whichmpi”
   [modified] #touch "$whichmpi"

.. container::

   Update ~/OpenFOAM/OpenFOAM-5.x/src/parallel/decompose/Allwmake as
   follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/src/parallel/decompose/Allwmake

   At line 34:
   [original] touch "$whichmpi" "$whichscotch"
   [modified] #touch "$whichmpi" "$whichscotch"

.. container::

   Update ptscotchDecomp.C as follows:

::

   % vi ~/OpenFOAM/OpenFOAM-5.x/src/parallel/decompose/ptscotchDecomp/ptscotchDecomp.C

   At lines 31-38:
   [original]
   #include "SubField.H"

   extern "C"
   {
       #include <stdio.h>
       #include <mpi.h>
       #include "ptscotch.h"
   }

   [modified]
   #include "SubField.H"
   #include "mpi.h"

   extern "C"
   {
       #include <stdio.h>
   //    #include <mpi.h>
       #include "ptscotch.h"
   }

Updating and building ~/OpenFOAM//ThirdParty-5.x
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   Create Makefile.inc.BlueWaters as follows:

::

   % cd ~/OpenFOAM/ThirdParty-5.x/scotch_6.0.3/src/Make.inc
   % cp Makefile.inc.x86-64_pc_linux2.shlib Makefile.inc.BlueWaters
   % vi Makefile.inc.BlueWaters

   At lines 6-11:
   [original]
   AR              = gcc
   ARFLAGS         = -shared -o
   CAT             = cat
   CCS             = gcc
   CCP             = mpicc
   CCD             = gcc

   [modified]
   AR              = cc
   ARFLAGS         = -shared -o
   CAT             = cat
   CCS             = cc
   CCP             = cc
   CCD             = cc

.. container::
   :name: cke_pastebin

   Update ~/OpenFOAM/ThirdParty-5.x/Allwmake:

::

   % vi ~/OpenFOAM/ThirdParty-5.x/Allwmake

   At line 192:
   [original] scotchMakefile=../../etc/wmakeFiles/scotch/Makefile.inc.i686_pc_linux2
   .shlib-OpenFOAM
   [modified] scotchMakefile=./Make.inc/Makefile.inc.BlueWaters

.. container::

   Build ThirdParty-5.x, as follows (this step takes several minutes):

::

   % cd  ~/OpenFOAM/ThirdParty-5.x/
   % source ~/OpenFOAM/OpenFOAM-5.x/etc/bashrc
   % ./Allclean
   % ./Allwmake -j 8 > build_log_ThirdParty-5.x 2>&1

.. container::

   Check the built libraries:

::

   % ls -R ~/OpenFOAM/ThirdParty-5.x/platforms/linux64GccDPInt32/lib/
   ~/OpenFOAM/ThirdParty-5.x/platforms/linux64GccDPInt32/lib/:
   libscotcherrexit.so  libscotcherr.so  libscotchmetis.so  libscotch.so  mpi-system

   ~/OpenFOAM/ThirdParty-5.x/platforms/linux64GccDPInt32/lib/mpi-system:
   libptscotcherrexit.so  libptscotcherr.so  libptscotchparmetis.so  libptscotch.so
   libscotcherrexit.so  libscotcherr.so  libscotch.so

Building ~/OpenFOAM/OpenFOAM-5.x/
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. container::

   Build OpenFOAM-5.x, as follows (this step takes around 1.5 - 2 hours
   on Blue Waters, so using "screen" command is recommended):

::

   % cd ~/OpenFOAM/OpenFOAM-5.x
   % ./Allwmake -j 8 > build_log_OpenFOAM-5.x 2>&1

Testing OpenFOAM-5.x
~~~~~~~~~~~~~~~~~~~~

Testing the serial OpenFOAM:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   % mkdir ~/OpenFOAM/Test_OpenFOAM
   % cd ~/OpenFOAM/Test_OpenFOAM
   % cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity/cavity cavity_serial
   % cd cavity_serial
   % blockMesh
   % icoFoam 

Testing the parallel OpenFOAM:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   % qsub -I -l nodes=1:ppn=32:xe -l walltime=00:30:00 -q debug
   ... after starting the interactive mode ...
   % cd ~/OpenFOAM/Test_OpenFOAM
   % cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity/cavity cavity_parallel
   % cd cavity_parallel/system
   % vi decomposeParDict               <------- paste the followings into the file

   /*--------------------------------*- C++ -*----------------------------------*\
   | =========                 |                                                 |
   | \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
   |  \\    /   O peration     | Version:  5                                     |
   |   \\  /    A nd           | Web:      www.OpenFOAM.org                      |
   |    \\/     M anipulation  |                                                 |
   \*---------------------------------------------------------------------------*/
   FoamFile
   {
       version     2.0;
       format      ascii;
       class       dictionary;
       location    "system";
       object      decomposeParDict;
   }
   // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

   numberOfSubdomains 4;

   method          simple;

   simpleCoeffs
   {
       n               ( 2 2 1 );
       delta           0.001;
   }

   distributed     no;

   roots           ( );


   // ************************************************************************* //


   % cd ..
   % blockMesh
   % decomposePar -force
   % aprun -n 4 -j 1 icoFoam -parallel

Additional Information / References
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `OpenFOAM Home Page <https://openfoam.org>`__
-  `OpenFOAM-5.x GitHub
   repository <https://github.com/OpenFOAM/OpenFOAM-5.x>`__
-  `OpenFOAM-5.x Third-Party library GitHub
   repository <https://github.com/OpenFOAM/ThirdParty-5.x>`__
-  `OpenFOAM User Guide <https://cfd.direct/openfoam/user-guide/>`__
