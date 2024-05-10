# Snakemake on the supercomputer

Considerations and preparations to move a workflow to cluster

## Prerequisite: Portable workflow

Check your filepaths!

## Working with modules

`module load snakemake/version`

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


