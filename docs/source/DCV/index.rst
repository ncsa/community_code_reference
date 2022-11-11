DCV
===

**NICE** **D**\ esktop **C**\ loud **V**\ isualization (DCV,
https://www.nice-software.com/products/dcv) is a VNC server optimized
for high-performance OpenGL applications such as the molecular
visualization program VMD. DCV runs on a single XK node in cluster
compatibility mode.

Client Software for Connecting to Blue Waters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You must download and install `NICE DCV 2016.0
Endstation <https://download.nice-dcv.com/2016-0.html>`__ on your
desktop or laptop to connect to a Blue Waters XK node with accelerated
OpenGL performance. Earlier versions of NICE DCV Endstation appear to
work with acceptable performance. Other VNC clients may also connect but
OpenGL performance will be very poor. **NICE DCV 2017.0 uses a different
protocol that is incompatible with the DCV server software on Blue
Waters.**

Running DCV Endstation on MacOS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is a problem with the DCV Endstation software that prevents it
from launching on some versions of MacOS. If the DCV Endstation software
quits immediately after launching, try removing the ``libz.1.dylib``
file from the ``DCV Endstation.app/Contents/Frameworks`` folder in the
DCV Endstation application package. You can do this from the Finder by
control-clicking on the DCV Endstation application to select "Show
Package Contents", then navigating to the Contents/Frameworks folder and
dragging the file out of the application.

Establishing Port Tunnels
~~~~~~~~~~~~~~~~~~~~~~~~~

To enable DCV Endstation to connect to your interactive session on a
Blue Waters XK compute node you will need to ssh-tunnel ports 5901 and
7300-7310 (more or less, each OpenGL window will consume a port starting
at 7300) through a Blue Waters login node. On a Linux or Mac client the
easiest way to accomplish this is to create a script (named, e.g.,
bwforward) with the following (replace **username** with your Blue
Waters login):

::

   #!/bin/bash

   if (( $# != 1 )); then
     echo "usage: $0 <internal ip address>"
     exit -1
   fi

   HOSTIP=$1

   fwds="-L 5901:$HOSTIP:5901"
   for (( i=7300; $i<=7310; i=$i+1 )); do
     fwds="$fwds -L $i:$HOSTIP:$i"
   done

   cmd="ssh -v $fwds username@bw-duo.ncsa.illinois.edu"
   echo $cmd
   $cmd

Launching an Interactive Visualization Job on Blue Waters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, launch an interactive job on a single XK (GPU-accelerated)
compute node:

::

   qsub -I -lgres=ccm -lnodes=1:ppn=16:xk -lwalltime=8:00:00

If you require more memory than the 32GB on a regular XK node you may
want to use a himem node, after using showbf to confirm that one is
available and for how long:

::

   showbf -f xkhimem
   qsub -I -lgres=ccm -lnodes=1:ppn=16:xkhimem -lwalltime=8:00:00

When your interactive job starts (this may take ten minutes or more
depending on the scheduler iteration time), you will be connected to a
shell on a shared "mom" node, from which you must connect to your
assigned compute node as follows:

::

   module load ccm
   ccmlogin

After connecting to the login node you must launch the DCV server as
follows:

::

   module load dcv
   dcvlaunch -r 1920x1080

The first time you launch DCV it will ask you to set a password.

The dcvlaunch script will print the internal (private) IP address of the
compute node. You must open an ssh tunnel for ports 5901 and 7300...
from your client desktop or laptop to the compute node through a login
node. This can be done through the bwforward script given above.

A full transcript of launching a DCV session is shown below.

::

   username@h2ologin4:~/dcv> qsub -I -lgres=ccm -lnodes=1:ppn=16:xk -lwalltime=2:00:00
   INFO: Job submitted to account: xxx
   qsub: waiting for job 8311609.bw to start
   qsub: job 8311609.bw ready

   ----------------------------------------
   Begin Torque Prologue on nid25431
   at Wed Mar 14 16:57:29 CDT 2018
   Job Id:            8311609.bw
   Username:        username
   Group:            usergroup
   Job name:        STDIN
   Requested resources:    gres=ccm,neednodes=1:ppn=16:xk,nodes=1:ppn=16:xk,walltime=02:00:00
   Queue:            normal
   Account:        xxx
   End Torque Prologue:  0.359 elapsed
   ----------------------------------------
   In CCM JOB:  8311609.bw  JID  8311609  USER  username  GROUP  usergroup  WLM  torque
   Initializing CCM environment, Please Wait
   Warning: The -E option is deprecated and has no effect
   CCM Start success, 1 of 1 responses
   Directory: /u/sciteam/username
   Wed Mar 14 16:57:35 CDT 2018
   username@nid25431:~> module load ccm
   username@nid25431:~> ccmlogin
   username@nid18415:~> module load dcv
   username@nid18415:~> dcvlaunch -r 1920x1080
   Starting X server

   Display Info: 10.128.72.128:5901

Leave the terminal connected, as killing it will terminate the
interactive job and your dcv session.

In order to connect to this session from a client (i.e., a laptop or
desktop) one would first establish the required ssh tunnels by running
"``bwforward 10.128.72.128``" **on the client** and authenticating with
your PIN and token code.

With the tunnels established you can then launch the NICE DCV Endstation
software. In the dialogue, for "VNC Server" enter "``localhost:1``" and
press the Connect button. At the "Authentication Credentials" prompt the
username will be grayed out. Enter the password you chose the first time
you ran dcvlaunch and the desktop session should open in a new window.
Do not be concerned if the viewer reports that encryption is disabled,
as the ssh tunnel is encrypted.

Exiting an Interactive Visualization Session
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may disconnect and reconnect to your visualization session as many
times as you like. Exiting the window manager from within the viewer
will kill the DCV server, but not your interactive job. The DCV server
can be re-launched if it is killed. In order to kill the interactive job
completely, you can use Control-D to exit first from the shell on the
compute node, and then from the shell on the mom node. You can also run
"``qdel jobid``" from any shell to kill the job.

Running VMD for Interactive Visualization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

VMD (Visual Molecular Dynamics, http://www.ks.uiuc.edu/Research/vmd) is
a powerful molecular visualization and analysis package that relies on
OpenGL for graphics acceleration.

Once you have connected to your DCV session, open a terminal and run

::

   module load vmd
   vmd

This will launch the pre-installed VMD optimized for OpenGL
visualization. VMD will initialize a CUDA context on launch. Additional
copies of VMD launched while the first is still running will report CUDA
errors during startup but continue to run without CUDA acceleration (see
explanation below). VMD CUDA initialization can be prevented by defining
the ``VMDNOCUDA`` environment variable. The lengthy Optix raytracer
initialization can likewise be eliminated by defining ``VMDNOOPTIX``.

Limitations on CUDA Programs Due to Exclusive Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The GPUs on Blue Waters are set to the compute mode "Exclusive Process",
which allows only one CUDA context per device, although that context can
be accessed by multiple threads within the same process. As a result, if
a process has accessed the GPU via CUDA any other process attempting to
use CUDA will receive an error. This is different from the compute mode
"Default" that is most commonly encountered, in which multiple processes
can share the GPU freely, although with a loss of performance if their
actual use overlaps.

Note that this restriction applies only to CUDA (and likely other GPU
compute environments such as OpenACC and OpenCL). It specifically does
not apply to OpenGL, so multiple OpenGL programs can run simultaneously
(unless they also use CUDA).

In order to deal with this limitation, the ``dcvlaunch`` script sets
environment variables that restrict DCV from using video compression
modes that employ CUDA acceleration. No significant performance loss
from this restriction has been observed.
