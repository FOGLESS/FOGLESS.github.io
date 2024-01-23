---
layout: default
title: Other topics
nav_order: 6
last_modified_at: 2022-04-06T13:37:11
---

## Other topics
Here we describe other topics which are not essential, but which may be of interest to the user.

### Domain decomposition
The computational cost of running ASOHF in simulations with a large number of particles can be reduced by using a domain decomposition. The domain decomposition is a way to split the simulation box into smaller boxes, or subdomains, which are then processed independently. Since the wall time taken by ASOHF scales as <img src="https://render.githubusercontent.com/render/math?math=\propto N_\mathrm{part} N_\mathrm{haloes}">, running ASOHF with domain decomposition can decrease the wall time required to run the code in a factor  <img src="https://render.githubusercontent.com/render/math?math=1/d"> if all the subdomains are run sequentially (i.e., one after the other). If instead they are run concurrently (e.g., in a memory distributed system with many nodes), the computating time can be reduced by a factor <img src="https://render.githubusercontent.com/render/math?math=1/d^2">, with _d_ the number of subdomains. 

However, this is only useful in large domains (<img src="https://render.githubusercontent.com/render/math?math=\gtrsim 50 \, \mathrm{Mpc}">). This is due to the fact that the strategy requires an overlap, whose size has to be at least the radius of the largest expected halo, to avoid losing objects in the boundaries of the domains.

The domain decomposition strategy employs the domain selection feature in the [parameters file](set_parameters). Since no communication is needed between the different subdomains, until a final unification step, we have chosen to implement in a simple yet versatile way, which requires no libraries such as MPI.

To automatize the process, we provide a python3 script, `./tools/setup_domdecomp.py`. The user needs to set the sidelengths and left corner of the domain in the code input units, 

```python 
sidelength_x=25000.
sidelength_y=25000.
sidelength_z=25000.
domain_xleft=0.
domain_yleft=0.
domain_zleft=0.
```

the number of divisions of the domain along each direction and the size of this _security boundary_ (setting it to around 4 Mpc is enough for simulations with galaxy clusters; for example),
```python
div_x=2
div_y=2
div_z=2
sec_boundary=2500.
```

and several paths (to the executable, the simulation folder, and other required files to launch ASOHF).
```python 
dd_foldername='domain_{:}_{:}_{:}'
executable='asohf.x'
other_files=['run.sh']
simulation_foldername='simulation'
```

Once the script has been executed, a set of folders named `domain_0_0_0`, `domain_0_0_1`, etc. will have been created, all of them containing the executable, a symbolic link to the simulation folder, and the other files needed to run ASOHF. A `domains.out` plain text file will contain the information about the domain decomposition that is later needed to unify the catalogues. To run automatically the code in all the domains, we provide sample shell scripts in the `./tools` folder of the repository:

- `domdecomp_runall_sequential.sh`: runs the code in all the domains sequentially (one after the other)
- `domdecomp_runall_concurrent_slurm.sh`: sends _d_ jobs to a SLURM queue, each of one corresponding to one of the subdomains, which will be executed either sequentially, concurrently or mixed depending on the capabilities of your cluster. 

After all the domains have finished for a given snapshot (or for many of them), you can run the `merge_domdecomp_catalogues.py` script to merge the results of the different domains into a single file. Again, in this script you need to set the iterations you want to analise, some units, the number of divisions and the overlap length between domains, as well as some flags regarding which outputs of ASOHF you have produced.

The code will read the outputs of all domains, and will unify the catalogues of haloes, stellar haloes and the particles lists (if any), and will write the unified files into `./output_files`.

### Merger tree

We also include a python code, `./tools/mtree.py` to build the merger tree from ASOHF outputs.

