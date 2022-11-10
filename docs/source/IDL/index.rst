IDL
=========

IDL provides data analysis and visualization for small to medium size
datasets. IDL does not operate in parallel and as such all individual
analysis tasks must fit on a single node. That is, data to processed
must fit within the RAM of a single node. Data that can be analyzed
separately and do not require inter process communication can be spread
across several nodes and processed independently. Time series data often
has this property.

Using IDL on Blue Waters
------------------------

Use of IDL on the login nodes is discouraged as is all data analyis. To
use IDL on Blue Waters it is recommended that an interactive single node
partition be obtained and the application run on that partition. One can
also run IDL as a batch job on the compute nodes by selecting off screen
buffering of the images and saving them to disk as they are created. See
the example given below.

To use IDL interactively, load the module file "module load idl". Then
start the command line interpreter by typing the command "idl". This
will give you a command prompt. Type idl commands at the prompt.
Plotting windows appear on you workstation or laptop if you are running
an x windows session. Microsoft windows users will have to obtain some
sort of X emulation for this to work.

Scripting IDL from compute nodes.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

IDL can be used from compute nodes by creating a file with idl commands
in it. For example place the following commands in a file called
test.com.

::

   data=sin(2.0*findgen(200)*!pi/25.0)*exp(-0.02*findgen(200))
   myplot = plot(data,BUFFER=1)
   myplot.save, "wiggle.png", HEIGHT=480,WIDTH=640
   exit

Notice the "BUFFER=1" argument to the plot command. This argument
causes the plot command to not open a window for plotting and to plot
off screen. Once this test file has been created one can call it in a
pbs batch script in the following way.

::

      #!/bin/bash
      #PBS -j oe
      #PBS -V
      #PBS -N idltest
      #PBS -l nodes=1:ppn=1
      #PBS -l walltime=00:05:00
      . ${MODULESHOME}/init/bash
      module load idl

      date

      cd $PBS_O_WORKDIR

      aprun -B idl ./test.com

This pbs script runs the idl commands whyc create the wiggle.png
file that contains the graphics output.
