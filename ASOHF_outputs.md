---
layout: default
title: Outputs structure
nav_order: 5
last_modified_at: 2022-04-06T13:37:11
---

## ASOHF outputs
Here, we describe the basic structure of ASOHF outputs, which are stored in `./output_files`.

- The basic output of ASOHF is the halo catalogue (one file per snapshot analysed), which is named `familiesXXXXX` (with `XXXXX` the snapshot number).
    - If you choose to write the lists of dark matter particles, you will find a file named `particlesXXXXX` containing the particle IDs of each halo.
- If stellar halo finding is enabled (see [the section on parameters](set_parameters#stellar-halo-finding-parameters-block)), a `stelar_haloesXXXXX` will contain the catalogue of stellar haloes.
    - If you choose to write the lists of stellar particles, you will find a file named `particles_stellarXXXXX` containing the particle IDs of each stellar halo.

Below we describe the output format of each file.

### The halo catalogue (`familiesXXXXX`)

Halo catalogues are stored in a simple text file, with one line per halo. 

The first 7 lines contain some general information. In particular, the second line indicates the iteration number, the number of tentative and actual haloes found, and the redshift.

Subsequent lines contain a description of the variables to be written and their units.

From the 8th line on, each line contains the values of these quantities for each halo.

Several clarifications about the outputs have to be made:

- Halo IDs are not necessarily consecutive.
- The 'Substructure of.' column indicates the ID of the progenitor halo, or -1 if the halo is not a substructure.
- Positions are always given in the original extent of the domain, but in cMpc (regardless of the input units).

### The stellar halo catalogue (`stellar_haloesXXXXX`)

Stellar halo catalogues are stored in a simple text file, with one line per stellar halo. The format followed is parallel to the one of the `families_XXXXX` file. Some subtleties:

- The stellar halo ID does not necessarily coincide with the DM halo ID. In fact, the `stelar_haloesXXXXX` file also stores the ID of the DM halo that hosts the stellar halo.
- We give the DM density peak and the stellar density peak. The latter is the one regarded as centre of the stellar halo.

### The particles files (`particlesXXXXX` and `stellar_particlesXXXXX`)

Particle lists are written as a Fortran unformatted file. For reading them with python, we use the [Cython Fortran File package](https://pypi.org/project/cython-fortran-file/), which you can install with a simple

```bash 
pip install cython-fortran-file
```

The format is as follows, record by record:

- An integer containing the number of haloes
- A look-up table of haloes, with one record per halo, each of which contains:
    - An integer for the halo ID
    - Two integers, containing the lower and upper indices of the particles belonging to the halo in the particle list (below)
- An integer containing the number of particles
- An array of integers, containing a list of particle IDs

Therefore, in order to access the particles of a given halo, one has to find the corresponding indices in the look-up table (e.g., `I1` and `I2`), and then read the particles IDs from the list of particles (e.g., `particles(I1:I2)`). Note the indices are given in Fortran convention, i.e., starting from 1.

## Loading ASOHF outputs into python
Since it may be a minor inconvenience to have to load the data into python, one of the most currently popular languages for analysis tools, since it is necessary to change the indices back and forth to match the different indexing convention, we provide a library to load the different outputs into python (`./tools/reader.py`). The library can be imported as follows:

```python
import sys
sys.path.append('/path/to/the/folder/tools/')
import reader
```

Then, one can directly use the different functions to load the different files specified above.

- `read_families()`: reads the halo catalogue. The user can choose between two useful output formats:
    - `output_format='dictionaries'` (default): returns list of python dictionaries, each of these corresponding to a different halo. Each dictionary contains all the quantities for the given halo. One can `print(catalogue[0].keys())` to see the available keys.
    - `output_format='arrays'`: returns a single dictionary, each of which contains a numpy array with the values of the quantities for all the haloes. The same order is kept in all arrays.
- `read_stellar_haloes()`: same as `read_families()`, but for the stellar halo catalogue. Same output format conventions.
- `read_particles()`: reads the particle lists. The parameter `parttype` (either `DM` or `stellar`) specifies which particle files to read. The output is a dictionary, whose keys are the halo IDs and the values are the list of particles belonging to the corresponding halo, sorted by particle ID.

## Other output files
The intermediate files for debugging purposes, if required to be written (see [the section on parameters](set_parameters)) are saved in the same folder. We do not provide python readers for these files, since they are not meant to be read by the general, black-box user. However, you can look for them in the source code to see the structure of their contents.
