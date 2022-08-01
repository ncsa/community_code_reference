# CM1

### A. Description

CM1 is a three-dimensional, time-dependent, non-hydrostatic numerical
model that has been developed primarily by George Bryan at The
Pennsylvania State University (PSU) (circa 2000-2002) and at the
National Center for Atmospheric Research (NCAR) (2003-present). CM1 is
designed primarily for idealized research, particularly for deep
precipitating convection (i.e., thunderstorms). CM1 is freely available
to anybody that wants to use it, though restrictions on distribution
[apply](http://www2.mmm.ucar.edu/people/bryan/cm1/getcode.html). See CM1
home page┬á<http://www2.mmm.ucar.edu/people/bryan/cm1/> for additional
information.

These instructions were tested with CM1 version 18 and 20.1. They may
work with other versions as well.

### B. How to download CM1

Please follow the instructions on the CM1 website where you have to
agree to the CM1 license
<http://www2.mmm.ucar.edu/people/bryan/cm1/getcode.html>.

Once you have downloaded the source tarball, do:

    tar xf cm1r18.tar.gz

or

    tar xf cm1r20.1.tar.gz

### C. How to build CM1

The CM1 website describes building it in
[detail](http://www2.mmm.ucar.edu/people/bryan/cm1/user_guide_brief.html)
and should be taken as authorative, this page for the most part
summarizes what is described there.

On Cray XE6 platform, CM1 can be built under the Cray environments as
described in the Makefile. You can enable both NetCDF or HDF5 output by
uncommenting the OUTPUTOPT option in the respective section of the
Makefile in src/Makefie.┬á Uncomment the compiler options in the Blue
Waters section (search for bluewaters) and remove -lfast_mv
from┬áLINKOPTS since the libfast module is no longer supported. For
version 18 you will also have to fix param.F in subroutine param, where
the write directive is used incorrectly. For version 20.1 you will have
to allow up to 1023 columns per line and reduce optimization level for
domaindiag.F to avoid compiler failures.

    module load PrgEnv-cray
    module load cray-netcdf

    cd cm1r*/src

    # fix up param.F for version 18 by removing a "," after write
    sed -i 's/write(outfile,[*]),/write(outfile,*) /' param.F

    # reduce optimization level for domaindiag.F (\t is a tab that make requires)
    echo -e >>Makefile '
    domaindiag.o: domaindiag.F
    \t$(CPP) $(DM) $(DP) $(ADV) $(OUTPUTOPT) $*.F > $*.f90
    \t$(FC) $(FFLAGS) $(OUTPUTINC) -O2 -Ovector0 -c $*.f90

    hifrq.o: hifrq.F
    \t$(CPP) $(DM) $(DP) $(ADV) $(OUTPUTOPT) $*.F > $*.f90
    \t$(FC) $(FFLAGS) $(OUTPUTINC) -O1 -c $*.f90
    '


    # this sets the options on the command line rather than edit the Makefile
    # these options are for a pure MPI build (no OpenMP)
    # There seem to be some missing dependency declarations in the Makefile
    # leading to make aborting if build in parallel. You can use higher values
    # but might have to restart make if an error is detected.
    make FC=ftn OPTS="-I../include -O3 -Ovector3 -Oscalar3 -Othread3 -h noomp -N 1023" \
      LINKOPTS="" CPP="cpp -C -P -traditional" DM=-DMPI -j1

Successful compilation will create a cm1.exe binary file in the run
directory, one level up from src.

The compilation was conducted on 2020-09-28 under the following
environment:

    Currently Loaded Modulefiles:
      1) modules/3.2.10.4                      17) PrgEnv-cray/5.2.82
      2) eswrap/1.3.3-1.020200.1280.0          18) cray-mpich/7.7.4
      3) cce/8.7.7                             19) craype-interlagos
      4) craype-network-gemini                 20) torque/6.0.4
      5) craype/2.5.16                         21) moab/9.1.2-sles11
      6) cray-libsci/18.12.1                   22) openssh/7.5p1
      7) udreg/2.3.2-1.0502.10518.2.17.gem     23) xalt/0.7.6.local
      8) ugni/6.0-1.0502.10863.8.28.gem        24) scripts
      9) pmi/5.0.14                            25) OpenSSL/1.0.2m
     10) dmapp/7.0.1-1.0502.11080.8.74.gem     26) cURL/7.59.0
     11) gni-headers/4.0-1.0502.10859.7.8.gem  27) git/2.17.0
     12) xpmem/0.1-2.0502.64982.5.3.gem        28) wget/1.19.4
     13) dvs/2.5_0.9.0-1.0502.2188.1.113.gem   29) user-paths
     14) alps/5.2.4-2.0502.9774.31.12.gem      30) gnuplot/5.0.5
     15) rca/1.0.0-2.0502.60530.1.63.gem       31) darshan/3.1.3
     16) atp/2.0.4                             32) cray-netcdf/4.6.1.3

