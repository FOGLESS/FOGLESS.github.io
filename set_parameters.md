---
layout: default
title: Setting the parameters
nav_order: 4
last_modified_at: 2022-04-06T13:37:11
---

## Setting the parameters of ASOHF

In this section, we describe the run-time parameters of ASOHF, which are set in the `./input_files/asohf.dat` file. These can be changed after [compilation of the code](get_ASOHF#compilation).

### General parameters block

These parameters refer to the general behaviour of the code.

```
Files: first, last, every -------------------------------------------->
0,0,50
```
- Iterations to analyse. It is assumed that the outputs have been saved each `every` iteration. If instead, outputs have been saved at a pace that does not correspond to constant interval of iterations, it is straightforward to change the main loop of the code (look in the `asohf.f` file for the `DO IFI2=1, NFILE2` line). 

```
Cells per direction (NX,NY,NZ) --------------------------------------->
128,128,128
```
- Number of grid cells in each direction. These grid sizes should be smaller or equal than the [compilation parameters `NMAX`, `NMAY` and `NMAZ`](get_ASOHF#compilation-time-parameters).


```
Hubble constant (h), omega matter, fraction of DM to total mass ------>
0.678,0.31,0.845
```
- The Hubble dimensionless constant (<img src="https://render.githubusercontent.com/render/math?math=h\equiv H_0/(100\,\mathrm{km}\,\mathrm{s}^{-1}\,\mathrm{Mpc}^{-1})">), the matter density parameter (<img src="https://render.githubusercontent.com/render/math?math=\Omega_m=\rho_B(z=0)/\rho_\mathrm{crit}(z=0)">) and the fraction of DM to total mass (<img src="https://render.githubusercontent.com/render/math?math=f_\mathrm{DM}\equiv 1 - \Omega_b / \Omega_m">).

```
Max box sidelength (in input length units) --------------------------->
40.0
```
- The largest of the box side lengths, in the input length units.

```
Reading flags: IS_MASCLET (=0, no; =1, yes), GENERIC_READER (see docs) ----->
0,0
```
- Generally, set the first parameter to 0. As for the second:
  - `GENERIC_READER=0` for the generic `particlesXXXXX` files described in the [input data page](input_data).
  - `GENERIC_READER=1` for the GADGET-2 unformatted files reader.
>#### MASCLET users
>- If reading data from MASCLET, set the first parameter to 1

```
Output flags: grid_asohf,density,haloes_grids,subs_grids,subs_part --->
0,0,0,0,0
```
- Output intermediate files for debugging purposes (=0, no; =1, yes):
    - ASOHF grid file 
    - ASOHF density interpolation 
    - List of haloes pre-identified within the grid
    - List of subhaloes pre-identified within the grid 
    - List of substructures identified using particles

```
Input units: MASS (Msun; <0 for Msun/h), LENGTH (cMpc; <0 for cMpc/h),
 SPEED (km/s), ALPHA (v_input = a^alpha dx/dt; 1 is peculiar vel.) --->
9.1717e18,1.0,299792.458,1.0
```
- Units of the input data:
    - Input unit of mass in solar masses; if negative, the input mass is assumed to be given in Msun/h.
    - Input unit of length in comoving Mpc; if negative, the input length is assumed to be given in cMpc/h.
    - Input unit of speed in km/s.
    - Since different codes use different velocity variables, specify the exponent <img src="https://render.githubusercontent.com/render/math?math=\alpha"> such that <img src="https://render.githubusercontent.com/render/math?math=\mathbf{v}_\mathrm{input} = a(t)^\alpha \frac{\mathrm{d}\mathbf{x}}{\mathrm{d}t}">, with a(t) the scale factor of the given cosmology and x the comoving position. For example, with <img src="https://render.githubusercontent.com/render/math?math=\alpha=1">, the input velocity is the usual peculiar velocity.
>#### MASCLET users:
>This line can be ignored, as the reader will automatically take care of MASCLET units.

```
Input domain (in input length units; x1,x2,y1,y2,z1,z2) -------------->
-20.0,20.0,-20.0,20.0,-20.0,20.0
```
- Specify the input domain of the simulation, with the x, y and z left and right corners of the domain.

### Domain decompose block
This parameters are used to decompose the input domain into subdomains, or to restrict the finding process to a specific reason. If you do not want to use the feature, set the first parameter to 0 and do not worry about the second line.

```
Keep only particles inside a given domain (=0, no; =1, yes) ---------->
0
```
- Set this parameter to 1 if you want to analyse only particles inside a given subdomain.

```
Domain to keep particles (in input length units; x1,x2,y1,y2,z1,z2) -->
0.,0.,0.,0.,0.,0.
```
- If the above parameter is set to 1, specify the volume where you want to keep the particles. Use the same units for positions as for the input domain.

### Mesh creation parameters block
These parameters control the mesh creation process, which is crucial for the identification of density peaks.

```
Levels for the mesh (stand-alone) ------------------------------------>
4
```
- Maximum number of refinement levels to create. Take into account that your best resolution will be <img src="https://render.githubusercontent.com/render/math?math=L/(N_x \cdot 2^\mathrm{NLEVELS})">, with <img src="https://render.githubusercontent.com/render/math?math=L, \, N_x"> the domain length and the number of grid cells. A typical suggestion is to set it to the force resolution of the simulation, since you are not expected to form structures below this scale. If you are not interested in smaller scales, you could use less levels; but it is nevertheless recommended to use a sufficient number of levels for being able to recenter the density peaks properly.

```
PARCHLIM(=0 no limit patches/level,>0 limit) ------------------------->
0
Max num of patches per level(needs PARCHLIM>0) ----------------------->
1000
```
- Use this parameter to limit the number of patches per level. If you do not want to use this feature, set the first parameter to 0. If you want to limit them, set it to 1, and write the maximum number of patches per level in the second line (one integer per level, comma-separated).

```
Refinement threshold (num. part.), refinable fraction to extend ------>
3,0.05
```
- Cells hosting more (or equal) particles than the refinement threshold will be flagged as refinable. You can lower this parameter (up to around 3) to refine more regions, while you can increase it to refine less regions (and therefore reduce computational cost, at the expense of losing small-scale structures). Reasonable values are in the rangle 3-8.
- A patch fill be extended one cell along a given face if the fraction of refinable cells among the cells to be added exceeds this threshold. If you want to accept any extension of a patch with, at least, 1 refinable cell, set this parameter to an arbitrarily small positive value (e.g., 1.e-4).

```
Minimum patch size (child cells) ------------------------------------->
14
```
- A patch has to reach this number of (fine) cells along each dimension in order to be accepted; otherwise the region does not get refined. This parameter is used to avoid the creation of very small patches, which are not expected to be useful for the identification of density peaks and increase drastically the computational cost. Set it to a value larger or equal to 14, and lower than the maximum patch size (`NAMRX`, `NAMRY`, `NAMRZ`). This parameter is crucial to control the number of patches (and, thus, to keep memory usage contained). If you think that the code is creating too many patches, you may want to increase this parameter.

```
Base grid refinement border, AMR grids refinement border ------------->
4,0
```
- Do not flag as refinable the first parameter number of cells close to the domain border. This parameter is used to avoid problems with the boundary conditions. 
- Likewise, the second parameter is the number of cells close to each AMR patch boundary to exclude from further refinement. It can be safely set to 0 if you accept that a patch can share a face, edge or corner with its father (which is not problematic).

```
Allow for additional overlap (to avoid losing signal) in the mesh ---->
0
```
- If this parameter is set to 1, the cells in the boundary of a patch will not be considered refined, so as to try to place another patch covering them so that they are in a more central position. Generally it is not required, but it may rarely happen that some small halo is lost due to its density peak being too close to the boundary of the patch. 

```
Density interpolation kernel (1=linear, 2=quadratic) ----------------->
2
```
- Order of the kernel used to interpolate each particle into the density field: 1 for a trilinear kernel; 2 for a tri-quadratic kernel.

```
Variable for mesh halo finding: 1(dm), 2(dm+stars) ------------------->
1
```
- This parameter is used to specify the variable to use for the halo finding. If you want to use only DM, set it to 1. If you want to use also stars, set it to 2.

```
Kernel level for stars (if VAR=2) ------------------------------------>
5
```
- If `VAR=2`, the following parameter can be used to specify the kernel level of stars. That is, stars will be interpolated onto the density field using a kernel with size <img src="https://render.githubusercontent.com/render/math?math=L/(N_x \cdot 2^{\ell_\mathrm{stars}})">.

```
Particle especies (0=there are different mass particles, 1=equal mass
 particles, use local density, 2=equal mass particles, do nothing) --->
1
```
- How to set the kernel size for each particle:
    - If 0, ASOHF assumes DM particles have different masses, and each particle will get a cloud representing its original size back in the initial conditions.
    - If 1, ASOHF computes the kernel size from a pre-estimation of the density field on the base grid. Particles in a cell with density <img src="https://render.githubusercontent.com/render/math?math=\rho"> will get a cloud equivalent to the refinement level <img src="https://render.githubusercontent.com/render/math?math=\lfloor \log_8 (\rho/\rho_B) \rfloor)"> (bounded by 0 and `N_ESP`-1).
    - If 2, ASOHF assumes that all particles have the same mass, and therefore the kernel size is the same for all of them.

### Halo finding parameters block
These parameters control the halo finding process.

```
Max. reach around halos (cMpc), excluded cells in boundaries --------->
4.0,1
```
- The first parameter is the maximum expected size for an object (or an upper limit to it).
- The second parameter is the number of cells to exclude from the boundary of a patch to look for density peaks. Set it, at least, to 1, in order to be able to compute central differences.

```
Minimum fraction of shared volume to merge (in grid search) ---------->
0.6
```
- Haloes overlapping a fraction of the volume of the smallest larger than this are no longer considered within the process of (non-substructure) halo finding.

```
FLAG_WDM (=1 write DM particles, =0 no) ------------------------------>
1
```
- Whether to write the DM (and stellar, if any) lists of particles belonging to each halo.

```
Search for substructure (=1 yes, =0 no) ------------------------------>
0
```
- Whether to run the substructure search.

```
Compute energies (=1 yes, =0 no), max num of part. for direct sum ---->
0,4000
```
- The first parameter controls whether to run the gravitational energy computation.
- The second parameter establishes the maximum number of particles so that the gravitational binding energy is computed by direct summation. For haloes with more particles, a sampling estimate is used instead.

```
Minimum number of particles per halo --------------------------------->
25
```
- Haloes with less than this number of particles are regarded as 'poor' haloes and discarded.

### Stellar halo finding parameters block
These parameters control the stellar halo finding process. All can be ignored if the first one (whether to look for stellar haloes) is set to 0.

```
Look for stellar haloes (=1 yes, =0 no) ------------------------------>
0
```
- Whether to look for stellar haloes.

```
Minimum number of stellar particles per stellar halo ----------------->
25
```
- Stellar haloes with less than this number of stellar particles within the stellar half-mass radius are regarded as 'poor' stellar haloes and discarded.

```
Cut stellar halo if density increases more than factor from min ------>
5.0
```
- Consider a maximum radius for the stellar halo if density increases by a factor larger than this from the minimum value of the density inside that radius.

```
Cut stellar halo if radial distance of consecutive stars > (ckpc) ---->
10.0
```
- Consider a maximum radius for the stellar halo if there is a gap in radial space without any stellar particle larger than this comoving length.

```
Cut stellar halo if rho_* falls below this factor of rho_B ----------->
1.0
```
- Consider a maximum radius for the stellar halo if the density of the stars falls below this factor of the density of the background.
```
Cut stellar halo at a maximum (>0, physical; <0, comoving) radius of
 (kpc) --------------------------------------------------------------->
 200.0
```
- Consider a maximum radius for the stellar halo. This might be useful to identify BCGs and separate them from the rest of the ICL. If positive, it is interpreted as a physical radius (in kpc). If negative, it is interpreted as a comoving radius (in ckpc).
