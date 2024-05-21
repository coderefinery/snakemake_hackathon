# Snakemake on the supercomputer

```{objectives}
- Understand the different ways Snakemake can be run on clusters
- Evaluate which way suits your needs 
- Make own workflow ready for HPC
```

> Note on versions. Earlier (before version 8.0) Snakemake versions deal differently with slurm. Please check the documentation for your version of snakemake. These materials assume Snakemake version 8.0 or later.

## Prerequisites

### Snakemake installation

1. Preinstalled and made available via module: `module load snakemake/version`
2. Conda/Pip based installation
3. Container based installation

Always check first if your cluster has the tool available as module, as then you do not have to worry about installation at all.
If you need additional packages, you can usually add own packages by `pip install` for your own user or project. If that for some reason does not work, you can create your own installation with Conda or Pip, or by making use of containers. **Check your clusters documentation for information on how to do these.**

### Command line arguments for own scripts

Make it possible to provide arguments like paths and other "maybe changing" parameters of your scripts as command line arguments. 


`````{tabs}
  ````{group-tab} Python
    ```Python
    e.g. sys.argv, click, argparse
    ```
  ```` 
  ````{group-tab} R
    ```R
    e.g. args = commandArgs(trailingOnly=TRUE)
    ```
  ````
  ````{group-tab} Bash 
    Use `$1` to access the first command line argument to a bash script.
  ````
`````



## Rule based resources

You can define resources per rule in your snakefile. E.g. one rule may require 4 CPUs to compute, then `threads: 4` can be added to that rule. When snakemake is then run with `--cores = 4` as argument. 
* Other rules can be run in parallel until resources used
* Rule requesting all available resources, ie `threads:4` when `--cores = 4` will use all available resources, and therefore not run in parallel

> Note that if number of cores given to the workflow is smaller than the number of cores requested by a rule, the rule will be run with the amount of cores available to it, ie scaled down. 


## Ways of running Snakemake on clusters

### Interactive or within one batch job

You can run snakemake the same way you would run it on your own computer also on cluster within one interactive or normal batch job. This is a useful first step to try it out, but does not scale well (only to the resource limits of one job).

Example of submitting Snakemake as a Batch Job; request more resources if needed. All rules are run in the same job allocation.

```

#!/bin/bash
#SBATCH --job-name=myTest
#SBATCH --account=project_xxxxx
#SBATCH --time=00:10:00
#SBATCH --mem-per-cpu=2G
#SBATCH --partition=test
#SBATCH --cpus-per-task=4

module load snakemake/8.4.6
snakemake -s Snakefile --use-singularity --jobs 4
```

### Slurm integration with the [Snakemake executor plugin](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html)

With the plugin you can define SLURM batch job resources for your snakemake workflow and let Snakemake submit SLURM jobs on your behalf. 

Snakemake provides both a [Snakemake SLURM executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) and a [generic cluster executor ](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/cluster-generic.html) as plugins, which can be used to run Snakemake workflows on clusters at scale:

```
module load snakemake

snakemake -s Snakefile --jobs 1 \
 --latency-wait 60 \
 --executor cluster-generic \
 --cluster-generic-submit-cmd "sbatch --time 10 \
 --account=project_xxxx --job-name=hello-world \
 --tasks-per-node=1 --cpus-per-task=1 --mem-per-cpu=4000 --partition=test"
```

## Config/Profile files

Snakemake takes many parameters as command line arguments, however, when working with clusters these can become quite many very fast. One way to make the command line call shorter, is to define some of the parameters in cofiguration files. These can be global or workflow specific. 

https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles

(from Snakemake docs)
Configuration of a workflow should be handled via config files and, if needed, tabular configuration like sample sheets (either via Pandas or PEPs). Use such configuration for metadata and experiment information, not for runtime specific configuration like threads, resources and output folders. For those, just rely on Snakemake’s CLI arguments like --set-threads, --set-resources, --set-default-resources, and --directory. This makes workflows more readable, scalable, and portable.

### Hyperqueue

Another way would be to use slurm executor plugin or [HyperQueue](https://docs.csc.fi/support/tutorials/snakemake-puhti/#running-snakemake-workflow-with-apptainer-containers): `snakemake --executor cluster-generic --cluster-generic-submit-cmd "hq submit ..."  `

## Good practices

- Use version control for your workflows
- Use containers for portability
- Avoid unnecessary read and write operations
- Summarize small jobs/job steps into one job
- Use restarting option when running long jobs
- Avoid creating a lot of files, especially in the same folder
- Remove temporary files after the job is finnished: https://carpentries-incubator.github.io/snakemake-novice-bioinformatics/instructor/12-cleaning_up.html
- Separate serial from parallel jobs for efficient use of resources
- Consider job grouping: https://snakemake.readthedocs.io/en/stable/executing/grouping.html

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
