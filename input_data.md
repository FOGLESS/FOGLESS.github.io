---
layout: default
title: Input data
nav_order: 3
last_modified_at: 2022-04-06T13:37:11
---

## Input format

In this section, we describe the different ways of loading your particle data into ASOHF. In particular, we describe the general reader format, and how to modify it and create your custom reader subroutine.

### The general reader format

The general reader (subroutine `READ_PARTICLES_GENERAL` in the `reader.f` source file) assumes the data is contained in the file `./simulation/particlesXXXXX` where `XXXXX` is the iteration number. The file is a binary file with the following format, record by record:

- Redshift of the snapshot (single-precision real)
- Number of DM particles (integer), <img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}">
- X position of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Y position of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Z position of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Velocity component X of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Velocity component Y of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Velocity component Z of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Mass of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> single-precision reals)
- Unique ID of the DM particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{DM}"> integers)

If there are stars in the simulation (and it is so indicated in the parameters file; see [the section on parameters](set_parameters)), the following data is also read:

- Number of stellar particles (integer), <img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}">
- X position of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Y position of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Z position of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Velocity component X of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Velocity component Y of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Velocity component Z of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Mass of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> single-precision reals)
- Unique ID of the stellar particles (<img src="https://render.githubusercontent.com/render/math?math=N_\mathrm{stellar}"> integers)

The units of length, velocity and mass can be freely specified by the user (see [the section on parameters](set_parameters)).

If your input data uses double precision reals, please change the `REAL*4 UBAS(0:PARTI_READ)` line in `READ_PARTICLES_GENERAL` to `REAL*8 UBAS(0:PARTI_READ)`. If you use over 2^31 particles, please compile with `-fdefault-integer-8` (GNU) or `-i8` (Intel) flags.

>#### MASCLET users:
>
>When using ASOHF coupled to MASCLET, take into account the MASCLET version and the flavour of the simulation to adapt the reader format (`gridsXXXXX` and `cldmXXXXX` files).

### Creating your custom reader subroutine

You can alter the reader format by creating a custom subroutine in the `reader.f` source file, or modifying the `READ_PARTICLES_GENERAL` subroutine. If creating your own subroutine, set the correct call to it inside the `READ_AND_ALLOC_PARTICLES` subroutine in `alloc.f`. 

In any case, leave the unit conversion loop unaltered, 

```fortran
DO I=1,N_DM+N_ST
    RXPA(I)=(RXPA(I)-CIO_XC)*FACT_LENGTH
    RYPA(I)=(RYPA(I)-CIO_YC)*FACT_LENGTH
    RZPA(I)=(RZPA(I)-CIO_ZC)*FACT_LENGTH

    U2DM(I)=U2DM(I)*FACT_SPEED
    U3DM(I)=U3DM(I)*FACT_SPEED
    U4DM(I)=U4DM(I)*FACT_SPEED

    MASAP(I)=MASAP(I)*FACT_MASS
END DO
```

and be sure to specify the correct input units in the `asohf.dat` file (see [the section on parameters](set_parameters)).

#### Contributing

If you have created a general reader routine for a widely-used, publicly-available simulation code and want it included in the public version of ASOHF, do not hesitate to create a pull request or to [contact us](mailto:david.valles-perez@uv.es).