**Table of Contents**

- [Changing the Model](#cha:-Changing-the)
  - [Changing the Crustal Model](#sec:Changing-the-Crustal)
  - [Changing the Mantle Model](#sec:Changing-the-Mantle)
    - [Isotropic Models](#sub:Isotropic-Models)
    - [Anisotropic Models](#sub:Anisotropic-Models)
    - [Point-Profile Models](#sub:Point-Profile-Models)
  - [Anelastic Models](#sec:Anelastic-Models)
  - [References](#references)

Changing the Model
==================

In this section we explain how to change the crustal, mantle, or inner core models. These changes involve contributing specific subroutines that replace existing subroutines in the `SPECFEM3D_GLOBE` package.

Changing the Crustal Model
--------------------------

The 3D crustal model Crust2.0 (Bassin, Laske, and Masters 2000) is superimposed onto the mesh by the subroutine `model_crust`
`.f90`. To accomplish this, the flag `CRUSTAL`, set in the subroutine `get_model_parameters.f90`, is used to indicate a 3D crustal model. When this flag is set to `.true.`, the crust on top of the 1D reference model (PREM, IASP91, or AK135F_NO_MUD) is removed and replaced by extending the mantle. The 3D crustal model is subsequently overprinted onto the crust-less 1D reference model. The call to the 3D crustal routine is of the form

    call model_crust(lat,lon,r,vp,vs,rho,moho,foundcrust,CM_V,elem_in_crust)

Input to this routine consists of:

`lat`  
Latitude in degrees.

`lon`  
Longitude in degrees.

`r`  
Non-dimensionalized radius ($0<\texttt{r}<1$).

Output from the routine consists of:

`vp`  
Non-dimensionalized compressional wave speed at location (`lat`,`lon`,`r`).

`vs`  
Non-dimensionalized shear wave speed.

`rho`  
Non-dimensionalized density.

`moho`  
Non-dimensionalized Moho depth.

`found_crust`  
Logical that is set to `.true.` only if crust exists at location (`lat`,`lon`,`r`), i.e., `.false.` for radii `r` in the mantle. This flags determines whether or not a particular location is in the crust and, if so, what parameters to assign to the mesh at this location.

`CM_V`  
Fortran structure that contains the parameters, variables and arrays that describe the model.

`elem_in_crust`  
Logical that is used to force the routine to return crustal values, even if the location would be below the crust.

All output needs to be non-dimensionalized according to the convention summarized in Appendix [\[cha:Non-Dimensionalization-Conventions\]](#cha:Non-Dimensionalization-Conventions). You can replace this subroutine by your own routine *provided you do not change the call structure of the routine*, i.e., the new routine should take exactly the same input and produce the required, properly non-dimensionalized output.

Part of the file `model_crust.f90` is the subroutine `model_crust_broadcast`. The call to this routine takes argument `CM_V` and is used to once-and-for-all read in the databases related to Crust2.0 and broadcast the model to all parallel processes. If you replace the file `model_crust.f90` with your own implementation, you *must* provide a `model_crust_broadcast` routine, even if it does nothing. Model constants and variables read by the routine `model_crust_broadcast` are passed to the subroutine `read_crust_model` through the structure `CM_V`. An alternative crustal model could use the same construct. Please feel free to contribute subroutines for new models and send them to us so that they can be included in future releases of the software.

> **NOTE:** If you decide to create your own version of file `model_crust.f90`, you must add calls to `MPI_BCAST` in the subroutine `model_crust_broadcast` after the call to the `read_crust_model` subroutine that reads the isotropic mantle model once and for all in the mesher. This is done in order to read the (potentially large) model data files on the main node (which is the processor of rank 0 in our code) only and then send a copy to all the other nodes using an MPI broadcast, rather than using an implementation in which all the nodes would read the same model data files from a remotely-mounted home file system, which could create a bottleneck on the network in the case of a large number of nodes. For example, in the current call to that routine from `model_crust.f90,` we write:
>
>     ! the variables read are declared and stored in structure CM_V
>       if(myrank == 0) call read_crust_model(CM_V)
>
>     ! broadcast the information read on the main node to all the nodes
>       call MPI_BCAST(CM_V%thlr,NKEYS_CRUST*NLAYERS_CRUST,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(CM_V%velocp,NKEYS_CRUST*NLAYERS_CRUST,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(CM_V%velocs,NKEYS_CRUST*NLAYERS_CRUST,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(CM_V%dens,NKEYS_CRUST*NLAYERS_CRUST,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(CM_V%abbreviation,NCAP_CRUST*NCAP_CRUST,MPI_CHARACTER,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(CM_V%code,2*NKEYS_CRUST,MPI_CHARACTER,0,MPI_COMM_WORLD,ier)

Changing the Mantle Model
-------------------------

This section discusses how to change isotropic and anisotropic 3D mantle models. Usually such changes go hand-in-hand with changing the 3D crustal model.

### Isotropic Models

The 3D mantle model S20RTS (Ritsema, Van Heijst, and Woodhouse 1999) is superimposed onto the mantle mesh by the subroutines in the file `model_s20rts.f90`. The key call is to the subroutine

    call mantle_s20rts(radius,theta,phi,dvs,dvp,drho,D3MM_V)

Input to this routine consists of:

`radius`  
Non-dimensionalized radius ($\texttt{RCMB/R\_ EARTH}<\texttt{r}<\texttt{RMOHO/R\_ EARTH}$; for a given 1D reference model, the constants `RCMB` and `RMOHO` are set in the `get_model_parameters``.f90` file). The code expects the isotropic mantle model to be defined between the Moho (with radius `RMOHO` in m) and the core-mantle boundary (CMB; radius `RCMB` in m) of a 1D reference model. When a 3D crustal model is superimposed, as will usually be the case, the 3D mantle model is stretched to fill any potential gap between the radius of the Moho in the 1D reference model and the Moho in the 3D crustal model. Thus, when the Moho in the 3D crustal model is shallower than the Moho in the reference model, e.g., typically below the oceans, the mantle model is extended to fill this gap.

`theta`  
Colatitude in radians.

`phi`  
Longitude in radians.

Output from the routine are the following non-dimensional perturbations:

`dvs`  
Relative shear-wave speed perturbations $\delta\beta/\beta$ at location (`radius`,`theta`,`phi`).

`dvp`  
Relative compressional-wave speed perturbations $\delta\alpha/\alpha$.

`drho`  
Relative density perturbations $\delta\rho/\rho$.

`D3MM_V`  
Fortran structure that contains the parameters, variables and arrays that describe the model.

You can replace the `model_s20rts.f90` file with your own version *provided you do not change the call structure of the routine*, i.e., the new routine should take exactly the same input and produce the required relative output.

Part of the file `model_s20rts.f90` is the subroutine `model_s20rts_broadcast`. The call to this routine takes argument` D3MM_V` and is used to once-and-for-all read in the databases related to S20RTS. If you replace the file `model_s20rts.f90` with your own implementation, you *must* provide a `model_s20rts_broadcast` routine, even if it does nothing. Model constants and variables read by the routine `model_s20rts_broadcast` are passed to the subroutine `read_model_s20rts` through the structure `D3MM_V.` An alternative mantle model should use the same construct.

> **NOTE:** If you decide to create your own version of file `model_s20rts.f90`, you must add calls to `MPI_BCAST` in the subroutine `model_s20rts_broadcast` after the call to the `read_model_s20rts` subroutine that reads the isotropic mantle model once and for all in the mesher. This is done in order to read the (potentially large) model data files on the main node (which is the processor of rank 0 in our code) only and then send a copy to all the other nodes using an MPI broadcast, rather than using an implementation in which all the nodes would read the same model data files from a remotely-mounted home file system, which could create a bottleneck on the network in the case of a large number of nodes. For example, in the current call to that routine from `model_s20rts.f90,` we write:
>
>     ! the variables read are declared and stored in structure D3MM_V
>       if(myrank == 0) call read_model_s20rts(D3MM_V)
>
>     ! broadcast the information read on the main node to all the nodes
>       call MPI_BCAST(D3MM_V%dvs_a,(NK+1)*(NS+1)*(NS+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%dvs_b,(NK+1)*(NS+1)*(NS+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%dvp_a,(NK+1)*(NS+1)*(NS+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%dvp_b,(NK+1)*(NS+1)*(NS+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%spknt,NK+1,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%qq0,(NK+1)*(NK+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(D3MM_V%qq,3*(NK+1)*(NK+1),MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)

### Anisotropic Models

Three-dimensional anisotropic mantle models may be superimposed on the mesh based upon the subroutines in the file

    model_aniso_mantle.f90

The key call is to the subroutine

    call mantle_aniso_mantle(r,theta,phi,rho, &
             c11,c12,c13,c14,c15,c16,c22,c23,c24,c25,c26, &
             c33,c34,c35,c36,c44,c45,c46,c55,c56,c66,AMM_V)

Input to this routine consists of:

`r`  
Non-dimensionalized radius ($\texttt{RCMB/R\_ EARTH}<\texttt{r}<\texttt{RMOHO/R\_ EARTH}$; for a given 1D reference model, the constants `RCMB` and `RMOHO` are set in the `get_model_parameters``.f90` file). The code expects the anisotropic mantle model to be defined between the Moho and the core-mantle boundary (CMB). When a 3D crustal model is superimposed, as will usually be the case, the 3D mantle model is stretched to fill any potential gap between the radius of the Moho in the 1D reference model and the Moho in the 3D crustal model. Thus, when the Moho in the 3D crustal model is shallower than the Moho in the reference model, e.g., typically below the oceans, the mantle model is extended to fill this gap.

`theta`  
Colatitude in radians.

`phi`  
Longitude in radians.

Output from the routine consists of the following non-dimensional model parameters:

`rho`  
Non-dimensionalized density $\rho$.

`c11`,  
**$\cdots$,** **`c66`** 21 non-dimensionalized anisotropic elastic parameters.

`AMM_V`  
Fortran structure that contains the parameters, variables and arrays that describe the model.

You can replace the `model_aniso_mantle.f90` file by your own version *provided you do not change the call structure of the routine*, i.e., the new routine should take exactly the same input and produce the required relative output. Part of the file `model_aniso_mantle.f90` is the subroutine `model_aniso_mantle_broadcast`. The call to this routine takes argument `AMM_V` and is used to once-and-for-all read in the static databases related to the anisotropic model. When you choose to replace the file `model_aniso_mantle.f90` with your own implementation you *must* provide a `model_aniso_mantle_broadcast` routine, even if it does nothing. Model constants and variables read by the routine `model_aniso_mantle_broadcast` are passed through the structure `AMM_V`. An alternative anisotropic mantle model should use the same construct.

> **NOTE:** If you decide to create your own version of file `model_aniso_mantle.f90`, you must add calls to `MPI_BCAST` in file `model_aniso_mantle.f90` after the call to the `read_aniso_mantle_model` subroutine that reads the anisotropic mantle model once and for all in the mesher. This is done in order to read the (potentially large) model data files on the main node (which is the processor of rank 0 in our code) only and then send a copy to all the other nodes using an MPI broadcast, rather than using an implementation in which all the nodes would read the same model data files from a remotely-mounted home file system, which could create a bottleneck on the network in the case of a large number of nodes. For example, in the current call to that routine from `model_aniso_mantle.f90,` we write:
>
>     ! the variables read are declared and stored in structure AMM_V
>       if(myrank == 0) call read_aniso_mantle_model(AMM_V)
>
>     ! broadcast the information read on the main node to all the nodes
>       call MPI_BCAST(AMM_V%npar1,1,MPI_INTEGER,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(AMM_V%beta,14*34*37*73,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)
>       call MPI_BCAST(AMM_V%pro,47,MPI_DOUBLE_PRECISION,0,MPI_COMM_WORLD,ier)

Rotation of the anisotropic tensor elements from one coordinate system to another coordinate system may be accomplished based upon the subroutine `rotate_aniso_tensor`. Use of this routine requires understanding the coordinate system used in `SPECFEM3D_GLOBE`, as discussed in Appendix [\[cha:Reference-Frame-Convention\]](#cha:Reference-Frame-Convention).

### Point-Profile Models

In order to facilitate the use of your own specific mantle model, you can choose `PPM` as model in the `DATA/Par_file` file and supply your own model as an ASCII-table file. These generic models are given as depth profiles at a specified lon/lat location and a perturbation (in percentage) with respect to the shear-wave speed values from PREM. The ASCII-file should have a format like:

    #lon(deg), lat(deg), depth(km), Vs-perturbation wrt PREM(%), Vs-PREM (km/s)
     -10.00000       31.00000       40.00000      -1.775005       4.400000
     -10.00000       32.00000       40.00000      -1.056823       4.400000
     ...

where the first line is a comment line and all following ones are specifying the Vs-perturbation at a lon/lat location and a given depth. The last entry on each line is specifying the absolute value of Vs (however this value is only given as a supplementary information and not used any further). The background model is PREM with a transverse isotropic layer between Moho and 220 km depth. The specified Vs-perturbations are added as isotropic perturbations. Please see the file `DATA/PPM/README` for more informations how to setup the directory `DATA/PPM` to use your own ASCII-file.

To change the code behavior of these PPM-routines, please have a look at the implementation in the source code file `model_ppm.f90` and set the flags and scaling factors as needed for your purposes. Perturbations in density and Vp may be scaled to the given Vs-perturbations with constant scaling factors by setting the appropriate values in this source code file. In case you want to change the format of the input ASCII-file, see more details in the Appendix [\[cha:Troubleshooting\]](#cha:Troubleshooting).

Anelastic Models
----------------

Three-dimensional anelastic (attenuation) models may be superimposed onto the mesh based upon your subroutine . `model_atten3D.f90`. The call to this routine would be as follows

    call model_atten3D(radius, colatitude, longitude, Qmu, QRFSI12_Q, idoubling)

Input to this routine consists of:

`radius`  
scaled radius of the earth: $0\,(\mathrm{center})<=r\,<=1$(surface)

`latitude`  
Colatitude in degrees: $0^{\circ}<=\theta<=180^{\circ}$

`longitude`  
Longitude in degrees: $-180^{\circ}<=\phi<=180^{\circ}$

`QRFSI12_Q`  
Fortran structure that contains the parameters, variables and arrays that describe the model

`idoubling`  
value of the doubling index flag in each radial region of the mesh

Output to this routine consists of:

`Qmu`  
Shear wave quality factor: $0<Q_{\mu}<5000$

A 3-D attenuation model QRFSI12 (Dalton, Ekström, and Dziewoński 2008) is provided, as well as 1-D models with a PREM and a 1DREF attenuation structure. By default the PREM attenuation model is taken, using the routine `model_attenuation_1D_PREM`, found in `model_attenuation.f90`.

To create your own three-dimensional attenuation model, you add your model using a routine like the
`model_atten3D_QRFSI12` subroutine and the example routine above as a guide and replace the call in file `meshfem3D_models.f90` to your own subroutine.

Note that the resolution and maximum value of anelastic models are truncated. This speeds the construction of the standard linear solids during the meshing stage. To change the resolution, currently at one significant figure following the decimal, or the maximum value (5000), consult `constants.h`. In order to prevent unexpected results, quality factors $Q_{\mu}$ should never be equal to 0 outside of the inner core.

References
----------

Bassin, C., G. Laske, and G. Masters. 2000. “The Current Limits of Resolution for Surface Wave Tomography in North America.” *EOS* 81: F897.

Dalton, C. A., G. Ekström, and A. M. Dziewoński. 2008. “The Global Attenuation Structure of the Upper Mantle.” *J. Geophys. Res.* 113: B05317. <https://doi.org/10.1029/2006JB004394>.

Ritsema, J., H. J. Van Heijst, and J. H. Woodhouse. 1999. “Complex Shear Velocity Structure Imaged Beneath Africa and Iceland.” *Science* 286: 1925–28.

-----
> This documentation has been automatically generated by [pandoc](http://www.pandoc.org)
> based on the User manual (LaTeX version) in folder doc/USER_MANUAL/
> (Nov  9, 2022)