D. Sample test

CM1 provides some sample namelist.input files on its
[website](http://www2.mmm.ucar.edu/people/bryan/cm1/namelists/) (r18) or
in the tarball (r20.1). To run the supercell case do

    cd .. # now in main directory
    cp -r run ~/scratch/supercell
    rm ~/scratch/supercell/namelist.input
    # for r18
    wget -O ~/scratch/supercell/namelist.input \
      http://www2.mmm.ucar.edu/people/bryan/cm1/namelists/namelist.input.supercell.r18
    # for 20.1
    cp run/config_files/supercell/namelist.input ~/scratch/supercell/
    cd ~/scratch/supercell

To reduce runtime to about 5 minutes, you should reduce the timax
setting in namelist.input to 480 to stop after integrating 8 minutes of
time.

    sed -i 's/timax[^,]*,/timax = 480.,/' namelist.input

Then create a run.pbs file having the following content:

    #!/bin/bash
    #PBS -l nodes=1:ppn=32:xe
    #PBS -l walltime=00:05:00
    #PBS -q debug
    #PBS -N supercell
    cd $PBS_O_WORKDIR
    aprun ./cm1.exe >cm1.out 2>cm1.err

Submit the job

    qsub run.pbs

which will produce a file cm1.out which will contain lines like this

       stattim =  60.
                 0              0.000000 min
       Entering writeout ...
       nwrite =  1
       nloop =  1
       Opening cm1out_s.dat
     51,  1 rain
     51,  2 prate
     51,  3 sws
     51,  4 svs
     51,  5 sps

### E. Known issues

For some (high resolution input), the model can┬áhang without giving an
error message, until its time in the queue runs out.┬á To our knowledge,
it only occurs when one has chosen to use the Morrison microphysics
routine.

Here's what the output file looked like when that was happening:

    --------------------
       nwrite =  1
    Test_10m_0_000001_s.dat
    Test_10m_0_000001_u.dat
    Test_10m_0_000001_v.dat
    Test_10m_0_000001_w.dat
         2d vars
         s vars
    -----------------

It turns out that for the Morrison scheme in CM1, there is a check for
numerical convergence in the saturation adjustment routine, and if that
is not met, then it prints out this error message and stops the model:

    print *
    ┬á print *,' Convergence cannot be reached in satadj2 subroutine.'
    ┬á print *
    ┬á print *,' This may be a problem with the algorithm in satadj2.'
    ┬á print *,' However, the model may have became unstable somewhere'
    ┬á print *,' else and the symptoms first appeared here.'
    ┬á print *
    ┬á print *,' Try decreasing the timestep (dtl and/or nsound).'
    ┬á print *
    ┬á print *,'┬á ... stopping cm1 ... '
    ┬á print *

For some reason, at some resolutions this bit of code doesn\'t get
executed, and things just hang, \"spinning\" down time in the queue. ┬áTo
fix the problem (not the code), one must just increase his/her┬ávalue of
the variable \'nsound\' in the namelist.input file. ┬áIf the user has
already maxed that out (I think a value up to 12 is allowed), then they
must decrease the main timestep (\'dtl\') in namelist.input.
