**Table of Contents**

-   [Simulation features supported in SPECFEM3D\_GLOBE](#simulation-features-supported-in-specfem3d95globe)

Simulation features supported in SPECFEM3D\_GLOBE
=================================================

The following lists all available features for a SPECFEM3D\_GLOBE simulation, where <span>*CPU*</span>, <span>*CUDA*</span> and <span>*OpenCL*</span> denote the code versions for CPU-only simulations, CUDA and OpenCL hardware support, respectively.

[table:features]

| <span>**Feature**</span>             |                                | <span>*CPU*</span> | <span>*CUDA*</span> | <span>*OpenCL*</span> |
|:-------------------------------------|:-------------------------------|:------------------:|:-------------------:|:---------------------:|
|                                      |                                |                    |                     |                       |
| <span>**Physics**</span>             | Ocean load                     |          X         |          X          |           X           |
|                                      | Ellipticity                    |          X         |          X          |           X           |
|                                      | Topography                     |          X         |          X          |           X           |
|                                      | Gravity                        |          X         |          X          |           X           |
|                                      | Rotation                       |          X         |          X          |           X           |
|                                      | Attenuation                    |          X         |          X          |           X           |
|                                      |                                |                    |                     |                       |
| <span>**Simulation Setup**</span>    | Global (6-chunks)              |          X         |          X          |           X           |
|                                      | Regional (1,2-chunk)           |          X         |          X          |           X           |
|                                      | Restart/Checkpointing          |          X         |          X          |           X           |
|                                      | Simultaneous runs              |          X         |          X          |           X           |
|                                      | ADIOS file I/O                 |          X         |          X          |           X           |
|                                      |                                |                    |                     |                       |
| <span>**Sensitivity kernels**</span> | Partial physical dispersion    |          X         |          X          |           X           |
|                                      | Undoing of attenuation         |          X         |          X          |           X           |
|                                      | Anisotropic kernels            |          X         |          X          |           X           |
|                                      | Transversely isotropic kernels |          X         |          X          |           X           |
|                                      | Isotropic kernels              |          X         |          X          |           X           |
|                                      | Approximate Hessian            |          X         |          X          |           X           |
|                                      |                                |                    |                     |                       |
| <span>**Time schemes**</span>        | Newmark                        |          X         |          X          |           X           |
|                                      | LDDRK                          |          X         |          -          |           -           |
|                                      |                                |                    |                     |                       |
| <span>**Visualization**</span>       | Surface movie                  |          X         |          X          |           X           |
|                                      | Volumetric movie               |          X         |          X          |           X           |
|                                      |                                |                    |                     |                       |
| <span>**Seismogram formats**</span>  | Ascii                          |          X         |          X          |           X           |
|                                      | SAC                            |          X         |          X          |           X           |
|                                      | ASDF                           |          X         |          X          |           X           |
|                                      | Binary                         |          X         |          X          |           X           |
|                                      |                                |                    |                     |                       |

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Feb 14, 2021)

