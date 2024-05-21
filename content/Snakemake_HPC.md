# Snakemake on the supercomputer

```{objectives}
- Understand the different ways Snakemake can be run on clusters
- Evaluate which way suits your needs 
- Make own workflow ready for HPC
```

> Note on versions: These materials assume Snakemake version 8.0 or later. Earlier (before version 8.0) Snakemake versions deal differently with SLURM. Please check the documentation for your version of snakemake. 


## Cluster Snakemake installation

1. Preinstalled and made available via module: `module load snakemake/version`
2. Own Conda/Pip based installation
3. Own container based installation

Always check first if your cluster has the tool available as **module**, then you do not have to worry about installation yourself.
If you need additional packages, you can usually add own packages by `pip install` for your own user or project. It may also be possible to load multiple modules at the same time (feasibility depends on the modules). If that for some reason does not work, you can create your own installation with Conda or Pip, or by making use of containers. **Check your clusters documentation for information on how to do these.**


## Rule based resources

You can define resources per rule in your snakefile. E.g. one rule may require 4 CPUs to compute, then `threads: 4` can be added to that rule. When snakemake is then run with `--cores = 4` as argument.

* Other rules are automatically run in parallel until all resources are used
* Rule requesting all available resources, i.e. `threads:4` when `--cores = 4` will use all available resources, and will therefore not run in parallel with other rules

> Note that if number of cores given to the workflow is smaller than the number of cores requested by a rule, the rule will be run with the amount of cores available to it, i.e. scaled down.


## Ways of running Snakemake on clusters

### Within one SLURM job

#### Interactive

You can run snakemake the same way you would run it on your own computer also on cluster within one interactive or normal batch job (one resource allocation for all rules). This is a useful first step to try it out, but does not scale well (only to the resource limits of one job).

In an interactive job (using the interactive partition; some clusters also support `Sinteractive` command), you would load the snakemake module, and then run `snakemake --cores = 1`.

You can run snakemake in the background for example with `screen`.

##### Sbatch script

Another way to run snakemake in **one job** is to use an sbatch script for submission.

```
#!/bin/bash
#SBATCH --job-name=myTest
#SBATCH --account=project_xxxxx
#SBATCH --time=00:10:00
#SBATCH --mem-per-cpu=2G
#SBATCH --partition=test
#SBATCH --cpus-per-task=4

module load snakemake/version
snakemake -s Snakefile --jobs 4
```

#### Slurm integration with the [Snakemake executor plugin](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html)

With the plugin you can define SLURM batch job resources for your snakemake workflow instead of writing an sbatch file.


Snakemake provides a [Snakemake SLURM executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) as plugin, which can be used to run Snakemake workflows on clusters:

```
module load snakemake/version

snakemake -s Snakefile --keep-going  --jobs 1 --executor slurm --default-resources slurm_account=project_xxx slurm_partition=small  
```

The SLURM integration also provides the possibility to start multiple jobs for you; for example one for each rule. However, this option can be heavy on the SLURM database and filesystem, and should therefore be avoided. A better solution to scale up your Snakemake workflow is to use Hyperqueue instead:

### Snakemake at scale using Hyperqueue

HyperQueue (HQ) is an efficient sub-node task scheduler. Instead of submitting each of your computational tasks as separate Slurm jobs or job steps, you can allocate a large resource block and then use HyperQueue to submit your tasks to this allocation.
The [generic cluster executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/cluster-generic.html) plugin can be used to configure hyperqueue runs, e.g. `snakemake --executor cluster-generic --cluster-generic-submit-cmd "hq submit ..." `.

- Useful for high-throughput jobs (many steps and rules on many files, needing a lot of resources)
- Multiple nodes can be deployed under the same job allocation


## Config/Profile files

Snakemake takes many parameters as command line arguments. However, when working with clusters these can become quite many very fast. One way to make the command line call shorter, is to define some of the parameters in [**profiles**](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles). These can be global or workflow specific. 

A profile is written in `YAML format` and can be provided to Snakemake using the `--profile` flag. Option `--someoption` becomes `someoption: ` in the profile. Example profile:

```
executor: slurm
jobs: 1
```

## Good practices

- General: 
    - As with all code, it is advised to also keep your Snakefiles under **version control**, e.g. using `git`
    - Consider **containerizing** your whole workflow for portability
- When running Snakemake on clusters: 
    - Try to avoid unnecessary read and write operations, check if use of fast local disk is possible
    - Consider summarize small jobs/job steps into one job to limit SLURM accounting overhead, e.g. by [job grouping](https://snakemake.readthedocs.io/en/stable/executing/grouping.html)
    - Use restarting option (`--retries X / --restart-times X , -T X`) when running long workflows, in order to avoid wasting resources
    - Try to avoid creating a lot of files, especially in the same folder, e.g. by letting [snakemake remove temporary files after the job is finnished](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#protected-and-temporary-files)

##  Cluster specific information

`````{tabs}
  ````{group-tab} CSC Puhti or LUMI
Running Snakemake on CSC's Puhti or LUMI: [Slides](https://a3s.fi/CSC_training/snakemake_hackathon.html#/snakemake-hackathon-in-csc-supercomputers).
  ````
  ````{group-tab} UPPMAX
TBD
  ````
`````


```{keypoints}
- TBD
```
