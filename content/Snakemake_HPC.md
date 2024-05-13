# Snakemake on the supercomputer

```{objectives}
- Understand the different ways Snakemake can be run on clusters
- ...
```

## Prerequisites

### Snakemake installation

1. Preinstalled and made available via module: `module load snakemake/version`
2. Conda/Pip based installation
3. Container based installation

Always check first if your cluster has the tool available as module, as then you do not have to worry about installation at all.
If you need additional packages, you can usually add own packages by `pip install`. If that for some reason does not work, you can create your own installation with Conda or Pip, or by making use of containers. **Check your clusters documentation for information on how to do these.**

### Command line arguments for own scripts

Make it possible to provide arguments like paths and other "maybe changing" parameters of your scripts as command line arguments. 

in R:
e.g. args = commandArgs(trailingOnly=TRUE)
in Python:
e.g. sys.argv, click, argparse

### Config files

(from Snakemake docs)
Configuration of a workflow should be handled via config files and, if needed, tabular configuration like sample sheets (either via Pandas or PEPs). Use such configuration for metadata and experiment information, not for runtime specific configuration like threads, resources and output folders. For those, just rely on Snakemake’s CLI arguments like --set-threads, --set-resources, --set-default-resources, and --directory. This makes workflows more readable, scalable, and portable.



## Ways of running Snakemake on clusters

### Interactive

### One job

### Slurm integration

### Hyperqueue

## Snakemake cluster execution

Snakemake provides a [generic cluster executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/cluster-generic.html) as plugin, which can be used to run Snakemake workflows on clusters:

```
module load snakemake

snakemake -s Snakefile --jobs 1 \
 --latency-wait 60 \
 --executor cluster-generic \
 --cluster-generic-submit-cmd "sbatch --time 10 \
 --account=project_xxxx --job-name=hello-world \
 --tasks-per-node=1 --cpus-per-task=1 --mem-per-cpu=4000 --partition=test"
```

Submit Snakemake as a Batch Job; request more resources if needed. All rules are run in the same job allocation.

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

Another way would be to use slurm executor plugin or [HyperQueue](https://docs.csc.fi/support/tutorials/snakemake-puhti/#running-snakemake-workflow-with-apptainer-containers): `snakemake --executor cluster-generic --cluster-generic-submit-cmd "hq submit ..."  `

## Good practices

- Use version control for your workflows
- Use containers for portability
- Avoid unnecessary read and write operations
- Summarize small jobs/job steps into one job
- Use restarting option when running long jobs
- Avoid creating a lot of files, especially in the same folder
- Remove temporary files after the job is finnished
- Separate serial from parallel jobs for efficient use of resources


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