> Note: for using the merger tree, you will need to install the package [`sortednp`](https://pypi.org/project/sortednp/). This can be easily done by
> ```python
> pip install sortednp
> ```

Running this script requires a minimal configuration from the user:

1. Set the parameters:
   - Set the path to the folder containing the outputs of ASOHF, and the path to the folter containing the simulation results.
   ```python
   outputs_ASOHF='output_files/'
   simulation_results='simu_masclet/'
   ```
   - Set the first iteration to analyse, the last iteration to analyse and the interval between iterations.
   ```python
   itini=100
   itfin=1550
   every=50
   ```
   - Set the value of the density parameters and the Hubble parameter used in the simulation.
   ```python
   h=0.6711
   omegam=0.3026
   omegalambda=1-omegam
   ```

   - Running the merger tree in parallel: set the number of threads to be used for the distance computation (first step of the merger tree), and for the intersection of the particle lists (second steps). The former can be set as high as you wish, while the latter must be kept reasonably low since `sortednp` can use quite a lot of memory.
   ```python
   ncores_distance=20
   ncores_intersect=4
   ```

   - Set the lower threshold of _given mass_. Only the progenitor haloes giving the child halo more than this fraction of its mass will be reported in the merger tree.
   ```python
   min_given_mass = 0.001 
   ```

   - Looking for further snapshots: it the parameter is set to `True`, the merger tree code will identify those haloes which have not been linked to any halo in the immediately previous snapshot, and will try to link it to haloes in snapshots further backwards in time. The `max_iterations_back` parameter sets the maximum number of snapshots that the code is allowed to go back.
   ```python 
   look_further_iterations = True
   max_iterations_back = 2 # set it to a very large number to go always until the first iteration
   ```

2. Write a function that relates each DM particle unique ID to its mass, by creating a dictionary whose keys are the IDs and whose values are the masses. An example is given below for `MASCLET`, but you should implement yours for your simulation.

```python
def IDs_to_masses(it, folder_simu):
    from masclet_framework.read_masclet import read_cldm
    from masclet_framework.units import mass_to_sun as munit
    mdm,oripa = read_cldm(it, path=folder_simu, output_deltadm=False, output_position=False, output_velocity=False,output_mass=True, output_id=True)
    # fix IDs from MASCLET to be unique
    oripa[mdm>mdm.max()/2]=-np.abs(oripa[mdm>mdm.max()/2])
    return {partid: mi for mi,partid in zip(mdm*munit,oripa)}
```

3. The script uses [tqdm](https://github.com/tqdm/tqdm) to display a progress bar. If you do not have `tqdm` installed and do not wish to install it, you can comment out the line `from tqdm import tqdm` and uncomment the line after it: 
```python 
def tqdm(x): return x
```

Then you can just run the `mtree.py` script from your favourite `python3` interpreter.

The outputs will consist on files with the denomination `mtree_XXXXX_YYYYY.json` in your `./output_files` folder, where `XXXXX` is the _previous_ iteration and `YYYYY` is the _posterior_ iteration. This is a `json` structured file, which can be easily read into python by:

```python
import json
with open('output_files/mtree_00100_00150.json', 'r') as f:
    data = json.load(f)
```

This will automatically load a dictionary. The keys are the IDs of the haloes in the _posterior_ iteration. The values are a list of dictionaries, each of these dictionaries corresponding to a progenitor halo. These dictionaries contain the halo ID, the fraction of progenitor mass given to the child, the fraction of the child mass which comes from this progenitor, amongst other data. Additionaly, the dictionary contains a key name `containsMostBound`, which is `True` if the most-bound particle of the progenitor is in the descendant halo and viceversa. This is the criterion we use to identify the main progenitor of each halo.

> NOTE: (which you may need to convert the keys from strings to integers, by running 
>
>```python
>data = {int(k): v for k, v in data.items()}
>``` 

If the main progenitor of a halo has not been found in the immediately previous snapshot, but it is found in some further one, you will find it in the file connecting these two iterations. For example, it the progenitor of a halo in the iteration 10 is not found in the iteration 9, but it is in the iteration 8, you might find it in the file `mtree_8_10.json` file.