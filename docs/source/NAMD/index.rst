NAMD
====

Description
-----------

NAMD is a parallel molecular dynamics code designed for high-performance
simulation of large biomolecular systems. It is written and maintained
by the Theoretical and Computational Biophysics Group at the University
of Illinois at Urbana–Champaign. More details can be found
at http://www.ks.uiuc.edu/Research/namd/.

Quick-Start
-----------

The easiest way to get started running NAMD on Blue Waters is to run a
pre-compiled version of the code. Jim Phillips, lead NAMD developer,
maintains a version of NAMD in his home directory which you may use. The
latest development version can be found in
/u/sciteam/jphillip/NAMD_build.latest/. There are 7 different versions
of NAMD to choose from, based on the size of the simulation and whether
or not you want to use GPUs in your simulation. 

Jim also provides some scripts which will build and submit your
jobscript in /u/sciteam/jphillip/NAMD_scripts/runbatch*. Again, there
are several versions, differing in which of the 7 different versions of
NAMD they use. The "standard" mode would be
"/u/sciteam/jphillip/NAMD_scripts/runbatch". Running it with no options
gives a short description of how you would submit a simulation run. If
desired, you can look at the files it generates to understand how to
customize your own job script.  

Jim Phillips provides a tool called runbatch that generates and runs
a job script for you. 

There are 4 variations: 

XE CPU run: 

::

   ~jphillip/NAMD_scripts/runbatch

XK GPU run: 

::

   ~jphillip/NAMD_scripts/runbatch_cuda

Memory-optimized XE CPU run (requires memory-optimized input file preparation): 

::

   jphillip/NAMD_scripts/runbatch_memopt

Memory-optimized XK GPU run (memory-optimized input):

::

   ~jphillip/NAMD_scripts/runbatch_memopt_cuda

Run each command without options to see the list of input parameters 

For example, to simulate the system described in mysim.namd using 400 XE nodes, I would run: 

::

   ~jphillip/NAMD_scripts/runbatch mysim.namd mysim.log 400 4 normal

When you run the command, it will generate a script, which is output
to the screen, and submit that script as a job. To capture the script
so that you can customize it, you can redirect the output of runbatch
and edit out everything before "#!/bin/tcsh" and everything after the
final "aprun ..." line. 

Building NAMD
-------------

#. Obtain the NAMD source code from
   `here <http://www.ks.uiuc.edu/Development/Download/download.cgi?PackageName=NAMD>`__. I
   recommend either the 2.10 or newer version, or the nightly source
   release. If you use the NAMD 2.9 release, you should download a more
   recent version of Charm++ than the one included in the NAMD 2.9
   release that fixes a number of BW-related bugs. 
#. Unpack NAMD and cd to the extracted directory 
#. Extract NAMD source:

   tar zxvf NAMD_CVS-2014-11-12_Source.tar.gz 

#. Go to extracted directory:

   cd NAMD_CVS-2014-11-12_Source

#. Obtain a tweaked version of the Tcl library:

   wget http://www.ks.uiuc.edu/Research/namd/libraries/tcl8.5.9-crayxe-threaded.tar.gz

#. Unpack the Tcl library and set up a link to the extracted directory:

   tar zxvf tcl8.5.9-crayxe-threaded.tar.gz; ln -s tcl8.5.9-crayxe-threaded ./tcl

#. Select the proper modules for compilation:

   module swap PrgEnv-cray PrgEnv-gnu; module load fftw rca craype-hugepages8M

#. Extract the version of Charm++ included in the NAMD source distribution:

   tar xvf charm-6.6.1.tar

#. Build charm++:

   cd charm-6.6.1; ./build charm++ gni-crayxe smp persistent -j16 --with-production; cd .. 

#. Build NAMD:
   
::

   ./config CRAY-XE-gnu --with-fftw --with-fftw3 --fftw-prefix $FFTW_DIR/.. --charm-arch gni-crayxe-persistent-smp; cd CRAY-XE-gnu/; gmake -j16
