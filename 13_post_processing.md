**Table of Contents**

- [Post-Processing Scripts](#post-processing-scripts)
  - [Clean Local Database](#clean-local-database)
  - [Process Data and Synthetics](#sec:Process-data-and-syn)
    - [`process_data.pl`](#process_data.pl)
    - [`process_syn.pl`](#sub:process_syn.pl)
    - [`rotate.pl`](#rotate.pl)
    - [`clean_sac_headers_after_crash.sh`](#clean_sac_headers_after_crash.sh)
  - [Map Local Database](#map-local-database)

Post-Processing Scripts
=======================

Several post-processing scripts/programs are provided in the `utils/seis_process` directory, most of which need to be adjusted for different systems, for example, the path of the executable programs. Here we only list the available scripts and provide a brief description, and you can either refer to the related sections for detailed usage or, in many cases, type the script/program name without arguments to see its usage.

Clean Local Database
--------------------

After all the simulations are done, you may need to clean the local scratch disks for the next simulation. This is especially important in the case of 1- or 2-chunk kernel simulations, where very large files are generated for the absorbing boundaries to help with the reconstruction of the regular forward wavefield. A sample script is provided in `utils/Cluster/lsf`:

    cleanbase.pl machines

Process Data and Synthetics
---------------------------

In many cases, the SEM synthetics are calculated and compared to observed seismograms recorded at seismic stations. Since the SEM synthetics are accurate for a certain frequency range, both the original data and the synthetics need to be processed before a comparison can be made. For such comparisons, the following steps are recommended:

1.  Make sure that both synthetic and observed seismograms have the correct station/event and timing information.

2.  Convolve synthetic seismograms with a source time function with the half duration specified in the `CMTSOLUTION` file, provided, as recommended, you used a zero half duration in the SEM simulations.

3.  Resample both observed and synthetic seismograms to a common sampling rate.

4.  Cut the records using the same window.

5.  Remove the trend and mean from the records and taper them.

6.  Remove the instrument response from the observed seismograms (recommended) or convolve the synthetic seismograms with the instrument response.

7.  Make sure that you apply the same filters to both observed and synthetic seismograms. Preferably, avoid filtering your records more than once.

8.  Now, you are ready to compare your synthetic and observed seismograms.

We generally use the following scripts for processing:

### `process_data.pl`

This script cuts a given portion of the original data, filters it, transfers the data into a displacement record, and picks the first P and S arrivals. For more functionality, type ‘`process_data.pl`’ without any argument. An example of the usage of the script:

    process_data.pl -m CMTSOLUTION -s 1.0 -l 0/4000 -i -f -t 40/500 -p -x bp DATA/1999.330*.LH?.SAC

which has resampled the SAC files to a sampling rate of 1 seconds, cut them between 0 and 4000 seconds, transfered them into displacement records and filtered them between 40 and 500 seconds, picked the first P and S arrivals, and added suffix ‘`bp`’ to the file names.

Note that all of the scripts in this section actually use SAC, saclst and/or IASP91 to do the core operations; therefore make sure that the SAC, saclst and IASP91 packages are installed on your system, and that all the environment variables are set properly before running these scripts.

### `process_syn.pl`

This script converts the synthetic output from the SEM code from ASCII to SAC format, and performs similar operations as ‘`process_data.pl`’. An example of the usage of the script:

    process_syn.pl -m CMTSOLUTION -h -a STATIONS -s 1.0 -l 0/4000 -f -t 40/500 -p -x bp SEM/*.MX?.sem

which will convolve the synthetics with a triangular source-time function from the `CMTSOLUTION` file, convert the synthetics into SAC format, add event and station information into the SAC headers, resample the SAC files with a sampling rate of 1 seconds, cut them between 0 and 4000 seconds, filter them between 40 and 500 seconds with the same filter used for the observed data, pick the first P and S arrivals, and add the suffix ‘`bp`’ to the file names.

More options are available for this script, such as adding a time shift to the origin time of the synthetics, convolving the synthetics with a triangular source time function with a given half duration, etc. Type `process_syn.pl` without any argument for detailed usage.

### `rotate.pl`

To rotate the horizontal components of both the data and the synthetics (i.e., `MXN` and `MXE`) to the transverse and radial directions (i.e., `MXT` and `MXR`),` `use `rotate.pl`:
data example:

    rotate.pl -l 0 -L 4000 -d DATA/*.LHE.SAC.bp

synthetics example:

    rotate.pl -l 0 -L 4000 SEM/*.MXE.sem.sac.bp

where the first command performs rotation on the SAC data obtained through IRIS (which may have timing information written in the filename), while the second command rotates the processed synthetics.

For synthetics, another (simpler) option is to set flag `ROTATE_SEISMOGRAMS_RT` to `.true.` in the parameter file `DATA/Par_file`.

### `clean_sac_headers_after_crash.sh`

Note: You need to have the `sismoutil-0.9b` package installed on your computer if you want to run this script on binary SAC files. The software is available via the ORFEUS web site .

In case the simulation crashes during run-time without computing and writing all time steps, the SAC files (if flags `OUTPUT_SEISMOS_SAC_ALPHANUM` or `OUTPUT_SEISMOS_SAC_BINARY` have been set to `.true.`) are corrupted and cannot be used in SAC. If the simulation ran long enough so that the synthetic data may still be of use, you can run the script called `clean_sac_headers_after_crash.sh` (located in the `utils/` directory) on the SAC files to correct the header variable NPTS to the actually written number of time steps. The script must be called from the `SPECFEM3D` main directory, and the input argument to this script is simply a list of SAC seismogram files.

Map Local Database
------------------

A sample program `remap_database` is provided to map the local database from a set of machines to another set of machines. This is especially useful when you want to run mesher and solver, or different types of solvers separately through a scheduler (refer to Chapter [\[cha:Running-Scheduler\]](#cha:Running-Scheduler)).

    run_lsf.bash --gm-no-shmem --gm-copy-env remap_database \
      old_machines 150 [old_jobid new_jobid]

where `old_machines` is the LSF machine file used in the previous simulation, and `150` is the number of processors in total. Note that you need to supply `old_jobid` and `new_jobid(%J)` which are the LSF job-IDs for the old and new run if your databases are stored in a sub-directory named after the jobid on the scratch disk.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Nov 21, 2022)

