**Table of Contents**

- [Copyright](#cha:Copyright)

Copyright
=========

Main ‘historical’ authors: Dimitri Komatitsch and Jeroen Tromp (there are now many more!).

Princeton University, USA, and CNRS / University of Marseille, France. $\copyright$ Princeton University, USA and CNRS / University of Marseille, France, April 2014

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation (see Appendix [\[cha:License\]](#cha:License)).

Please note that by contributing to this code, the developer understands and agrees that this project and contribution are public and fall under the open source license mentioned above.

***Evolution of the code:***

version 8.0, March 2023: Wolfgang Bangerth, Stephen Beller, Ebru Bozdag, Caio Ciardelli, Congyue Cui, Armando Espindola-Carmona, Rene Gassmoeller, Sunny Gogar, Leopold Grinberg, Elodie Kendall, Wenjie Lei, Amanda McPherson, Ridvan Orsvuran, Daniel Peter, Norbert Podhorszki, Eduardo Valero Cano. support for new Earth, Moon & Mars models; ADIOS2 file I/O support, GLL models for azimuthal anisotropy & Q, LDDRK on GPU support, Laplacian smoothing, monochromatic source time functions, sponge absorbing boundaries, steady-state kernels; mesh cut-off, seismogram down-sampling, non-static compilation.

version 7.0.2, July 2019: Kazuto Ando, Etienne Bachmann, Hom Nath Gharti, Matthieu Lefebvre, Wenjie Lei, Dimitri Komatitsch, Andy Nowacki, Daniel Peter, Elliott Sales de Andrade, Malte Schirwon, James Smith, Kai Tao, Brice Videau, Victor (@MisterFruits). code improvements for adjoint reader for ASDF, CUDA and OpenCL, LDDRK, OpenMP and loop performance for compute forces, undo attenuation, BOAST kernels; support for full SH models, LIBXSMM, point force sources.

version 7.0.1, July 2015: Matthieu Lefebvre, Dimitri Komatitsch, Daniel Peter, Elliott Sales de Andrade, James Smith. code improvements (performance for noise simulations, ASDF provenance); updates examples.

version 7.0, January 2015: many developers. simultaneous MPI runs, ADIOS file I/O support, ASDF seismograms, new seismogram names, tomography tools, CUDA and OpenCL GPU support, CEM model support, updates AK135 model, binary topography files, fixes geocentric/geographic conversions, updates ellipticity and gravity factors, git versioning system.

version 6.0, April 2014: Daniel Peter (ETH Zürich, Switzerland), Dimitri Komatitsch and Zhinan Xie (CNRS / University of Marseille, France), Elliott Sales de Andrade (University of Toronto, Canada), and many others, in particular from Princeton University, USA. more flexible MPI implementation, GPU support, exact undoing of attenuation, LDDRK4-6 higher-order time scheme, etc...

version 5.1, February 2011: Dimitri Komatitsch, University of Toulouse, France and Ebru Bozdag, Princeton University, USA. non blocking MPI for much better scaling on large clusters; new convention for the name of seismograms, to conform to the IRIS standard; new directory structure.

version 5.0, February 2010: many developers. new moho mesh stretching honoring crust2.0 moho depths, new attenuation assignment, new SAC headers, new general crustal models, faster performance due to Deville routines and enhanced loop unrolling, slight changes in code structure.

version 4.0, February 2008: David Michéa and Dimitri Komatitsch, University of Pau, France. first port to GPUs using CUDA, new doubling brick in the mesh, new perfectly load-balanced mesh, more flexible routines for mesh design, new inflated central cube with optimized shape, far fewer mesh files saved by the mesher, global arrays sorted to speed up the simulation, seismos can be written by the main process, one more doubling level at the bottom of the outer core if needed (off by default).

version 3.6, September 2006: Many people, many affiliations. adjoint and kernel calculations, fixed IASP91 model, added AK135 and 1066a, fixed topography/bathymetry routine, new attenuation routines, faster and better I/Os on very large systems, many small improvements and bug fixes, new "configure" script, new user’s manual etc.

version 3.5, July 2004: Dimitri Komatitsch, Brian Savage and Jeroen Tromp, Caltech. any size of chunk, 3D attenuation, case of two chunks, more precise topography/bathymetry model, new Par_file structure.

version 3.4, August 2003: Dimitri Komatitsch and Jeroen Tromp, Caltech. merged global and regional codes, no iterations in fluid, better movies.

version 3.3, September 2002: Dimitri Komatitsch, Caltech. flexible mesh doubling in outer core, inlined code, OpenDX support.

version 3.2, July 2002: Jeroen Tromp, Caltech. multiple sources and flexible PREM reading.

version 3.1, June 2002: Dimitri Komatitsch, Caltech. vectorized loops in solver and merged central cube.

version 3.0, May 2002: Dimitri Komatitsch and Jeroen Tromp, Caltech. ported to SGI and Compaq, double precision solver, more general anisotropy.

version 2.3, August 2001: Dimitri Komatitsch and Jeroen Tromp, Caltech. gravity, rotation, oceans and 3-D models.

version 2.2, March 2001: Dimitri Komatitsch and Jeroen Tromp, Caltech, USA. final MPI package.

version 2.0, January 2000: Dimitri Komatitsch, Harvard, USA. MPI code for the globe.

version 1.0, June 1999: Dimitri Komatitsch, UNAM, Mexico. first MPI code for a chunk.

Jeroen Tromp and Dimitri Komatitsch, Harvard, USA, July 1998: first chunk solver using OpenMP on a Sun machine.

Dimitri Komatitsch, IPG Paris, France, December 1996: first 3-D solver for the CM-5 Connection Machine, parallelized on 128 processors using Connection Machine Fortran.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Mar 21, 2023)

