**Table of Contents**

- [Noise Cross-correlation Simulations](#noise-cross-correlation-simulations)
  - [New Requirements on ‘Old’ Input Parameter Files](#new-requirements-on-old-input-parameter-files)
  - [Noise Simulations: Step by Step](#noise-simulations-step-by-step)
    - [Pre-simulation](#pre-simulation)
    - [Simulations](#simulations)
    - [Post-simulation](#post-simulation)
  - [Examples](#examples)
  - [References](#references)

Noise Cross-correlation Simulations
===================================

The new version of SPECFEM3D_GLOBE includes functionality for seismic noise tomography. Users are recommended to familiarize themselves first with the procedures for running regular earthquake simulations (Chapters [\[cha:Running-the-Mesher\]](#cha:Running-the-Mesher)–[\[cha:Regional-Simulations\]](#cha:Regional-Simulations)) and adjoint simulations (Chapter [\[cha:Adjoint-Simulations\]](#cha:Adjoint-Simulations)). Also, make sure you read the paper ‘Noise cross-correlation sensitivity kernels’ (Tromp et al. 2010) in order to understand noise simulations from a theoretical perspective.

New Requirements on ‘Old’ Input Parameter Files
-----------------------------------------------

As usual, the three main input files are crucial: `DATA/Par_file`, `DATA/CMTSOLUTION` and `DATA/STATIONS`.
`DATA/CMTSOLUTION` is required for all simulations. However, it may seem unexpected to have it listed here, since the noise simulations should have nothing to do with the earthquake – hence the `DATA/CMTSOLUTION`. For noise simulations, it is critical to have no earthquakes. In other words, the moment tensor specified in `DATA/CMTSOLUTION` must be set to ZERO!
`DATA/STATIONS` remains the same as in previous earthquake simulations, except that the order of stations listed in `DATA/STATIONS` is now important. The order will be used later to identify the ‘main’ receiver, i.e., the one that simultaneously cross correlates with the others. Please be noted that the actual station file used in the simulation is `OUTPUT_FILES/STATIONS_FILTERED`, which is generated when you run your simulations. (e.g., in regional simulations we may have included stations out of the region of our interests in `DATA/STATIONS`, so we have to get rid of them.)
`DATA/Par_file` also requires careful attention. New to this version of SPECFEM3D_GLOBE, a parameter called `NOISE_TOMOGRAPHY` has been added that specifies the type of simulation to be run. `NOISE_TOMOGRAPHY` is an integer with possible values 0, 1, 2 and 3. For example, when `NOISE_TOMOGRAPHY` equals 0, a regular earthquake simulation will be run. When `NOISE_TOMOGRAPHY` is equal to 1/2/3, you are about to run step 1/2/3 of the noise simulations respectively. Should you be confused by the three steps, refer to Tromp et al. (2010) for details.
Another change to `DATA/Par_file` involves the parameter `RECORD_LENGTH_IN_MINUTES`. While for regular earthquake simulations this parameter specifies the length of synthetic seismograms generated, for noise simulations it specifies the length of the seismograms used to compute cross correlations. The actual cross correlations are thus twice this length. The code automatically makes modification accordingly, if `NOISE_TOMOGRAPHY` is not zero.
There are other parameters in `DATA/Par_file` which should be given specific values. For instance, `NUMBER_OF_RUNS` and `NUMBER_OF_THIS_RUN` must be 1; `ROTATE_SEISMOGRAMS_RT`, `SAVE_ALL_SEISMOGRAMS_IN_ONE_FILES`, `USE_BINARY_FOR_LARGE_FILE` and `MOVIE_COARSE` should be `.false.`. Moreover, since the first two steps for calculating noise cross-correlation kernels correspond to forward simulations, `SIMULATION_TYPE` must be 1 when `NOISE_TOMOGRAPHY` equals 1 or 2. Also, we have to reconstruct the ensemble forward wavefields in adjoint simulations, therefore we need to set `SAVE_FORWARD` to `.true.` for the second step, i.e., when `NOISE_TOMOGRAPHY` equals 2. The third step is for kernel constructions. Hence `SIMULATION_TYPE` should be 3, whereas `SAVE_FORWARD` must be `.false.`.

Noise Simulations: Step by Step
-------------------------------

Proper parameters in those ‘old’ input files are not enough for noise simulations to run. We have a lot more ‘new’ input parameter files to specify: for example, the ensemble-averaged noise spectrum, the noise distribution etc. However, since there are a few ‘new’ files, it is better to introduce them sequentially. Read through this section even if you don’t understand some parts temporarily, since some examples you can go through are provided in this package.

### Pre-simulation

- As usual, we first configure the software package using:

      ./configure FC=ifort MPIFC=mpif90

- Next, we need to compile the source code using:

      make xmeshfem3D
      make xspecfem3D

  Before compilation, the `DATA/Par_file` must be specified correctly, e.g., `NOISE_TOMOGRAPHY` shouldn’t be zero; `RECORD_LENGTH_IN_MINUTES`, `NEX_XI` and `NEX_ETA` must be what you want in your real simulations. Otherwise you may get wrong informations which will cause problems later. (it is good to always re-complie the code before you run simulations)

- After compiling, you will find two important numbers besides the needed executables:

      number of time steps = 31599
      time_stepping of the solver will be: 0.19000

  The first number will be denoted as `NSTEP` from now on, and the second one as `dt`. The two numbers are essential to calculate the ensemble-averaged noise spectrum from either Peterson’s noise model or just a simple flat power spectrum (corresponding to 1-bit preprocessing). Should you miss the two numbers, you can run `./xcreate_header_file` to bring them up again (with correct `DATA/Par_file`!). FYI, `NSTEP` is determined by `RECORD_LENGTH_IN_MINUTES` in `DATA/Par_file`, which is automatically doubled in noise simulations; whereas `dt` is derived from `NEX_XI` and `NEX_ETA`, or in other words your element sizes.

- A Matlab script is provided to generate the ensemble-averaged noise spectrum.

      EXAMPLES/noise_examples/NOISE_TOMOGRAPHY.m  (main program)
      EXAMPLES/noise_examples/PetersonNoiseModel.m

  In Matlab, simply run:

      NOISE_TOMOGRAPHY(NSTEP, dt, Tmin, Tmax, NOISE_MODEL)

  `NSTEP` and `dt` have been given when compiling the specfem3D source code; `Tmin` and `Tmax` correspond to the period range you are interested in; `NOISE_MODEL` denotes the noise model you will be using. Details can be found in the Matlab script.

  After running the Matlab script, you will be given the following information (plus a figure in Matlab):

      *************************************************************
      the source time function has been saved in:
      /data2/yangl/3D_NOISE/S_squared         (note this path must be different)
      S_squared should be put into directory:
      ./NOISE_TOMOGRAPHY/ in the SPECFEM3D_GLOBE package

  In other words, the Matlab script creates a file called `S_squared`, which is the first ‘new’ input file we encounter for noise simulations.

  One may choose a flat noise spectrum rather than Peterson’s noise model. This can be done easily by modifying the Matlab script a little bit.

- Create a new directory in the SPECFEM3D_GLOBE package, name it as `NOISE_TOMOGRAPHY`. In fact, this new directory should have been created already when checking out the package. We will add/replace some information needed in this folder.

- Put the Matlab-generated-file `S_squared` in `NOISE_TOMOGRAPHY`.

  That’s to say, you will have a file `NOISE_TOMOGRAPHY/S_squared` in the SPECFEM3D_GLOBE package.

- Create a file called `NOISE_TOMOGRAPHY/irec_main_noise`. Note that this file should be put in directory `NOISE_TOMOGRAPHY` as well. This file contains only one integer, which is the ID of the ‘main’ receiver. For example, if in this file shows 5, it means that the fifth receiver listed in `DATA/STATIONS` becomes the ‘main’. That’s why we mentioned previously that the order of receivers in `DATA/STATIONS` is important.

  Note that in regional (1- or 2-chunk) simulations, the `DATA/STATIONS` may contain receivers not within the selected chunk(s). Therefore, the integer in `NOISE_TOMOGRAPHY/irec_main_noise` is actually the ID in `DATA/STATIONS_FILTERED` (which is generated by `xspecfem3D`).

- Create a file called `NOISE_TOMOGRAPHY/nu_main`. This file holds three numbers, forming a (unit) vector. It describes which component we are cross-correlating at the ‘main’ receiver, i.e., ${\hat{\bf \nu}}^{\alpha}$ in Tromp et al. (2010). The three numbers correspond to N/E/Z components respectively.

- Describe the noise direction and distributions in `src/specfem3d/noise_tomography.f90`. Search for a subroutine called `noise_distribution_direction` in `noise_tomography.f90`. It is actually located at the very beginning of `noise_tomography.f90`. The default assumes vertical noises and a uniform distribution across the whole physical domain. It should be quite self-explanatory for modifications. Should you modify this part, you have to re-compile the source code. (again, that’s why we recommend that you always re-compile the code before you run simulations)

### Simulations

As discussed in Tromp et al. (2010), it takes three simulations to obtain one contribution of the ensemble sensitivity kernels:

- Step 1: simulation for generating wavefield

      SIMULATION_TYPE = 1
      NOISE_TOMOGRAPHY = 1
      SAVE_FORWARD  not used, can be either .true. or .false.

- Step 2: simulation for ensemble forward wavefield

      SIMULATION_TYPE = 1
      NOISE_TOMOGRAPHY = 2
      SAVE_FORWARD = .true.

- Step 3: simulation for ensemble adjoint wavefield and sensitivity kernels

      SIMULATION_TYPE = 3
      NOISE_TOMOGRAPHY = 3
      SAVE_FORWARD = .false.

  Note Step 3 is an adjoint simulation, please refer to previous chapters on how to prepare adjoint sources and other necessary files, as well as how adjoint simulations work.

It’s better to run the three steps continuously within the same job, otherwise you have to collect the saved surface movies from the old nodes to the new nodes.

### Post-simulation

After those simulations, you have all stuff you need, either in the `OUTPUT_FILES` or in the directory specified by `LOCAL_PATH` in `DATA/Par_file` (most probably on local nodes). Collect whatever you want from the local nodes to your workstation, and then visualize them. This process is the same as what you may have done for regular earthquake simulations. Refer Chapter [\[cha:graphics\]](#cha:graphics) if you have problems.
Simply speaking, two outputs are the most interesting: the simulated ensemble cross correlations and one contribution of the ensemble sensitivity kernels.
The simulated ensemble cross correlations are obtained after the second simulation (Step 2). Seismograms in `OUTPUT_FILES` are actually the simulated ensemble cross correlations. Collect them immediately after Step 2, or the Step 3 will overwrite them. Note that we have a ‘main’ receiver specified by `NOISE_TOMOGRAPHY/irec_main_noise`, the seismogram at one station corresponds to the cross correlation between that station and the ‘main’. Since the seismograms have three components, we may obtain cross correlations for different components as well, not necessarily the cross correlations between vertical components.
One contribution of the ensemble cross-correlation sensitivity kernels are obtained after Step 3, stored in the `LOCAL_PATH` on local nodes. The ensemble kernel files are named the same as regular earthquake kernels.
You need to run another three simulations to get the other contribution of the ensemble kernels, using different forward and adjoint sources given in Tromp et al. (2010).

Examples
--------

In order to illustrate noise simulations in an easy way, three examples are provided in `EXAMPLES/noise_examples/`. Note however that they are created for a specific cluster (SESAME@PRINCETON). You have to modify them to fit your own cluster.

The three examples can be executed using (in directory `EXAMPLES/noise_examples/`):

    ./run_this_example.sh regional
    ./run_this_example.sh global_short
    ./run_this_example.sh global_long

Each corresponds to one example, but they are pretty similar.

Although the job submission only works on SESAME@PRINCETON, the procedure itself is universal. You may review the whole process described in the last section by following commands in those examples.

Finally, note again that those examples show only one contribution of the ensemble kernels!

References
----------

Tromp, Jeroen, Yang Luo, Shravan Hanasoge, and Daniel Peter. 2010. “Noise Cross-Correlation Sensitivity Kernels.” *Geophys. J. Int.* 183: 791–819. <https://doi.org/10.1111/j.1365-246X.2010.04721.x>.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Nov  9, 2022)

