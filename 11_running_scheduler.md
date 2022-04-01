**Table of Contents**

-   [Running through a Scheduler](#running-through-a-scheduler)
    -   [`run_lsf.bash`](#run_lsfbash)
    -   [`go_mesher_solver_lsf_globe.bash`](#go_mesher_solver_lsf_globebash)
    -   [`run_lsf.kernel` and `go_mesher_solver_globe.kernel`](#run_lsfkernel-and-go_mesher_solver_globekernel)

Running through a Scheduler
===========================

The code is usually run on large parallel machines, often PC clusters, most of which use schedulers, i.e., queuing or batch management systems to manage the running of jobs from a large number of users. The following considerations need to be taken into account when running on a system that uses a scheduler:

-   The processors/nodes to be used for each run are assigned dynamically by the scheduler, based on availability. Therefore, in order for the mesher and the solver (or between successive runs of the solver) to have access to the same database files (if they are stored on hard drives local to the nodes on which the code is run), they must be launched in sequence as a single job.

-   On some systems, the nodes to which running jobs are assigned are not configured for compilation. It may therefore be necessary to pre-compile both the mesher and the solver. A small program provided in the distribution called `create_header_file.f90` can be used to directly create` OUTPUT_FILES/values_from_mesher.h` using the information in the `DATA/Par_file` without having to run the mesher (type `` `make ``<span> </span>`create_header_`
    `file`’ to compile it and ‘`./bin/xcreate_header_file`’ to run it; refer to the sample scripts below). The solver can now be compiled as explained above.

-   One feature of schedulers/queuing systems is that they allow submission of multiple jobs in a “launch and forget” mode. In order to take advantage of this property, care needs to be taken that output and intermediate files from separate jobs do not overwrite each other, or otherwise interfere with other running jobs.

We describe here in some detail a job submission procedure for the Caltech 1024-node cluster, CITerra, under the LSF scheduling system. We consider the submission of a regular forward simulation. The two main scripts are `run_lsf.bash`, which compiles the Fortran code and submits the job to the scheduler, and `go_mesher_solver_lsf`
`.bash`, which contains the instructions that make up the job itself. These scripts can be found in `utils/Cluster/lsf` directory and can straightforwardly be modified and adapted to meet more specific running needs.

`run_lsf.bash`
--------------

This script first sets the job queue to be ‘normal’. It then compiles the mesher and solver together, figures out the number of processors required for this simulation from `DATA/Par_file`, and submits the LSF job.

    #!/bin/bash
    # use the normal queue unless otherwise directed queue="-q normal"
    if [ $# -eq 1 ]; then
      echo"Setting the queue to $1"
      queue="-q $1"
    fi
    # compile the mesher and the solver
    d=`date`
    echo"Starting compilation $d"

    make clean
    make meshfem3D
    make create_header_file

    ./bin/xcreate_header_file

    make specfem3D

    d=`date`
    echo"Finished compilation $d"

    # compute total number of nodes needed
    NPROC_XI=`grep ^NPROC_XI DATA/Par_file | cut -c 34- `
    NPROC_ETA=`grep ^NPROC_ETA DATA/Par_file | cut -c 34- `
    NCHUNKS=`grep ^NCHUNKS DATA/Par_file | cut -c 34- `

    # total number of nodes is the product of the values read
    numnodes=$(( $NCHUNKS * $NPROC_XI * $NPROC_ETA ))

    echo "Submitting job"
    bsub $queue -n $numnodes -W 60 -K <go_mesher_solver_lsf_globe.bash

`go_mesher_solver_lsf_globe.bash`
---------------------------------

This script describes the job itself, including setup steps that can only be done once the scheduler has assigned a job-ID and a set of compute nodes to the job, the `run_lsf.bash` commands used to run the mesher and the solver, and calls to scripts that collect the output seismograms from the compute nodes and perform clean-up operations.

1.  First the script directs the scheduler to save its own output and output from `stdout` into `OUTPUT_FILES/%J.o`, where `%J` is short-hand for the job-ID; it also tells the scheduler what version of `mpich` to use (`mpich_gm`) and how to name this job (`go_mesher_solver_lsf`).

2.  The script then creates a list of the nodes allocated to this job by echoing the value of a dynamically set environment variable `LSB_MCPU_HOSTS` and parsing the output into a one-column list using the Perl script `utils/Cluster/lsf/remap_lsf_machines.pl`. It then creates a set of scratch directories on these nodes (`/scratch/`
    `$USER/DATABASES_MPI`) to be used as the `LOCAL_PATH` for temporary storage of the database files. The scratch directories are created using `shmux`, a shell multiplexor that can execute the same commands on many hosts in parallel. `shmux` is available from Shmux . Make sure that the `LOCAL_PATH` parameter in `DATA/Par_file` is also set properly.

3.  The next portion of the script launches the mesher and then the solver using `run_lsf.bash`.

4.  The final portion of the script performs clean up on the nodes using the Perl script `cleanmulti.pl`

<!-- -->

    #!/bin/bash -v
    #BSUB -o OUTPUT_FILES/%J.o
    #BSUB -a mpich_gm
    #BSUB -J go_mesher_solver_lsf

    BASEMPIDIR=/scratch/$USER/DATABASES_MPI
    echo "$LSB_MCPU_HOSTS" > OUTPUT_FILES/lsf_machines
    echo "$LSB_JOBID" > OUTPUT_FILES/jobid

    ./remap_lsf_machines.pl OUTPUT_FILES/lsf_machines > OUTPUT_FILES/machines

    # Modif : create a directory for this job
    shmux -M50 -Sall \
     -c "mkdir -p /scratch/$USER;mkdir -p $BASEMPIDIR.$LSB_JOBID" - < OUTPUT_FILES/machines >/dev/null

    # Set the local path in Par_file
    sed -e "s:^LOCAL_PATH .*:LOCAL_PATH = $BASEMPIDIR.$LSB_JOBID:" < DATA/Par_file > DATA/Par_file.tmp
    mv DATA/Par_file.tmp DATA/Par_file

    current_pwd=$PWD

    mpirun.lsf --gm-no-shmem --gm-copy-env $current_pwd/bin/xmeshfem3D
    mpirun.lsf --gm-no-shmem --gm-copy-env $current_pwd/bin/xspecfem3D

    # clean up
    cleanbase_jobid.pl OUTPUT_FILES/machines DATA/Par_file

`run_lsf.kernel` and `go_mesher_solver_globe.kernel`
----------------------------------------------------

For kernel simulations, you can use the sample run scripts `run_lsf.kernel` and `go_mesher_solver_globe`
`.kernel` provided in `utils/Cluster` directory, and modify the command-line arguments of `xcreate_adjsrc_traveltime` in `go_mesher_solver_globe.kernel` according to the start and end time of the specific portion of the forward seismograms you are interested in.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Apr  1, 2022)

