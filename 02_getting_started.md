**Table of Contents**

-   [Getting Started](#getting-started)
    -   [Configuring and compiling the source code](#configuring-and-compiling-the-source-code)
    -   [Using the GPU version of the code](#using-the-gpu-version-of-the-code)
    -   [Adding OpenMP support in addition to MPI](#adding-openmp-support-in-addition-to-mpi)
    -   [Configuration summary](#configuration-summary)
    -   [Compiling on an IBM BlueGene](#compiling-on-an-ibm-bluegene)
    -   [Compiling on an Intel Xeon Phi (Knights Landing KNL)](#compiling-on-an-intel-xeon-phi-knights-landing-knl)
    -   [Using a cross compiler](#using-a-cross-compiler)
    -   [Visualizing the subroutine calling tree of the source code](#visualizing-the-subroutine-calling-tree-of-the-source-code)
    -   [Becoming a developer of the code, or making small modifications in the source code](#becoming-a-developer-of-the-code-or-making-small-modifications-in-the-source-code)

Getting Started
===============

Configuring and compiling the source code
-----------------------------------------

To get the SPECFEM3D\_GLOBE software package, type this:

    git clone --recursive --branch devel https://github.com/geodynamics/specfem3d_globe.git

Then, to configure the software for your system, run the `configure` shell script. This script will attempt to guess the appropriate configuration values for your system. However, at a minimum, it is recommended that you explicitly specify the appropriate command names for your Fortran compiler (another option is to define FC, CC and MPIF90 in your .bash\_profile or your .cshrc file):

        ./configure FC=gfortran CC=gcc MPIFC=mpif90

You can replace the GNU compilers above (gfortran and gcc) with other compilers if you want to; for instance for Intel ifort and icc use FC=ifort CC=icc instead.

Before running the `configure` script, you should probably edit file `flags.guess` to make sure that it contains the best compiler options for your system. Known issues or things to check are:

<span>`GCC gfortran compiler`</span>  
The code makes use of Fortran 2008 features, e.g., the `contiguous` array attribute. We thus recommend using a gfortran version 4.6.0 or higher.

<span>`Intel ifort compiler`</span>  
See if you need to add `-assume byterecl` for your machine. In the case of that compiler, we have noticed that initial release versions sometimes have bugs or issues that can lead to wrong results when running the code, thus we *strongly* recommend using a version for which at least one service pack or update has been installed. In particular, for version 17 of that compiler, users have reported problems (making the code crash at run time) with the `-assume buffered_io` option; if you notice problems, remove that option from file `flags.guess` or change it to `-assume nobuffered_io` and try again.

<span>`IBM compiler`</span>  
See if you need to add `-qsave` or `-qnosave` for your machine.

<span>`Mac OS`</span>  
You will probably need to install `XCODE`.

When compiling on an IBM machine with the `xlf` and `xlc` compilers, we suggest running the `configure` script with the following options:

    ./configure FC=xlf90_r MPIFC=mpif90 CC=xlc_r CFLAGS="-O3 -q64" FCFLAGS="-O3 -q64"

If you have problems configuring the code on a Cray machine, i.e. for instance if you get an error message from the `configure` script, try exporting these two variables: `MPI_INC=$CRAY_MPICH2_DIR/include and FCLIBS= `, and for more details if needed you can refer to the `utils/Cray_compiler_information` directory. You can also have a look at the configure script called:
`utils/Cray_compiler_information/configure_SPECFEM_for_Piz_Daint.bash`.

On SGI systems, `flags.guess` automatically informs `configure` to insert “`TRAP_FPE=OFF`” into the generated `Makefile` in order to turn underflow trapping off.

If you run very large meshes on a relatively small number of processors, the static memory size needed on each processor might become greater than 2 gigabytes, which is the upper limit for 32-bit addressing (dynamic memory allocation is always OK, even beyond the 2 GB limit; only static memory has a problem). In this case, on some compilers you may need to add ``` ``-mcmodel=medium ```” (if you do not use the Intel ifort / icc compiler) or ``` ``-mcmodel=medium -shared-intel ```” (if you use the Intel ifort / icc compiler) to the configure options of CFLAGS, FCFLAGS and LDFLAGS otherwise the compiler will display an error message (for instance `relocation truncated to fit: R_X86_64_PC32 against .bss` or something similar); on an IBM machine with the `xlf` and `xlc` compilers, using `-q64` is usually sufficient.

We recommend that you add <span>`ulimit -S -s unlimited`</span> to your <span>`.bash_profile`</span> file and/or <span>`limit stacksize unlimited `</span> to your <span>`.cshrc`</span> file to suppress any potential limit to the size of the Unix stack.

Using the GPU version of the code
---------------------------------

SPECFEM3D\_GLOBE now supports CUDA, OpenCL and HIP GPU acceleration. CUDA configuration can be enabled with `--with-cuda` flag and `CUDA_FLAGS=`, `CUDA_LIB=`, `CUDA_INC=` and ` MPI_INC=` variables like

    ./configure --with-cuda CUDA_FLAGS= CUDA_LIB= CUDA_INC= MPI_INC= ..

When compiling for specific GPU cards, you can enable the corresponding Nvidia GPU card architecture version with:

      ./configure --with-cuda=cuda9 ..

where for example `cuda4,cuda5,cuda6,cuda7,..` specifies the target GPU architecture of your card, (e.g., with CUDA 9 this refers to Volta V100 cards), rather than the installed version of the CUDA toolkit. Before CUDA version 5, one version supported basically one new architecture and needed a different kind of compilation. Since version 5, the compilation has stayed the same, but newer versions supported newer architectures. However at the moment, we still have one version linked to one specific architecture:

    - CUDA 4 for Tesla,   cards like K10, Geforce GTX 650, ..
    - CUDA 5 for Kepler,  like K20
    - CUDA 6 for Kepler,  like K80
    - CUDA 7 for Maxwell, like Quadro K2200
    - CUDA 8 for Pascal,  like P100
    - CUDA 9 for Volta,   like V100
    - CUDA 10 for Turing, like GeForce RTX 2080
    - CUDA 11 for Ampere, like A100

So even if you have the new CUDA toolkit version 11, but you want to run on say a K20 GPU, then you would still configure with:

      ./configure --with-cuda=cuda5

The compilation with the cuda5 setting chooses then the right architecture (`-gencode=arch=compute_35,code=sm_35` for K20 cards).

The same applies to compilation for AMD cards with HIP:

      ./configure --with-hip ..

or

      ./configure --with-hip=MI8 ..

where for example `MI8,MI25,MI50,MI100,..` specifies the target GPU architecture of your card.

OpenCL can be enabled with the `--with-opencl` flag, and the compilation can be controlled through three variables: `OCL_LIB=`, `OCL_INC=` and `OCL_GPU_FLAGS=`.

    ./configure --with-opencl OCL_LIB= OCL_INC= OCL_GPU_FLAGS=..

Both CUDA and OpenCL environments can be compiled simultaneously by merging these two lines. For the runtime configuration, the `GPU_MODE` flag must be set to `.true.`. In addition, we use three parameters to select the environments and GPU:

    GPU_RUNTIME = 0|1|2|3
    GPU_PLATFORM = filter|*
    GPU_DEVICE = filter|*

`GPU_RUNTIME`  
sets the runtime environments: \(1\) for CUDA, \(2\) for OpenCL, \(3\) for HIP and \(0\) for compile-time decision (hence, SPECFEM should have been compiled with only one of `--with-cuda`, `--with-opencl` or `--with-hip`).

`GPU_PLATFORM` and `GPU_DEVICE`  
are both (case-insensitive) filters on the platform and device name in OpenCL, device name only in CUDA. In multiprocessor (MPI)runs, each process will pick a GPU in this filtered subset, in round-robin. The star filter (`*`) will match the first platform and all its devices.

`GPU_RUNTIME`, `GPU_PLATFORM` and `GPU_DEVICE` are not read if `GPU_MODE` is not activated. Regarding the code, `--with-opencl` defines the macro-processor flag `USE_OPENCL`, `--with-cuda` defines `USE_CUDA`, and `--with-hip` defines `USE_HIP`; and `GPU_RUNTIME` set the global variable `run_cuda`, `run_opencl` or `run_hip`. Texture support has not been validated in OpenCL, but works as expected in CUDA.

Note about the CUDA/OpenCL/HIP kernel versions: the CUDA/OpenCL/HIP device kernels were created using a software package called BOAST (Videau, Marangozova-Martin, and Cronsioe 2013) by Brice Videau and Kevin Pouget from Grenoble, France. This source-to-source translation tool reads the kernel definitions (written in ruby) in directory `src/gpu/boast` and generates the corresponding device kernel files provided in directory `src/gpu/kernels.gen`.

Adding OpenMP support in addition to MPI
----------------------------------------

OpenMP support can be enabled in addition to MPI. However, in many cases performance will not improve because our pure MPI implementation is already heavily optimized and thus the resulting code will in fact be slightly slower. A possible exception could be IBM BlueGene-type architectures.

To enable OpenMP, add the flag `--enable-openmp` to the configuration:

    ./configure --enable-openmp ..

This will add the corresponding OpenMP flag for the chosen Fortran compiler.

The DO-loop using OpenMP threads has a SCHEDULE property. The `OMP_SCHEDULE` environment variable can set the scheduling policy of that DO-loop. Tests performed by Marcin Zielinski at SARA (The Netherlands) showed that often the best scheduling policy is DYNAMIC with the size of the chunk equal to the number of OpenMP threads, but most preferably being twice as the number of OpenMP threads (thus chunk size = 8 for 4 OpenMP threads etc). If `OMP_SCHEDULE` is not set or is empty, the DO-loop will assume generic scheduling policy, which will slow down the job quite a bit.

Configuration summary
---------------------

A summary of the most important configuration variables follows.

<span>`FC`</span>  
Fortran compiler command name. By default, `configure` will execute the command names of various well-known Fortran compilers in succession, picking the first one it finds that works.

<span>`MPIFC`</span>  
MPI Fortran command name. The default is `mpif90`. This must correspond to the same underlying compiler specified by `FC`; otherwise, you will encounter compilation or link errors when you attempt to build the code. If you are unsure about this, it is usually safe to set both `FC` and `MPIFC` to the MPI compiler command for your system:

    ./configure FC=mpif90 MPIFC=mpif90

<!-- -->

<span>`FLAGS_CHECK`</span>  
Compiler flags.

<span>`LOCAL_PATH_IS_ALSO_GLOBAL`</span>  
If you want the parallel mesher to write a parallel (i.e., split) database for the solver on the local disks of each of the compute nodes, set this flag to `.false.`. Some systems have no local disks (e.g., BlueGene) and other systems have a fast parallel file system (LUSTRE, GPFS) that is easy and reliable to use, in which case this variable should be set to `.true.`. Note that this flag is not used by the mesher nor the solver; it is only used for some of the (optional) post-processing. If you do not know what is best on your system, setting it to `.true.` is usually fine; or else, ask your system administrator.

In addition to reading configuration variables, `configure` accepts the following options:

<span>`--enable-double-precision`</span>  
The package can run either in single or in double precision mode. The default is single precision because for almost all calculations performed using the spectral-element method using single precision is sufficient and gives the same results (i.e. the same seismograms); and the single precision code is faster and requires exactly half as much memory. To specify double precision mode, simply provide `--enable-double-precision` as a command-line argument to `configure`. On many current processors (e.g., Intel, AMD, IBM Power), single precision calculations are significantly faster; the difference can typically be 10% to 25%. It is therefore better to use single precision. What you can do once for the physical problem you want to study is run the same calculation in single precision and in double precision on your system and compare the seismograms. If they are identical (and in most cases they will), you can select single precision for your future runs.

<span>`--help`</span>  
Directs `configure` to print a usage screen which provides a short description of all configuration variables and options. Note that the options relating to installation directories (e.g., `--prefix`) do not apply to SPECFEM3D\_GLOBE.

The `configure` script runs a brief series of checks. Upon successful completion, it generates the files `Makefile`, `constants.h`, and `precision.h` in the working directory.

<span>Note:</span>  
If the `configure` script fails, and you don’t know what went wrong, examine the log file `config.log`. This file contains a detailed transcript of all the checks `configure` performed. Most importantly, it includes the error output (if any) from your compiler.

The `configure` script automatically runs the script `flags.guess`. This helper script contains a number of suggested flags for various compilers; e.g., Portland, Intel, Absoft, NAG, Lahey, NEC, IBM and SGI. The software has run on a wide variety of compute platforms, e.g., various PC clusters and machines from Sun, SGI, IBM, Compaq, and NEC. The `flags.guess` script attempts to guess which compiler you are using (based upon the compiler command name) and choose the related optimization flags. The `configure` script then automatically inserts the suggested flags into `Makefile`. Note that `flags.guess` may fail to identify your compiler; and in any event, the default flags chosen by `flags.guess` are undoubtedly not optimal for your system. So, we encourage you to experiment with these flags (by editing the generated `Makefile` by hand) and to solicit advice from your system administrator. Selecting the right compiler and compiler flags can make a tremendous difference in terms of performance. We welcome feedback on your experience with various compilers and flags.

When using a slow or not too powerful shared disk system or when running extremely large simulations (on tens of thousands of processor cores), one can add `-DUSE_SERIAL_CASCADE_FOR_IOs` to the compiler flags in file `flags.guess` before running `configure` to make the mesher output mesh data to the disk for one MPI slice after the other, and to make the solver do the same thing when reading the files back from disk. Do not use this option if you do not need it because it will slow down the mesher and the beginning of the solver if your shared file system is fast and reliable.

If you run scaling benchmarks of the code, for instance to measure its performance on a new machine, and are not interested in the physical results (the seismograms) for these runs, you can set `DO_BENCHMARK_RUN_ONLY` to `.true.` in file `setup/constants.h.in` before running the `configure` script.

If your compiler has problems with the `use mpi` statements that are used in the code, use the script called `replace_use_mpi_with_include_mpif_dot_h.pl` in the root directory to replace all of them with `include ’mpif.h’` automatically.

We recommend that you ask for exclusive use of the compute nodes when running on a cluster or a supercomputer, i.e., make sure that no other users are running on the same nodes at the same time. Otherwise your run could run out of memory if the memory of some nodes is used by other users, in particular when undoing attenuation using the UNDO\_ATTENUATION option in DATA/Par\_file. To do so, ask your system administrator for the option to add to your batch submission script; it is for instance `#BSUB -x` with SLURM and `#$ -l exclusive=TRUE` with Sun Grid Engine (SGE).

Compiling on an IBM BlueGene
----------------------------

Installation instructions for IBM BlueGene (from October 2012):

Edit file `flags.guess` and put this for `FLAGS_CHECK`:

    -g -qfullpath -O2 -qsave -qstrict -qtune=qp -qarch=qp -qcache=auto -qhalt=w \
      -qfree=f90 -qsuffix=f=f90 -qlanglvl=95pure -Q -Q+rank,swap_all -Wl,-relax

The most relevant are the -qarch and -qtune flags, otherwise if these flags are set to “auto” then they are wrongly assigned to the architecture of the frond-end node, which is different from that on the compute nodes. You will need to set these flags to the right architecture for your BlueGene compute nodes, which is not necessarily “qp”; ask your system administrator. On some machines if is necessary to use -O2 in these flags instead of -O3 due to a compiler bug of the XLF version installed. We thus suggest to first try -O3, and then if the code does not compile or does not run fine then switch back to -O2. The debug flags (-g, -qfullpath) do not influence performance but are useful to get at least some insights in case of problems.

Before running `configure`, select the XL Fortran compiler by typing `module load bgq-xl/1.0` or `module load bgq-xl` (another, less efficient option is to load the GNU compilers using `module load bgq-gnu/4.4.6` or similar).

Then, to configure the code, type this:

    ./configure FC=bgxlf90_r MPIFC=mpixlf90_r CC=bgxlc_r LOCAL_PATH_IS_ALSO_GLOBAL=true

To compile the code on an IBM BlueGene, Laurent Léger from IDRIS, France, suggests the following: compile the code with

    FLAGS\_CHECK="-O3 -qsave -qstrict -qtune=auto -qarch=450d -qcache=auto \
      -qfree=f90 -qsuffix=f=f90 -g -qlanglvl=95pure -qhalt=w -Q -Q+rank,swap_all -Wl,-relax"

Option “-Wl,-relax” must be added on many (but not all) BlueGene systems to be able to link the binaries `xmeshfem3D` and `xspecfem3D` because the final link step is done by the GNU `ld` linker even if one uses `FC=bgxlf90_r, MPIFC=mpixlf90_r` and `CC=bgxlc_r` to create all the object files. On the contrary, on some BlueGene systems that use the native AIX linker option “-Wl,-relax” can lead to problems and must be suppressed from `flags.guess`.

One then just needs to pass the right commands to the `configure` script:

    ./configure --prefix=/path/to/SPECFEM3DG_SP --host=Babel --build=BGP \
          FC=bgxlf90_r MPIFC=mpixlf90_r CC=bgxlc_r  \
          LOCAL_PATH_IS_ALSO_GLOBAL=false

This trick can be useful for all hosts on which one needs to cross-compile.

On BlueGene, one also needs to run the `xcreate_header_file` binary file manually rather than in the Makefile:

    bgrun -np 1 -mode VN -exe ./bin/xcreate_header_file

Compiling on an Intel Xeon Phi (Knights Landing KNL)
----------------------------------------------------

In case you want to run simulations on a KNL chip, the compilation doesn’t require much more effort than with any other CPU system. All you could add is the flag `-xMIC-AVX512` to your Fortran flags in the `Makefile` and use `--enable-openmp` for configuration.

Since there are different memory types available with a KNL, make sure to use fast memory allocations, i.e. MCDRAM, which has a higher memory bandwidth. Assuming you use a flat mode setup of the KNL chip, you could use the Linux tool `numactl` to specify which memory node to bind to. For example, check with

    numactl --hardware

which node contains CPU cores and which one only binds to MCDRAM (\(\sim\)16GB). In flat mode setup, most likely node 1 does. For a small example on a single KNL with 4 MPI processes and 16 OpenMP threads each, you would run the solver with a command like

    OMP_NUM_THREADS=16 mpirun -np 4 numactl --membind=1 ./bin/xspecfem3D

The ideal setup of MPI processes and OpenMP threads per KNL depends on your specific hardware and simulation setup. We see good results when using a combination of both, with a total number of threads slightly less than the total count of cores on the chip.

As a side remark for developers, another possibility would be to add following compiler directives in the source code (in file `src/specfem3D/specfem3D_par.F90`):

      real(kind=CUSTOM_REAL), dimension(:,:), allocatable :: &
         displ_crust_mantle,veloc_crust_mantle,accel_crust_mantle
    ! FASTMEM attribute: note this attribute needs compiler flag -lmemkind to work...
    !DEC$ ATTRIBUTES FASTMEM :: displ_crust_mantle,veloc_crust_mantle,accel_crust_mantle

These directives will work with Intel ifort compilers and will need the additional linker/compiler flag `-lmemkind` to work properly. We omitted these directives for now to avoid confusion with other possible simulation setups.

Using a cross compiler
----------------------

The `configure` script assumes that you will compile the code on the same kind of hardware as the machine on which you will run it. On some systems (for instance IBM BlueGene, see also the previous section) this might not be the case and you may compile the code using a cross compiler on a frontend computer that does not have the same architecture. In such a case, typing `make all` on the frontend will fail, but you can use one of these two solutions:

1/ create a script that runs `make all` on a node instead of on the frontend, if the compiler is also installed on the nodes

2/ after running the `configure` script, create two copies of the Makefiles:
*TODO: this has not been tested out yet, any feedback is welcome*

In `src/create_header_file/Makefile` put this instead of the current values:

    FLAGS_CHECK = -O0

and replace

    create_header_file: $O/create_header_file.o $(XCREATE_HEADER_OBJECTS)
     ${FCCOMPILE_CHECK} -o ${E}/xcreate_header_file $O/create_header_file.o $(XCREATE_HEADER_OBJECTS)

with

    xcreate_header_file: $O/create_header_file.o $(XCREATE_HEADER_OBJECTS)
      ${MPIFCCOMPILE_CHECK} -o ${E}/xcreate_header_file $O/create_header_file.o $(XCREATE_HEADER_OBJECTS)

In `src/specfem3D/Makefile` comment out these last two lines:

    #${OUTPUT}/values_from_mesher.h: reqheader
    #       (mkdir -p ${OUTPUT}; cd ${S_TOP}/; ./bin/xcreate_header_file)

Then:

     make clean
     make create_header_file
     ./bin/xcreate_header_file
     make clean
     make meshfem3D
     make specfem3D

should work.

Visualizing the subroutine calling tree of the source code
----------------------------------------------------------

Packages such as `doxywizard` can be used to visualize the subroutine calling tree of the source code. `Doxywizard` is a GUI front-end for configuring and running `doxygen`.

To visualize the call tree (calling tree) of the source code, you can see the Doxygen tool available in directory `doc/call_trees_of_the_source_code`.

Becoming a developer of the code, or making small modifications in the source code
----------------------------------------------------------------------------------

If you want to develop new features in the code, and/or if you want to make small changes, improvements, or bug fixes, you are very welcome to contribute. To do so, i.e. to access the development branch of the source code with read/write access (in a safe way, no need to worry too much about breaking the package, there are CI tests based on BuildBot, Travis-CI and Jenkins in place that are checking and validating all new contributions and changes), please visit this Web page:
<https://github.com/geodynamics/specfem3d_globe/wiki>

References
----------

Videau, B., V. Marangozova-Martin, and J. Cronsioe. 2013. “BOAST: Bringing Optimization through Automatic Source-to-Source Tranformations.” In *Proceedings of the 7th International Symposium on Embedded Multicore/Manycore System-on-Chip (MCSoC)*. Tokyo, Japan.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Jun 29, 2021)
